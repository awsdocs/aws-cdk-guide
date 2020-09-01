# Testing constructs<a name="testing"></a>

With the AWS CDK, your infrastructure can be as testable as any other code you write\. This article illustrates one approach to testing AWS CDK apps written in TypeScript using the [Jest](https://jestjs.io/) test framework\. Currently, TypeScript is the only supported language for testing AWS CDK infrastructure, though we intend to eventually make this capability available in all languages supported by the AWS CDK\.

There are three categories of tests you can write for AWS CDK apps\.
+  **Snapshot tests** test the synthesized AWS CloudFormation template against a previously\-stored baseline template\. This way, when you're refactoring your app, you can be sure that the refactored code works exactly the same way as the original\. If the changes were intentional, you can accept a new baseline for future tests\.
+  **Fine\-grained assertions** test specific aspects of the generated AWS CloudFormation template, such as "this resource has this property with this value\." These tests help when you're developing new features, since any code you add will cause your snapshot test to fail even if existing features still work\. When this happens, your fine\-grained tests will reassure you that the existing functionality is unaffected\. 
+  **Validation tests** help you "fail fast" by making sure your AWS CDK constructs raise errors when you pass them invalid data\. The ability to do this type of testing is a big advantage of developing your infrastructure in a general\-purpose programming language\. 

## Getting started<a name="testing_getting_started"></a>

As an example, we'll create a [dead letter queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) construct\. A dead letter queue holds messages from another queue that have failed delivery for some time\. This usually indicates failure of the message processor, which we want to know about, so our dead letter queue has an alarm that fires when a message arrives\. The user of the construct can hook up actions such as notifying an Amazon SNS topic to this alarm\. 

### Creating the construct<a name="testing_creating_construct"></a>

 Start by creating an empty construct library project using the AWS CDK Toolkit and installing the construct libraries we'll need: 

```
mkdir dead-letter-queue && cd dead-letter-queue
cdk init --language=typescript lib
npm install @aws-cdk/aws-sqs @aws-cdk/aws-cloudwatch
```

 Place the following code in `lib/index.ts`: 

```
import * as cloudwatch from '@aws-cdk/aws-cloudwatch';
import * as sqs from '@aws-cdk/aws-sqs';
import { Construct, Duration } from '@aws-cdk/core';

export class DeadLetterQueue extends sqs.Queue {
  public readonly messagesInQueueAlarm: cloudwatch.IAlarm;

  constructor(scope: Construct, id: string) {
    super(scope, id);

    // Add the alarm
    this.messagesInQueueAlarm = new cloudwatch.Alarm(this, 'Alarm', {
      alarmDescription: 'There are messages in the Dead Letter Queue',
      evaluationPeriods: 1,
      threshold: 1,
      metric: this.metricApproximateNumberOfMessagesVisible(),
    });
  }
}
```

### Installing the testing framework<a name="testing_installing"></a>

Since we're using the Jest framework, our next setup step is to install Jest\. We'll also need the AWS CDK assert module, which includes helpers for writing tests for CDK libraries, including `assert` and `expect`\.

```
npm install --save-dev jest @types/jest @aws-cdk/assert
```

### Updating `package.json`<a name="testing_project"></a>

Finally, edit the project's `package.json` to tell NPM how to run Jest, and to tell Jest what kinds of files to collect\. The necessary changes are as follows\. 
+ Add a new `test` key to the `scripts` section
+ Add Jest and its types to the `devDependencies` section
+ Add a new `jest` top\-level key with a `moduleFileExtensions` declaration

These changes are shown in outline below\. Place the new text where indicated in `package.json`\. The "\.\.\." placeholders indicate existing parts of the file that should not be changed\. 

```
{
  ...
 "scripts": {
    ...
    "test": "jest"
  },
  "devDependencies": {
    ...
    "@types/jest": "^24.0.18",
    "jest": "^24.9.0",
  },
  "jest": {
    "moduleFileExtensions": ["js"]
  }
}
```

## Snapshot tests<a name="testing_snapshot"></a>

Add a snapshot test by placing the following code in `test/dead-letter-queue.test.ts`\.

```
import { SynthUtils } from '@aws-cdk/assert';
import { Stack } from '@aws-cdk/core';

import * as dlq from '../lib/index';

test('dlq creates an alarm', () => {
  const stack = new Stack();
  new dlq.DeadLetterQueue(stack, 'DLQ');
  expect(SynthUtils.toCloudFormation(stack)).toMatchSnapshot();
});
```

To build the project and run the test, issue these commands\.

```
npm run build && npm test
```

The output from Jest indicates that it has run the test and recorded a snapshot\.

```
PASS  test/dead-letter-queue.test.js
 ✓ dlq creates an alarm (55ms)
 › 1 snapshot written.
Snapshot Summary
› 1 snapshot written
```

Jest stores the snapshots in a directory named `__snapshots__` inside the project\. In this directory is a copy of the AWS CloudFormation template generated by the dead letter queue construct\. The beginning looks something like this\.

```
exports[`dlq creates an alarm 1`] = `
Object {
  "Resources": Object {
    "DLQ581697C4": Object {
      "Type": "AWS::SQS::Queue",
    },
    "DLQAlarm008FBE3A": Object {
     "Properties": Object {
        "AlarmDescription": "There are messages in the Dead Letter Queue",
        "ComparisonOperator": "GreaterThanOrEqualToThreshold",
...
```

### Testing the test<a name="testing_testing_test"></a>

To make sure the test works, change the construct so that it generates different AWS CloudFormation output, then build and test again\. For example, add a `period` property of 1 minute to override the default of 5 minutes\. The boldface line below shows the code that needs to be added to `index.ts`\. 

```
    this.messagesInQueueAlarm = new cloudwatch.Alarm(this, 'Alarm', {
    alarmDescription: 'There are messages in the Dead Letter Queue',
    evaluationPeriods: 1,
    threshold: 1,
    metric: this.metricApproximateNumberOfMessagesVisible(),
    period: Duration.minutes(1),
});
```

Build the project and run the tests again\.

```
npm run build && npm test
```

```
FAIL test/dead-letter-queue.test.js
✕ dlq creates an alarm (58ms)

● dlq creates an alarm

expect(received).toMatchSnapshot()

Snapshot name: `dlq creates an alarm 1`

- Snapshot
+ Received

@@ -19,11 +19,11 @@
               },
             ],
             "EvaluationPeriods": 1,
             "MetricName": "ApproximateNumberOfMessagesVisible",
             "Namespace": "AWS/SQS",
     -       "Period": 300,
     +       "Period": 60,
             "Statistic": "Maximum",
             "Threshold": 1,
           },
           "Type": "AWS::CloudWatch::Alarm",
         },

 › 1 snapshot failed.
Snapshot Summary
 › 1 snapshot failed from 1 test suite. Inspect your code changes or run `npm test -- -u` to update them.
```

### Accepting the new snapshot<a name="testing_accepting"></a>

Jest has told us that the `Period` attribute of the synthesized AWS CloudFormation template has changed from 300 to 60\. To accept the new snapshot, issue:

```
npm test -- -u
```

Now we can run the test again and see that it passes\.

### Limitations<a name="testing_limitations"></a>

Snapshot tests are easy to create and are a powerful backstop when refactoring\. They can serve as an early warning sign that more testing is needed\. Snapshot tests can even be useful for test\-driven development: modify the snapshot to reflect the result you're aiming for, and adjust the code until the test passes\.

The chief limitation of snapshot tests is that they test the *entire* template\. Consider that our dead letter queue uses the default retention period\. To give ourselves as much time as possible to recover the undelivered messages, for example, we might set the queue's retention time to the maximum—14 days—by changing the code as follows\.

```
export class DeadLetterQueue extends sqs.Queue {
  public readonly messagesInQueueAlarm: cloudwatch.IAlarm;

  constructor(scope: Construct, id: string) {
    super(scope, id, {
      // Maximum retention period
      retentionPeriod: Duration.days(14)
    });
```

When we run the test again, it breaks\. The name we've given the test hints that we are interested mainly in testing whether the alarm is created, but the snapshot test also tests whether the queue is created with default options—along with literally everything else about the synthesized template\. This problem is magnified when a project contains many constructs, each with a snapshot test\.

## Fine\-grained assertions<a name="testing_fine_grained"></a>

To avoid needing to review every snapshot whenever you make a change, use the custom assertions in the `@aws-cdk/assert/jest` module to write fine\-grained tests that verify only part of the construct's behavior\. For example, the test we called "dlq creates an alarm" in our example really should assert only that an alarm is created with the appropriate metric\.

The [AWS::CloudWatch::Alarm](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-cw-alarm.html) resource specification reveals that we're interested in the properties Namespace, MetricName and Dimensions\. We'll use the `expect(stack).toHaveResource(...)` assertion, which is in the `@aws-cdk/assert/jest` module, to make sure these properties have the appropriate values\. 

Replace the code in `test/dead-letter-queue.test.ts` with the following\.

```
import { Stack } from '@aws-cdk/core';
import '@aws-cdk/assert/jest';

import * as dlq from '../lib/index';

test('dlq creates an alarm', () => {
  const stack = new Stack();

  new dlq.DeadLetterQueue(stack, 'DLQ');

  expect(stack).toHaveResource('AWS::CloudWatch::Alarm', {
    MetricName: "ApproximateNumberOfMessagesVisible",
    Namespace: "AWS/SQS",
    Dimensions: [
      {
        Name: "QueueName",
        Value: { "Fn::GetAtt": [ "DLQ581697C4", "QueueName" ] }
      }
    ],
  });
});

test('dlq has maximum retention period', () => {
  const stack = new Stack();

  new dlq.DeadLetterQueue(stack, 'DLQ');

  expect(stack).toHaveResource('AWS::SQS::Queue', {
    MessageRetentionPeriod: 1209600
  });
});
```

There are now two tests\. The first checks that the dead letter queue creates an alarm on its `ApproximateNumberOfMessagesVisible` metric\. The second verifies the message retention period\.

Again, build the project and run the tests\.

```
npm run build && npm test
```

**Note**  
Since we've replaced the snapshot test, the first time we run the new tests, Jest reminds us that we have a snapshot that is not used by any test\. Issue `npm test -- -u` to tell Jest to clean it up\.

## Validation tests<a name="testing_validation"></a>

Suppose we want to make the dead letter queue's retention period configurable\. Of course, we also want to make sure that the value provided by the user of the construct is within an allowable range\. We can write a test to make sure that the validation logic works: pass in invalid values and see what happens\.

First, create a `props` interface for the construct\. 

```
export interface DeadLetterQueueProps {
    /**
     * The amount of days messages will live in the dead letter queue
     *
     * Cannot exceed 14 days.
     *
     * @default 14
     */
    retentionDays?: number;
}

export class DeadLetterQueue extends sqs.Queue {
  public readonly messagesInQueueAlarm: cloudwatch.IAlarm;

  constructor(scope: Construct, id: string, props: DeadLetterQueueProps = {}) {
    if (props.retentionDays !== undefined && props.retentionDays > 14) {
      throw new Error('retentionDays may not exceed 14 days');
    }

    super(scope, id, {
        // Given retention period or maximum
        retentionPeriod: Duration.days(props.retentionDays || 14)
    });
    // ...
  }
}
```

To test that the new feature actually does what we expect, we write two tests:
+ One that makes sure the configured value ends up in the template
+ One that supplies an incorrect value to the construct and checks it raises the expected error

Add the following to `test/dead-letter-queue.test.ts`\.

```
test('retention period can be configured', () => {
  const stack = new Stack();

  new dlq.DeadLetterQueue(stack, 'DLQ', {
    retentionDays: 7
  });

  expect(stack).toHaveResource('AWS::SQS::Queue', {
    MessageRetentionPeriod: 604800
  });
});

test('configurable retention period cannot exceed 14 days', () => {
  const stack = new Stack();

  expect(() => {
    new dlq.DeadLetterQueue(stack, 'DLQ', {
      retentionDays: 15
    });
  }).toThrowError(/retentionDays may not exceed 14 days/);
});
```

Run the tests to confirm the construct behaves as expected\.

```
npm run build && npm test
```

```
PASS  test/dead-letter-queue.test.js
  ✓ dlq creates an alarm (62ms)
  ✓ dlq has maximum retention period (14ms)
  ✓ retention period can be configured (18ms)
  ✓ configurable retention period cannot exceed 14 days (1ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
```

## Tips for tests<a name="testing_tips"></a>

Remember, your tests will live just as long as the code they test, and be read and modified just as often, so it pays to take a moment to consider how best to write them\. Don't copy and paste setup lines or common assertions, for example; refactor this logic into helper functions\. Use good names that reflect what each test actually tests\.

Don't assert too much in one test\. Preferably, a test should test one and only one behavior\. If you accidentally break that behavior, exactly one test should fail, and the name of the test should tell you exactly what failed\. This is more an ideal to be striven for, however; sometimes you will unavoidably \(or inadvertently\) write tests that test more than one behavior\. Snapshot tests are, for reasons we've already described, especially prone to this problem, so use them sparingly\.