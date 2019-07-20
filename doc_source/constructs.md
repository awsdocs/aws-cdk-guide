# Constructs<a name="constructs"></a>

Constructs are the basic building blocks of AWS CDK apps\. A construct represents a "cloud component" and encapsulates everything AWS CloudFormation needs to create the component\. 

A construct can represent a single resource, such as an Amazon Simple Storage Service \(Amazon S3\) bucket, or it can represent a higher\-lever component consisting of multiple AWS CDK resources\. Examples of such components include a worker queue with its associated compute capacity, a cron job with monitoring resources and a dashboard, or even an entire app spanning multiple AWS accounts and regions\.

## AWS Construct Library<a name="constructs_lib"></a>

The AWS CDK includes the [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html), which contains constructs representing AWS resources\.

This library includes constructs that represent all the resources available on AWS\. For example, the `[s3\.Bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html)` class represents an Amazon S3 bucket, and the `[dynamodb\.Table](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-dynamodb.Table.html)` class represents an Amazon DynamoDB table\.

There are different levels of constructs in this library, beginning with low\-level constructs, which we call *CFN Resources*\. These constructs represent all of the AWS resources that are available in AWS CloudFormation\. CFN Resources are generated from the [AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html) on a regular basis\. They are named **Cfn***Xyz*, where *Xyz* represents the name of the resource\. For example, [s3\.CfnBucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.CfnBucket.html) represents the [AWS::S3::Bucket](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html) CFN Resource\. When you use CFN constructs, you must explicitly configure all resource properties, which requires a complete understanding of the details of the underlying resource model\.

The next level of constructs also represent AWS resources, but with a higher\-level, intent\-based API\. They provide the same functionality, but handle much of the details, boilerplate, and glue logic required by CFN constructs\. AWS constructs offer convenient defaults and reduce the need to know all the details about the AWS resources they represent, while providing convenience methods that make it simpler to work with the resource\. For example, the [s3\.Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html#aws_s3_Bucket) class represents an Amazon S3 bucket with additional properties and methods, such as [bucket\.addLifeCycleRule\(\)](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html#aws_s3_Bucket_addLifecycleRule), which adds a lifecycle rule to the bucket\.

Finally, the AWS Construct Library includes even higher\-level constructs, which we call *patterns*\. These constructs are designed to help you complete common tasks in AWS, often involving multiple kinds of resources\. For example, the [aws\-ecs\-patterns\.LoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecs-patterns.LoadBalancedFargateService.html) construct represents an architecture that includes an AWS Fargate container cluster employing Elastic Load Balancing \(ELB\)\. The [aws\-apigateway\.LambdaRestApi](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigateway.LambdaRestApi.html) construct represents an Amazon API Gateway API that's backed by an AWS Lambda function\.

For more information about how to navigate the library and discover constructs that can help you build your apps, see the [API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)\.

## Composition<a name="constructs_composition"></a>

The key pattern for defining higher\-level abstractions through constructs is called *composition*\. A high\-level construct can be composed from any number of lower\-level constructs, and in turn, those could be composed from even lower\-level constructs\. To enable this pattern, constructs are always defined within the scope of another construct\. This scoping pattern results in a hierarchy of constructs known as a *construct tree*\. In the AWS CDK, the root of the tree represents your entire **[AWS CDK app](https://docs.aws.amazon.com/cdk/latest/guide/apps_and_stacks.html#apps)**\. Within the app, you typically define one or more **[stacks](https://docs.aws.amazon.com/cdk/latest/guide/apps_and_stacks.html#stacks)**, which are the unit of deployment, analogous to AWS CloudFormation stacks\. Within stacks, you define resources, or other constructs that eventually contain resources\.

Composition of constructs means that you can define reusable components and share them like any other code\. For example, a central team can define a construct that implements the company's best practice for a DynamoDB table with backup, global replication, auto\-scaling, and monitoring, and share it with teams across a company or publicly\. Teams can now use this construct as they would any other library package in their favorite programming language to define their tables and comply with their team's best practices\. When the library is updated, developers can pick up the updates and enjoy any bug fixes and improvements through the workflows they already have for their other types of code\.

## Initialization<a name="constructs_init"></a>

Constructs are implemented in classes that extend the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html) base class\. You define a construct by instantiating the class\. All constructs take three parameters when they are initialized:
+ **scope** – The construct within which this construct is defined\. You should almost always pass `this` for the scope, because it represents the current scope in which you are defining the construct\.
+ **id** – An [identifier](identifiers.md) that must be unique within this scope\. The identifier serves as a namespace for everything that's encapsulated within the scope's subtree and is used to allocate unique identities such as [resource names](resources.md#resources_physical_names) and AWS CloudFormation logical IDs\.
+ **props** – A set of properties or keyword arguments, depending upon the supported language, that define the construct's initial configuration\. In most cases, constructs provide sensible defaults, and if all props elements are optional, you can leave out the **props** parameter completely\.

Identifiers need only be unique within a scope\. This lets you instantiate and reuse constructs without concern for the constructs and identifiers they might contain, and enables composing constructs into higher level abstractions\. In addition, scopes make it possible to refer to groups of constructs all at once, for example for [tagging](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/tag.html) or for specifying where the constructs will be deployed\.

## Apps and Stacks<a name="constructs_apps_stacks"></a>

We call your CDK application an *app*, which is represented by the AWS CDK class [App](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/app.html)\. The following example defines an app with a single stack that contains a single Amazon S3 bucket with versioning enabled:

```
import { App, Stack, StackProps } from '@aws-cdk/core';
import s3 = require('@aws-cdk/aws-s3');

class HelloCdkStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}

const app = new App();
new HelloCdkStack(app, "HelloCdkStack");
```

As you can see, you need a scope within which to define your bucket\. Since resources eventually need to be deployed as part of a AWS CloudFormation stack into an *AWS *[https://docs.aws.amazon.com/cdk/latest/guide/apps_and_stacks.html#environments](https://docs.aws.amazon.com/cdk/latest/guide/apps_and_stacks.html#environments), which covers a specific AWS account and AWS region\. AWS constructs, such as `s3.Bucket`, must be defined within the scope of a [https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/stack.html#cdk_Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/stack.html#cdk_Stack)\.

Stacks in AWS CDK apps extend the **Stack** base class, as shown in the previous example\. This is a common pattern when creating a stack within your AWS CDK app: extend the **Stack** class, define a constructor that accepts **scope**, **id**, and **props**, and invoke the base class constructor via `super` with the received **scope**, **id**, and **props**, as shown in the following example\.

```
class HelloCdkStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

    //...
  }
}
```

## Using Constructs<a name="constructs_using"></a>

Once you have defined a stack, you can populate it with resources\. The following example imports the [Amazon S3](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-readme.html) module and uses it to define a new Amazon S3 bucket by creating an instance of the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) class within the current scope \(`this`\) which, in our case is the `HelloCdkStack` instance\.

```
import s3 = require('@aws-cdk/aws-s3');

// "this" is HelloCdkStack
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

The [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) includes constructs that represent many AWS resources\.

**Note**  
`MyFirstBucket` is not the name of the bucket that AWS CloudFormation creates\. It is a logical identifier given to the new construct\. See [Physical Names](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/resource.html#core_Resource_physicalName) for details\.

## Configuration<a name="constructs_config"></a>

Most constructs accept `props` as their third initialization argument\. This is a name/value collection that defines the construct’s configuration\. The following example defines a bucket with AWS Key Management Service \(AWS KMS\) encryption and static website hosting enabled\. Since it does not explicitly specify an encryption key, the `Bucket` construct defines a new `kms.Key` and associates it with the bucket\.

```
new s3.Bucket(this, 'MyEncryptedBucket', {
  encryption: s3.BucketEncryption.Kms,
  websiteIndexDocument: 'index.html'
});
```

 AWS constructs are designed around the concept of "sensible defaults\." Most constructs have a minimal required configuration, enabling you to quickly get started while also providing full control over the configuration when you need it\. 

## Interacting with Constructs<a name="constructs_interact"></a>

Constructs are classes that extend the base [Construct](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/construct.html) class\. After you instantiate a construct, the construct object exposes a set of methods and properties that enable you to interact with the construct and pass it around as a reference to other parts of the system\. The AWS CDK framework doesn't put any restrictions on the APIs of constructs; authors can define any API they wish\. However, the AWS constructs that are included with the AWS Construct Library, such as `s3.Bucket`, follow guidelines and common patterns in order to provide a consistent experience across all AWS resources\.

For example, almost all AWS constructs have a set of [grant](permissions.md#permissions_grants) methods that you can use to grant AWS Identity and Access Management \(IAM\) permissions on that construct to a principal\. The following example grants the IAM group `data-science` permission to read from the Amazon S3 bucket `raw-data`\.

```
const rawData = new s3.Bucket(this, 'raw-data');
const dataScience = new iam.Group(this, 'data-science');
rawData.grantRead(dataScience);
```

Another common pattern is for AWS constructs to set one of the resource’s attributes, such as its Amazon Resource Name \(ARN\), name, or URL from data supplied elsewhere\. For example, the following code defines an AWS Lambda function and associates it with an Amazon Simple Queue Service \(Amazon SQS\) queue through the queue's URL in an environment variable\.

```
const jobsQueue = new sqs.Queue(this, 'jobs');
const createJobLambda = new lambda.Function(this, 'create-job', {
  runtime: lambda.Runtime.NODEJS_8_10,
  handler: 'index.handler',
  code: lambda.Code.asset('./create-job-lambda-code'),
  environment: {
    QUEUE_URL: jobsQueue.queueUrl
  }
});
```

For information about the most common API patterns in the AWS Construct Library, see [Resources](https://docs.aws.amazon.com/cdk/latest/guide/resources.html)\.

## Authoring Constructs<a name="constructs_author"></a>

In addition to using existing constructs like `s3.Bucket`, you can also author your own constructs, and then anyone can use them in their apps\. All constructs are equal in the AWS CDK\. An AWS CDK construct such as `s3.Bucket` or `sns.Topic` behaves the same as a construct imported from a third\-party library that someone published on npm or Maven or PyPI—or to your company's internal package repository\.

To declare a new construct, create a class that extends the [Construct](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/construct.html) base class, then follow the pattern for initializer arguments\.

For example, you could declare a construct that represents an Amazon S3 bucket which sends an Amazon Simple Notification Service \(Amazon SNS\) notification every time someone uploads a file into it:

```
export interface NotifyingBucketProps {
  prefix?: string;
}

export class NotifyingBucket extends Construct {
  constructor(scope: Construct, id: string, props: NotifyingBucketProps = {}) {
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    const topic = new sns.Topic(this, 'topic');
    bucket.onObjectCreated(topic, { prefix: props.prefix }); 
  }
}
```

The `NotifyingBucket` constructor has the same signature as the base `Construct` class: `scope`, `id`, and `props`\. The last argument, `props`, is optional \(gets the default value `{}`\) because all props are optional\. This means that you could define an instance of this construct in your app without `props`, for example: 

```
new NotifyingBucket(this, 'MyNotifyingBucket');
```

Or you could use `props` to specify the path prefix to filter on, for example:

```
new NotifyingBucket(this, 'MyNotifyingBucket', { prefix: 'images/' });
```

Typically, you would also want to expose some properties or methods from your constructs\. For example, it's not very useful to have a topic hidden behind your construct, because it wouldn't be possible to subscribe to it\. Adding a `topic` property allows consumers to access the inner topic, as shown in the following example:

```
export class NotifyingBucket extends Construct {
  public readonly topic: sns.Topic;

  constructor(scope: Construct, id: string, props: NotifyingBucketProps) {
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    this.topic = new sns.Topic(this, 'topic');
    bucket.onObjectCreated(this.topic, { prefix: props.prefix });
  }
}
```

Now, consumers can subscribe to the topic, for example:

```
const queue = new sqs.Queue(this, 'NewImagesQueue');
const images = new NotificationBucket(this, 'Images');
images.topic.addSubscription(new sns_sub.QueueSubscription(queue));
```
