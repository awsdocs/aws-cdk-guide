# Testing constructs<a name="testing"></a>

With the AWS CDK, your infrastructure can be as testable as any other code you write\. This article illustrates the standard approach to testing AWS CDK apps using the AWS CDK's [assertions](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.assertions-readme.html) module and popular test frameworks such as [Jest](https://jestjs.io/) for TypeScript and JavaScript or [Pytest](https://docs.pytest.org/en/6.2.x/) for Python\.

There are two categories of tests you can write for AWS CDK apps\.
+  **Fine\-grained assertions** test specific aspects of the generated AWS CloudFormation template, such as "this resource has this property with this value\." These tests can detect regressions, and are also useful when you're developing new features using test\-driven development \(write a test first, then make it pass by writing a correct implementation\)\. Fine\-grained assertions are the tests you'll write the most of\.
+  **Snapshot tests** test the synthesized AWS CloudFormation template against a previously\-stored baseline template or "master\." Snapshot tests let you refactor freely, since you can be sure that the refactored code works exactly the same way as the original\. If the changes were intentional, you can accept a new baseline for future tests\. However, CDK upgrades can also cause synthesized templates to change, so you can't rely only on snapshots to make sure your implementation is correct\.

**Note**  
Complete versions of the TypeScript, Python, and Java apps used as examples in this topic are [available on GitHub](https://github.com/cdklabs/aws-cdk-testing-examples/)\.

## Getting started<a name="testing_getting_started"></a>

To illustrate how to write these tests, we'll create a stack that contains an AWS Step Functions state machine and a AWS Lambda function\. The Lambda function is subscribed to an Amazon SNS topic and simply forwards the message to the state machine\.

First, create an empty CDK application project using the CDK Toolkit and installing the libraries we'll need\. The constructs we'll use are all in the main CDK package, which is a default dependency in projects created with the CDK Toolkit, but you'll need to install your testing framework\.

------
#### [ TypeScript ]

```
mkdir state-machine && cd state-machine
cdk init --language=typescript
npm install --save-dev jest @types/jest
```

Create a directory for your tests\.

```
mkdir test
```

Edit the project's `package.json` to tell NPM how to run Jest, and to tell Jest what kinds of files to collect\. The necessary changes are as follows\. 
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
    "jest": "^24.9.0"
  },
  "jest": {
    "moduleFileExtensions": ["js"]
  }
}
```

------
#### [ JavaScript ]

```
mkdir state-machine && cd state-machine
cdk init --language=javascript
npm install --save-dev jest
```

Create a directory for your tests\.

```
mkdir test
```

Edit the project's `package.json` to tell NPM how to run Jest, and to tell Jest what kinds of files to collect\. The necessary changes are as follows\. 
+ Add a new `test` key to the `scripts` section
+ Add Jest to the `devDependencies` section
+ Add a new `jest` top\-level key with a `moduleFileExtensions` declaration

These changes are shown in outline below\. Place the new text where indicated in `package.json`\. The "\.\.\." placeholders indicate existing parts of the file that s\.hould not be changed\. 

```
{
  ...
  "scripts": {
    ...
    "test": "jest"
  },
  "devDependencies": {
    ...
    "jest": "^24.9.0"
  },
  "jest": {
    "moduleFileExtensions": ["js"]
  }
}
```

------
#### [ Python ]

```
mkdir state-machine && cd state-machine
cdk init --language=python
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install -r requirements-dev.txt
```

------
#### [ Java ]

```
mkdir state-machine && cd-state-machine
cdk init --language=java
```

Open the project in your preferred Java IDE\. \(In Eclipse, use **File** > **Import** > Existing Maven Projects\.\)

------
#### [ C\# ]

```
mkdir state-machine && cd-state-machine
cdk init --language=csharp
```

Open `src\StateMachine.sln` in Visual Studio\.

Right\-click the solution in Solution Explorer and choose **Add** > **New Project**\. Search for MSTest C\# and add an **MSTest Test Project** for C\#\. \(The default name `TestProject1`is fine\.\)

Right\-click `TestProject1` and choose **Add** > **Project Reference**, and add the `StateMachine` project as a reference\.

------

## The example stack<a name="testing_app"></a>

Here's the stack we'll be testing in this topic\. As we've previously described, it contains a Lambda function and a Step Functions state machine, and accepts one or more Amazon SNS topics\. The Lambda function is subscribed to the Amazon SNS topics and forwards them to the state machine\. 

You don't have to do anything special to make the app testable\. In fact, this CDK stack is not different in any important way from the other example stacks in this Guide\.

------
#### [ TypeScript ]

Place the following code in `lib/state-machine-stack.ts`: 

```
import * as cdk from "aws-cdk-lib";
import * as sns from "aws-cdk-lib/aws-sns";
import * as sns_subscriptions from "aws-cdk-lib/aws-sns-subscriptions";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as sfn from "aws-cdk-lib/aws-stepfunctions";
import { Construct } from "constructs";

export interface ProcessorStackProps extends cdk.StackProps {
  readonly topics: sns.Topic[];
}

export class ProcessorStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props: ProcessorStackProps) {
    super(scope, id, props);

    // In the future this state machine will do some work...
    const stateMachine = new sfn.StateMachine(this, "StateMachine", {
      definition: new sfn.Pass(this, "StartState"),
    });

    // This Lambda function starts the state machine.
    const func = new lambda.Function(this, "LambdaFunction", {
      runtime: lambda.Runtime.NODEJS_14_X,
      handler: "handler",
      code: lambda.Code.fromAsset("./start-state-machine"),
      environment: {
        STATE_MACHINE_ARN: stateMachine.stateMachineArn,
      },
    });
    stateMachine.grantStartExecution(func);

    const subscription = new sns_subscriptions.LambdaSubscription(func);
    for (const topic of props.topics) {
      topic.addSubscription(subscription);
    }
  }
}
```

------
#### [ JavaScript ]

Place the following code in `lib/state-machine-stack.js`: 

```
const cdk = require("aws-cdk-lib");
const sns = require("aws-cdk-lib/aws-sns");
const sns_subscriptions = require("aws-cdk-lib/aws-sns-subscriptions");
const lambda = require("aws-cdk-lib/aws-lambda");
const sfn = require("aws-cdk-lib/aws-stepfunctions");

class ProcessorStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // In the future this state machine will do some work...
    const stateMachine = new sfn.StateMachine(this, "StateMachine", {
      definition: new sfn.Pass(this, "StartState"),
    });

    // This Lambda function starts the state machine.
    const func = new lambda.Function(this, "LambdaFunction", {
      runtime: lambda.Runtime.NODEJS_14_X,
      handler: "handler",
      code: lambda.Code.fromAsset("./start-state-machine"),
      environment: {
        STATE_MACHINE_ARN: stateMachine.stateMachineArn,
      },
    });
    stateMachine.grantStartExecution(func);

    const subscription = new sns_subscriptions.LambdaSubscription(func);
    for (const topic of props.topics) {
      topic.addSubscription(subscription);
    }
  }
}

module.exports = { ProcessorStack }
```

------
#### [ Python ]

Place the following code in `state_machine/state_machine_stack.py`:

```
from typing import List

import aws-cdk.aws_lambda as lambda_
import aws-cdk.aws_sns as sns
import aws-cdk.aws_sns_subscriptions as sns_subscriptions
import aws-cdk.aws_stepfunctions as sfn
import aws-cdk as cdk

class ProcessorStack(cdk.Stack):
    def __init__(
        self,
        scope: cdk.Construct,
        construct_id: str,
        *,
        topics: List[sns.Topic],
        **kwargs
    ) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # In the future this state machine will do some work...
        state_machine = sfn.StateMachine(
            self, "StateMachine", definition=sfn.Pass(self, "StartState")
        )

        # This Lambda function starts the state machine.
        func = lambda_.Function(
            self,
            "LambdaFunction",
            runtime=lambda_.Runtime.NODEJS_14_X,
            handler="handler",
            code=lambda_.Code.from_asset("./start-state-machine"),
            environment={
                "STATE_MACHINE_ARN": state_machine.state_machine_arn,
            },
        )
        state_machine.grant_start_execution(func)

        subscription = sns_subscriptions.LambdaSubscription(func)
        for topic in topics:
            topic.add_subscription(subscription)
```

------
#### [ Java ]

```
package software.amazon.samples.awscdkassertionssamples;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.services.lambda.Code;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.sns.ITopicSubscription;
import software.amazon.awscdk.services.sns.Topic;
import software.amazon.awscdk.services.sns.subscriptions.LambdaSubscription;
import software.amazon.awscdk.services.stepfunctions.Pass;
import software.amazon.awscdk.services.stepfunctions.StateMachine;

import java.util.Collections;
import java.util.List;

public class ProcessorStack extends Stack {
    public ProcessorStack(final Construct scope, final String id, final List<Topic> topics) {
        this(scope, id, null, topics);
    }

    public ProcessorStack(final Construct scope, final String id, final StackProps props, final List<Topic> topics) {
        super(scope, id, props);

        // In the future this state machine will do some work...
        final StateMachine stateMachine = StateMachine.Builder.create(this, "StateMachine")
                .definition(new Pass(this, "StartState"))
                .build();

        // This Lambda function starts the state machine.
        final Function func = Function.Builder.create(this, "LambdaFunction")
                .runtime(Runtime.NODEJS_14_X)
                .handler("handler")
                .code(Code.fromAsset("./start-state-machine"))
                .environment(Collections.singletonMap("STATE_MACHINE_ARN", stateMachine.getStateMachineArn()))
                .build();
        stateMachine.grantStartExecution(func);

        final ITopicSubscription subscription = new LambdaSubscription(func);
        for (final Topic topic : topics) {
            topic.addSubscription(subscription);
        }
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using Amazon.CDK.AWS.StepFunctions;
using Amazon.CDK.AWS.SNS;
using Amazon.CDK.AWS.SNS.Subscriptions;
using Constructs;

using System.Collections.Generic;

namespace AwsCdkAssertionSamples
{
    public class StateMachineStackProps : StackProps
    {
        public Topic[] Topics;
    }

    public class StateMachineStack : Stack
    {

        internal StateMachineStack(Construct scope, string id, StateMachineStackProps props = null) : base(scope, id, props)
        {
            // In the future this state machine will do some work...
            var stateMachine = new StateMachine(this, "StateMachine", new StateMachineProps
            {
                Definition = new Pass(this, "StartState")
            });

            // This Lambda function starts the state machine.
            var func = new Function(this, "LambdaFunction", new FunctionProps
            {
                Runtime = Runtime.NODEJS_14_X,
                Handler = "handler",
                Code = Code.FromAsset("./start-state-machine"),
                Environment = new Dictionary<string, string>
                {
                    { "STATE_MACHINE_ARN", stateMachine.StateMachineArn }
                }
            });
            stateMachine.GrantStartExecution(func);
           
            foreach (Topic topic in props?.Topics ?? new Topic[0])
            {
                var subscription = new LambdaSubscription(func);
            }

        }
    }
}
```

------

We'll modify the app's main entry point to not actually instantiate our stack, since we don't want to accidentally deploy it\. Our tests will create an app and an instance of the stack for testing\. This is a useful tactic when combined with test\-driven development: make sure the stack passes all tests before you enable deployment\.

------
#### [ TypeScript ]

In `bin/state-machine.ts`:

```
#!/usr/bin/env node
import * as cdk from "aws-cdk-lib";

const app = new cdk.App();

// Stacks are intentionally not created here -- this application isn't meant to
// be deployed.
```

------
#### [ JavaScript ]

In `bin/state-machine.js`:

```
#!/usr/bin/env node
const cdk = require("aws-cdk-lib");

const app = new cdk.App();

// Stacks are intentionally not created here -- this application isn't meant to
// be deployed.
```

------
#### [ Python ]

In `app.py`:

```
#!/usr/bin/env python3
import os

import aws_cdk  as cdk

app = cdk.App()

# Stacks are intentionally not created here -- this application isn't meant to
# be deployed.

app.synth()
```

------
#### [ Java ]

```
package software.amazon.samples.awscdkassertionssamples;

import software.amazon.awscdk.App;

public class SampleApp {
    public static void main(final String[] args) {
        App app = new App();

        // Stacks are intentionally not created here -- this application isn't meant to be deployed.

        app.synth();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;

namespace AwsCdkAssertionSamples
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            var app = new App();

            // Stacks are intentionally not created here -- this application isn't meant to be deployed.

            app.Synth();
        }
    }
}
```

------

## The Lambda function<a name="testing_lambda"></a>

Our example stack includes a Lambda function that starts our state machine\. We must provide the source code for this function so the CDK can bundle it up and deploy it as part of creating the Lambda function resource\.
+ Create the folder `start-state-machine` in the app's main directory\.
+ In this folder, create at least one file\. For example, you can save the code below in `start-state-machines/index.js`\.

  ```
  exports.handler = async function (event, context) {
  	return 'hello world';
  };
  ```

  However, any file will work, since we won't actually be deploying the stack\.

## Running tests<a name="testing_running_tests"></a>

For reference, here are the commands you use to run tests in your AWS CDK app\. These are the same commands you'd use to run the tests in any project using the same testing framework\. For languages that require a build step, include that to make sure your tests have been compiled\.

------
#### [ TypeScript ]

```
tsc && npm test
```

------
#### [ JavaScript ]

```
npm test
```

------
#### [ Python ]

```
python -m pytest
```

------
#### [ Java ]

```
mvn compile && mvn test
```

------
#### [ C\# ]

Build your solution \(F6\) to discover the tests, then run the tests \(**Test** > **Run All Tests**\)\. To choose which tests to run, open Test Explorer \(**Test** > **Test Explorer**\)\.

Or:

```
dotnet test src
```

------

## Fine\-grained assertions<a name="testing_fine_grained"></a>

The first step for testing a stack with fine\-grained assertions is to synthesize the stack, because we're writing assertions against the generated AWS CloudFormation template\.

Our `ProcessorStack` requires that we pass it the Amazon SNS topic to be forwarded to the state machine\. So in our test, we'll create a separate stack to contain the topic\.

Ordinarily, if you were writing a CDK app, you'd subclass `Stack` and instantiate the Amazon SNS topic in the stack's constructor\. In our test, we instantiate `Stack` directly, then pass this stack as the `Topic`'s scope, attaching it to the stack\. This is functionally equivalent, less verbose, and helps make stacks used only in tests "look different" from stacks you intend to deploy\.

------
#### [ TypeScript ]

```
import { Capture, Match, Template } from "aws-cdk-lib/assertions";
import * as cdk from "aws-cdk-lib";
import * as sns from "aws-cdk-lib/aws-sns";
import { ProcessorStack } from "../lib/processor-stack";

describe("ProcessorStack", () => {
  test("synthesizes the way we expect", () => {
    const app = new cdk.App();

    // Since the ProcessorStack consumes resources from a separate stack
    // (cross-stack references), we create a stack for our SNS topics to live
    // in here. These topics can then be passed to the ProcessorStack later,
    // creating a cross-stack reference.
    const topicsStack = new cdk.Stack(app, "TopicsStack");

    // Create the topic the stack we're testing will reference.
    const topics = [new sns.Topic(topicsStack, "Topic1", {})];

    // Create the ProcessorStack.
    const processorStack = new ProcessorStack(app, "ProcessorStack", {
      topics: topics, // Cross-stack reference
    });

    // Prepare the stack for assertions.
    const template = Template.fromStack(processorStack);


}
```

------
#### [ JavaScript ]

```
const { Capture, Match, Template } = require("aws-cdk-lib/assertions");
const cdk = require("aws-cdk-lib");
const sns = require("aws-cdk-lib/aws-sns");
const { ProcessorStack } = require("../lib/processor-stack");

describe("ProcessorStack", () => {
  test("synthesizes the way we expect", () => {
    const app = new cdk.App();

    // Since the ProcessorStack consumes resources from a separate stack
    // (cross-stack references), we create a stack for our SNS topics to live
    // in here. These topics can then be passed to the ProcessorStack later,
    // creating a cross-stack reference.
    const topicsStack = new cdk.Stack(app, "TopicsStack");

    // Create the topic the stack we're testing will reference.
    const topics = [new sns.Topic(topicsStack, "Topic1", {})];

    // Create the ProcessorStack.
    const processorStack = new ProcessorStack(app, "ProcessorStack", {
      topics: topics, // Cross-stack reference
    });

    // Prepare the stack for assertions.
    const template = Template.fromStack(processorStack);
```

------
#### [ Python ]

```
from aws_cdk import aws_sns as sns
import aws_cdk as cdk
from aws_cdk.assertions import Template

from app.processor_stack import ProcessorStack

def test_synthesizes_properly():
    app = cdk.App()

    # Since the ProcessorStack consumes resources from a separate stack
    # (cross-stack references), we create a stack for our SNS topics to live
    # in here. These topics can then be passed to the ProcessorStack later,
    # creating a cross-stack reference.
    topics_stack = cdk.Stack(app, "TopicsStack")

    # Create the topic the stack we're testing will reference.
    topics = [sns.Topic(topics_stack, "Topic1")]

    # Create the ProcessorStack.
    processor_stack = ProcessorStack(
        app, "ProcessorStack", topics=topics  # Cross-stack reference
    )

    # Prepare the stack for assertions.
    template = Template.from_stack(processor_stack)
```

------
#### [ Java ]

```
package software.amazon.samples.awscdkassertionssamples;

import org.junit.jupiter.api.Test;
import software.amazon.awscdk.assertions.Capture;
import software.amazon.awscdk.assertions.Match;
import software.amazon.awscdk.assertions.Template;
import software.amazon.awscdk.App;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.services.sns.Topic;

import java.util.*;

import static org.assertj.core.api.Assertions.assertThat;

public class ProcessorStackTest {
    @Test
    public void testSynthesizesProperly() {
        final App app = new App();

        // Since the ProcessorStack consumes resources from a separate stack (cross-stack references), we create a stack
        // for our SNS topics to live in here. These topics can then be passed to the ProcessorStack later, creating a
        // cross-stack reference.
        final Stack topicsStack = new Stack(app, "TopicsStack");

        // Create the topic the stack we're testing will reference.
        final List<Topic> topics = Collections.singletonList(Topic.Builder.create(topicsStack, "Topic1").build());

        // Create the ProcessorStack.
        final ProcessorStack processorStack = new ProcessorStack(
                app,
                "ProcessorStack",
                topics // Cross-stack reference
        );

        // Prepare the stack for assertions.
        final Template template = Template.fromStack(processorStack)
```

------
#### [ C\# ]

```
using Microsoft.VisualStudio.TestTools.UnitTesting;

using Amazon.CDK;
using Amazon.CDK.AWS.SNS;
using Amazon.CDK.Assertions;
using AwsCdkAssertionSamples;

using ObjectDict = System.Collections.Generic.Dictionary<string, object>;
using StringDict = System.Collections.Generic.Dictionary<string, string>;

namespace TestProject1
{
    [TestClass]
    public class ProcessorStackTest
    {
        [TestMethod]
        public void TestMethod1()
        {
            var app = new App();

            // Since the ProcessorStack consumes resources from a separate stack (cross-stack references), we create a stack
            // for our SNS topics to live in here. These topics can then be passed to the ProcessorStack later, creating a
            // cross-stack reference.
            var topicsStack = new Stack(app, "TopicsStack");

            // Create the topic the stack we're testing will reference.
            var topics = new Topic[] { new Topic(topicsStack, "Topic1") };

            // Create the ProcessorStack.
            var processorStack = new StateMachineStack(app, "ProcessorStack", new StateMachineStackProps
            {
                Topics = topics
            });

            // Prepare the stack for assertions.
            var template = Template.FromStack(processorStack);
            
            // test will go here
        }
    }
}
```

------

Now we can assert that the Lambda function and the Amazon SNS subscription were created\.

------
#### [ TypeScript ]

```
    // Assert it creates the function with the correct properties...
    template.hasResourceProperties("AWS::Lambda::Function", {
      Handler: "handler",
      Runtime: "nodejs14.x",
    });

    // Creates the subscription...
    template.resourceCountIs("AWS::SNS::Subscription", 1);
```

------
#### [ JavaScript ]

```
    // Assert it creates the function with the correct properties...
    template.hasResourceProperties("AWS::Lambda::Function", {
      Handler: "handler",
      Runtime: "nodejs14.x",
    });

    // Creates the subscription...
    template.resourceCountIs("AWS::SNS::Subscription", 1);
```

------
#### [ Python ]

```
# Assert that we have created the function with the correct properties
    template.has_resource_properties(
        "AWS::Lambda::Function",
        {
            "Handler": "handler",
            "Runtime": "nodejs14.x",
        },
    )

    # Assert that we have created a subscription
    template.resource_count_is("AWS::SNS::Subscription", 1)
```

------
#### [ Java ]

```
        // Assert it creates the function with the correct properties...
        template.hasResourceProperties("AWS::Lambda::Function", Map.of(
                "Handler", "handler",
                "Runtime", "nodejs14.x"
        ));
          
         // Creates the subscription...
        template.resourceCountIs("AWS::SNS::Subscription", 1);
```

------
#### [ C\# ]

```
            // Prepare the stack for assertions.
            var template = Template.FromStack(processorStack);

            // Assert it creates the function with the correct properties...
            template.HasResourceProperties("AWS::Lambda::Function", new StringDict {
                { "Handler", "handler"},
                { "Runtime", "nodejs14x" }
            });

            // Creates the subscription...
            template.ResourceCountIs("AWS::SNS::Subscription", 1);
```

------

Our Lambda function test asserts that two particular properties of the function resource have specific values\. By default, the `hasResourceProperties` method performs a partial match on the resource's properties as given in the synthesized CloudFormation template\. This test requires that the provided properties exist and have the specified values, but the resource can also have other properties, and these are not tested\.

Our Amazon SNS assertion asserts that the synthesized template contains a subscription, but nothing about the subscription itself\. We included this assertion mainly to illustrate how to assert on resource counts\. The `Template` class offers more specific methods to write assertions against the `Resources`, `Outputs`, and `Mapping` sections of the CloudFormation template\. 

### Matchers<a name="testing_fine_grained_matchers"></a>

The default partial matching behavior of `hasResourceProperties` can be changed using *matchers* from the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.assertions.Match.html#methods](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.assertions.Match.html#methods) class\. 

Matchers range from the very lenient \(`Match.anyValue`\) to the quite strict \(`Match.objectEquals`\), and can be nested to apply different matching methods to different parts of the resource properties\. Using `Match.objectEquals` and `Match.anyValue` together, for example, we can test the state machine's IAM role more fully, while not requiring specific values for properties that may change\.

------
#### [ TypeScript ]

```
    // Fully assert on the state machine's IAM role with matchers.
    template.hasResourceProperties(
      "AWS::IAM::Role",
      Match.objectEquals({
        AssumeRolePolicyDocument: {
          Version: "2012-10-17",
          Statement: [
            {
              Action: "sts:AssumeRole",
              Effect: "Allow",
              Principal: {
                Service: {
                  "Fn::Join": [
                    "",
                    ["states.", Match.anyValue(), ".amazonaws.com"],
                  ],
                },
              },
            },
          ],
        },
      })
    );
```

------
#### [ JavaScript ]

```
    // Fully assert on the state machine's IAM role with matchers.
    template.hasResourceProperties(
      "AWS::IAM::Role",
      Match.objectEquals({
        AssumeRolePolicyDocument: {
          Version: "2012-10-17",
          Statement: [
            {
              Action: "sts:AssumeRole",
              Effect: "Allow",
              Principal: {
                Service: {
                  "Fn::Join": [
                    "",
                    ["states.", Match.anyValue(), ".amazonaws.com"],
                  ],
                },
              },
            },
          ],
        },
      })
    );
```

------
#### [ Python ]

```
from aws_cdk.assertions import Match

    # Fully assert on the state machine's IAM role with matchers.
    template.has_resource_properties(
        "AWS::IAM::Role",
        Match.object_equals(
            {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Action": "sts:AssumeRole",
                            "Effect": "Allow",
                            "Principal": {
                                "Service": {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "states.",
                                            Match.any_value(),
                                            ".amazonaws.com",
                                        ],
                                    ],
                                },
                            },
                        },
                    ],
                },
            }
        ),
    )
```

------
#### [ Java ]

```
        // Fully assert on the state machine's IAM role with matchers.
        template.hasResourceProperties("AWS::IAM::Role", Match.objectEquals(
                Collections.singletonMap("AssumeRolePolicyDocument", Map.of(
                        "Version", "2012-10-17",
                        "Statement", Collections.singletonList(Map.of(
                                "Action", "sts:AssumeRole",
                                "Effect", "Allow",
                                "Principal", Collections.singletonMap(
                                        "Service", Collections.singletonMap(
                                                "Fn::Join", Arrays.asList(
                                                        "",
                                                        Arrays.asList("states.", Match.anyValue(), ".amazonaws.com")
                                                )
                                        )
                                )
                        ))
                ))
        ));
```

------
#### [ C\# ]

```
            // Fully assert on the state machine's IAM role with matchers.
            template.HasResource("AWS::IAM::Role", Match.ObjectEquals(new ObjectDict
            {
                { "AssumeRolePolicyDocument", new ObjectDict
                    {
                        { "Version", "2012-10-17" },
                        { "Action", "sts:AssumeRole" },
                        { "Principal", new ObjectDict
                            {
                                { "Version", "2012-10-17" },
                                { "Statement", new object[]
                                    {
                                        new ObjectDict {
                                            { "Action", "sts:AssumeRole" },
                                            { "Effect", "Allow" },
                                            { "Principal", new ObjectDict
                                                {
                                                    { "Service", new ObjectDict
                                                        {
                                                            { "", new object[]
                                                                { "states", Match.AnyValue(), ".amazonaws.com" }
                                                            }
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }));
```

------

Many CloudFormation resources include serialized JSON objects represented as strings\. The `Match.serializedJson()` matcher can be used to match properties inside this JSON\. For example, Step Functions state machines are defined using a string in the JSON\-based [Amazon States Language](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html)\. We'll use `Match.serializedJson()` to make sure our initial state is the only step, again using nested matchers to apply different kinds of matching to different parts of the object\. 

------
#### [ TypeScript ]

```
    // Assert on the state machine's definition with the Match.serializedJson()
    // matcher.
    template.hasResourceProperties("AWS::StepFunctions::StateMachine", {
      DefinitionString: Match.serializedJson(
        // Match.objectEquals() is used implicitly, but we use it explicitly
        // here for extra clarity.
        Match.objectEquals({
          StartAt: "StartState",
          States: {
            StartState: {
              Type: "Pass",
              End: true,
              // Make sure this state doesn't provide a next state -- we can't
              // provide both Next and set End to true.
              Next: Match.absent(),
            },
          },
        })
      ),
    });
```

------
#### [ JavaScript ]

```
    // Assert on the state machine's definition with the Match.serializedJson()
    // matcher.
    template.hasResourceProperties("AWS::StepFunctions::StateMachine", {
      DefinitionString: Match.serializedJson(
        // Match.objectEquals() is used implicitly, but we use it explicitly
        // here for extra clarity.
        Match.objectEquals({
          StartAt: "StartState",
          States: {
            StartState: {
              Type: "Pass",
              End: true,
              // Make sure this state doesn't provide a next state -- we can't
              // provide both Next and set End to true.
              Next: Match.absent(),
            },
          },
        })
      ),
    });
```

------
#### [ Python ]

```
    # Assert on the state machine's definition with the serialized_json matcher.
    template.has_resource_properties(
        "AWS::StepFunctions::StateMachine",
        {
            "DefinitionString": Match.serialized_json(
                # Match.object_equals() is the default, but specify it here for clarity
                Match.object_equals(
                    {
                        "StartAt": "StartState",
                        "States": {
                            "StartState": {
                                "Type": "Pass",
                                "End": True,
                                # Make sure this state doesn't provide a next state --
                                # we can't provide both Next and set End to true.
                                "Next": Match.absent(),
                            },
                        },
                    }
                )
            ),
        },
    )
```

------
#### [ Java ]

```
        // Assert on the state machine's definition with the Match.serializedJson() matcher.
        template.hasResourceProperties("AWS::StepFunctions::StateMachine", Collections.singletonMap(
                "DefinitionString", Match.serializedJson(
                        // Match.objectEquals() is used implicitly, but we use it explicitly here for extra clarity.
                        Match.objectEquals(Map.of(
                                "StartAt", "StartState",
                                "States", Collections.singletonMap(
                                        "StartState", Map.of(
                                                "Type", "Pass",
                                                "End", true,
                                                // Make sure this state doesn't provide a next state -- we can't provide
                                                // both Next and set End to true.
                                                "Next", Match.absent()
                                        )
                                )
                        ))
                )
        ));
```

------
#### [ C\# ]

```
            // Assert on the state machine's definition with the Match.serializedJson() matcher
            template.HasResourceProperties("AWS::StepFunctions::StateMachine", new ObjectDict
            {
                { "DefinitionString", Match.SerializedJson(
                    // Match.objectEquals() is used implicitly, but we use it explicitly here for extra clarity.
                    Match.ObjectEquals(new ObjectDict {
                        { "StartAt", "StartState" },
                        { "States", new ObjectDict
                        {
                            { "StartState", new ObjectDict {
                                { "Type", "Pass" },
                                { "End", "True" },
                                // Make sure this state doesn't provide a next state -- we can't provide
                                // both Next and set End to true.
                                { "Next", Match.Absent() }
                            }}
                        }}
                    })    
                )}});
```

------

### Capturing<a name="testing_fine_grained_capture"></a>

It's often useful to test properties to make sure they follow specific formats, or have the same value as another property, without needing to know their exact values ahead of time\. The `assertions` module provides this capability in its [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.assertions.Capture.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.assertions.Capture.html) class\.

By specifying a `Capture` instance in place of a value in `hasResourceProperties`, that value is retained in the `Capture` object\. The actual captured value can be retrieved using the object's `as` methods, including `asNumber()`, `asString()`, and `asObject`, and subjected to test\. Use `Capture` with a matcher to specify the exact location of the value to be captured within the resource's properties, including serialized JSON properties\.

For example, this example tests to make sure that the starting state of our state machine has a name beginning with `Start` and also that this state is actually present within the list of states in the machine\.

------
#### [ TypeScript ]

```
    // Capture some data from the state machine's definition.
    const startAtCapture = new Capture();
    const statesCapture = new Capture();
    template.hasResourceProperties("AWS::StepFunctions::StateMachine", {
      DefinitionString: Match.serializedJson(
        Match.objectLike({
          StartAt: startAtCapture,
          States: statesCapture,
        })
      ),
    });

    // Assert that the start state starts with "Start".
    expect(startAtCapture.asString()).toEqual(expect.stringMatching(/^Start/));

    // Assert that the start state actually exists in the states object of the
    // state machine definition.
    expect(statesCapture.asObject()).toHaveProperty(startAtCapture.asString());
```

------
#### [ JavaScript ]

```
    // Capture some data from the state machine's definition.
    const startAtCapture = new Capture();
    const statesCapture = new Capture();
    template.hasResourceProperties("AWS::StepFunctions::StateMachine", {
      DefinitionString: Match.serializedJson(
        Match.objectLike({
          StartAt: startAtCapture,
          States: statesCapture,
        })
      ),
    });

    // Assert that the start state starts with "Start".
    expect(startAtCapture.asString()).toEqual(expect.stringMatching(/^Start/));

    // Assert that the start state actually exists in the states object of the
    // state machine definition.
    expect(statesCapture.asObject()).toHaveProperty(startAtCapture.asString());
```

------
#### [ Python ]

```
import re

    from aws_cdk.assertions import Capture

    # ...

    # Capture some data from the state machine's definition.
    start_at_capture = Capture()
    states_capture = Capture()
    template.has_resource_properties(
        "AWS::StepFunctions::StateMachine",
        {
            "DefinitionString": Match.serialized_json(
                Match.object_like(
                    {
                        "StartAt": start_at_capture,
                        "States": states_capture,
                    }
                )
            ),
        },
    )

    # Assert that the start state starts with "Start".
    assert re.match("^Start", start_at_capture.as_string())

    # Assert that the start state actually exists in the states object of the
    # state machine definition.
    assert start_at_capture.as_string() in states_capture.as_object()
```

------
#### [ Java ]

```
        // Capture some data from the state machine's definition.
        final Capture startAtCapture = new Capture();
        final Capture statesCapture = new Capture();
        template.hasResourceProperties("AWS::StepFunctions::StateMachine", Collections.singletonMap(
                "DefinitionString", Match.serializedJson(
                        Match.objectLike(Map.of(
                                "StartAt", startAtCapture,
                                "States", statesCapture
                        ))
                )
        ));

        // Assert that the start state starts with "Start".
        assertThat(startAtCapture.asString()).matches("^Start.+");

        // Assert that the start state actually exists in the states object of the state machine definition.
        assertThat(statesCapture.asObject()).containsKey(startAtCapture.asString());
```

------
#### [ C\# ]

```
            // Capture some data from the state machine's definition.
            var startAtCapture = new Capture();
            var statesCapture = new Capture();
            template.HasResourceProperties("AWS::StepFunctions::StateMachine", new ObjectDict
            {
                { "DefinitionString", Match.SerializedJson(
                    new ObjectDict
                    {
                        { "StartAt", startAtCapture },
                        { "States", statesCapture }
                    }
                )}
            });

            Assert.IsTrue(startAtCapture.ToString().StartsWith("Start"));
            Assert.IsTrue(statesCapture.AsObject().ContainsKey(startAtCapture.ToString()));
```

------

## Snapshot tests<a name="testing_snapshot"></a>

In *snapshot testing*, you compare the entire synthesized CloudFormation template against a previously\-stored master\. This isn't useful in catching regressions, as fine\-grained assertions are, because it applies to the entire template, and things besides code changes can cause small \(or not\-so\-small\) differences in synthesis results\. For example, we may update a CDK construct to incorporate a new best practice, which can cause changes to the synthesized resources or how they're organized, or we might update the CDK Toolkit to report additional metadata\. Changes to context values can also affect the synthesized template\. 

Snapshot tests can be of great help in refactoring, though, as long as you hold constant all other factors that might affect the synthesized template\. You will know immediately if a change you made has unintentionally changed the template\. If the change is intentional, simply accept a new master and proceed\.

For example, if we have this `DeadLetterQueue` construct:

------
#### [ TypeScript ]

```
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

------
#### [ JavaScript ]

```
class DeadLetterQueue extends sqs.Queue {
  
  constructor(scope, id) {
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

module.exports = { DeadLetterQueue }
```

------
#### [ Python ]

```
class DeadLetterQueue(sqs.Queue):
    def __init__(self, scope: Construct, id: str):
        super().__init__(scope, id)

        self.messages_in_queue_alarm = cloudwatch.Alarm(
            self,
            "Alarm",
            alarm_description="There are messages in the Dead Letter Queue.",
            evaluation_periods=1,
            threshold=1,
            metric=self.metric_approximate_number_of_messages_visible(),
        )
```

------
#### [ Java ]

```
public class DeadLetterQueue extends Queue {
    private final IAlarm messagesInQueueAlarm;

    public DeadLetterQueue(@NotNull Construct scope, @NotNull String id) {
        super(scope, id);

        this.messagesInQueueAlarm = Alarm.Builder.create(this, "Alarm")
                .alarmDescription("There are messages in the Dead Letter Queue.")
                .evaluationPeriods(1)
                .threshold(1)
                .metric(this.metricApproximateNumberOfMessagesVisible())
                .build();
    }

    public IAlarm getMessagesInQueueAlarm() {
        return messagesInQueueAlarm;
    }
}
```

------
#### [ C\# ]

```
namespace AwsCdkAssertionSamples
{
    public class DeadLetterQueue : Queue
    {
        public IAlarm messagesInQueueAlarm;

        public DeadLetterQueue(Construct scope, string id) : base(scope, id)
        {
            messagesInQueueAlarm = new Alarm(this, "Alarm", new AlarmProps
            { 
                AlarmDescription = "There are messages in the Dead Letter Queue.",
                EvaluationPeriods = 1,
                Threshold = 1,
                Metric = this.MetricApproximateNumberOfMessagesVisible()
            });
        }
    }
}
```

------

We can test it like this:

------
#### [ TypeScript ]

```
import { Match, Template } from "aws-cdk-lib/assertions";
import * as cdk from "aws-cdk-lib";
import { DeadLetterQueue } from "../lib/dead-letter-queue";

describe("DeadLetterQueue", () => {
  test("creates an alarm", () => {
    const stack = new cdk.Stack();
    new DeadLetterQueue(stack, "DeadLetterQueue");

    const template = Template.fromStack(stack);
    template.hasResourceProperties("AWS::CloudWatch::Alarm", {
      Namespace: "AWS/SQS",
      MetricName: "ApproximateNumberOfMessagesVisible",
      Dimensions: [
        {
          Name: "QueueName",
          Value: Match.anyValue(),
        },
      ],
    });
  });
});
```

------
#### [ JavaScript ]

```
const { Match, Template } = require("aws-cdk-lib/assertions");
const cdk = require("aws-cdk-lib");
const { DeadLetterQueue } = require("../lib/dead-letter-queue");

describe("DeadLetterQueue", () => {
  test("creates an alarm", () => {
    const stack = new cdk.Stack();
    new DeadLetterQueue(stack, "DeadLetterQueue");

    const template = Template.fromStack(stack);
    template.hasResourceProperties("AWS::CloudWatch::Alarm", {
      Namespace: "AWS/SQS",
      MetricName: "ApproximateNumberOfMessagesVisible",
      Dimensions: [
        {
          Name: "QueueName",
          Value: Match.anyValue(),
        },
      ],
    });
  });
});
```

------
#### [ Python ]

```
import aws_cdk as cdk
from aws_cdk.assertions import Match, Template

from app.dead_letter_queue import DeadLetterQueue

def test_creates_alarm():
    stack = cdk.Stack()
    DeadLetterQueue(stack, "DeadLetterQueue")

    template = Template.from_stack(stack)
    template.has_resource_properties(
        "AWS::CloudWatch::Alarm",
        {
            "Namespace": "AWS/SQS",
            "MetricName": "ApproximateNumberOfMessagesVisible",
            "Dimensions": [
                {
                    "Name": "QueueName",
                    "Value": Match.any_value(),
                },
            ],
        },
    )
```

------
#### [ Java ]

```
package software.amazon.samples.awscdkassertionssamples;

import org.junit.jupiter.api.Test;
import software.amazon.awscdk.assertions.Match;
import software.amazon.awscdk.assertions.Template;
import software.amazon.awscdk.Stack;

import java.util.Collections;
import java.util.Map;

public class DeadLetterQueueTest {
    @Test
    public void testCreatesAlarm() {
        final Stack stack = new Stack();
        new DeadLetterQueue(stack, "DeadLetterQueue");

        final Template template = Template.fromStack(stack);
        template.hasResourceProperties("AWS::CloudWatch::Alarm", Map.of(
                "Namespace", "AWS/SQS",
                "MetricName", "ApproximateNumberOfMessagesVisible",
                "Dimensions", Collections.singletonList(Map.of(
                        "Name", "QueueName",
                        "Value", Match.anyValue()
                ))
        ));
    }
}
```

------
#### [ C\# ]

```
using Microsoft.VisualStudio.TestTools.UnitTesting;

using Amazon.CDK;
using Amazon.CDK.Assertions;
using AwsCdkAssertionSamples;

using ObjectDict = System.Collections.Generic.Dictionary<string, object>;
using StringDict = System.Collections.Generic.Dictionary<string, string>;

namespace TestProject1
{
    [TestClass]
    public class ProcessorStackTest

    [TestClass]
    public class DeadLetterQueueTest
    {
    [TestMethod]
        public void TestCreatesAlarm()
        {
            var stack = new Stack();
            new DeadLetterQueue(stack, "DeadLetterQueue");

            var template = Template.FromStack(stack);
            template.HasResourceProperties("AWS::CloudWatch::Alarm", new ObjectDict
            {
                { "Namespace", "AWS/SQS" },
                { "MetricName", "ApproximateNumberOfMessagesVisible" },
                { "Dimensions", new object[]
                    {
                        new ObjectDict
                        {
                            { "Name", "QueueName" },
                            { "Value", Match.AnyValue() }
                        }
                    }
                }
            });
        }
    }
}
```

------

## Tips for tests<a name="testing_tips"></a>

Remember, your tests will live just as long as the code they test, and be read and modified just as often, so it pays to take a moment to consider how best to write them\. Don't copy and paste setup lines or common assertions, for example; refactor this logic into fixtures or helper functions\. Use good names that reflect what each test actually tests\.

Don't try to do too much in one test\. Preferably, a test should test one and only one behavior\. If you accidentally break that behavior, exactly one test should fail, and the name of the test should tell you exactly what failed\. This is more an ideal to be striven for, however; sometimes you will unavoidably \(or inadvertently\) write tests that test more than one behavior\. Snapshot tests are, for reasons we've already described, especially prone to this problem, so use them sparingly\.