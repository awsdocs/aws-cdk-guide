--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\.

--------

# Constructs<a name="constructs"></a>

Constructs can be viewed as "cloud components". They can represent architectures of any complexity. They can represent a single resource, such as an Amazon S3 Bucket or an Amazon SNS Topic, reusable components such as a static website, a part of a specific application and up to complex multi-stack applications that span multiple accounts and region. Everything in the AWS CDK is a construct.

This composition of constructs also means you can easily create sharable constructs, and make changes to any construct and have those changes available to consumers as shared class libraries\.

## The AWS Construct Library

The AWS CDK is shipped with a rich class library of constructs called the **AWS Construct Library**. This library consists of constructs that represent all the resources available on AWS.

Each module in the AWS Construct Library includes two types of constructs for each resource: low-level and high-level.

The low-level resources are automatically generated from the [AWS CloudFormation Resource Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html). Low-level constructs are named `CfnXyz` (where "Xyz" represents the name of the resource) and provide direct, one-to-one access to how a resource is synthesized in the CloudFormation template produced by your CDK app. Using low-level resources require explicit configuration of all resource properties, IAM policies and requires deep understanding of the details.

High-level resources are hand-written by AWS and offer a rich, intent-based API for using AWS services. They offer the same functionality as the low-level resources but encode much of the details, boilerplate and glue-logic required in order to use AWS. High-level resources offer convenient defaults and additional knowledge about the inner workings of the AWS resources they represent\. In general, you will be able to express your intent without worrying about the details too much, and the correct resources and configuration will automatically be defined for you\.  

Similarly to the AWS SDKs and AWS CloudFormation, the AWS Construct Library is organized into modules, one for each AWS service. For example, the `@aws-cdk/aws-ec2` module includes resources for EC2 instances, Auto Scaling Groups and VPCs, the aws-sns module includes resources such as Topic and Subscription, and so forth. See the [Reference](https://awslabs.github.io/aws-cdk/reference.html#reference) section for descriptions of the AWS CDK packages and constructs\.

AWS Construct Library members are found in the `@aws-cdk/aws-NAMESPACE` packages, where NAMESPACE is the short name for the associated service, such as SQS for the AWS Construct Library for the Amazon SQS service\.

## Construct Structure<a name="constructs_structure"></a>

Constructs are represented as normal classes in your code and are defined by instantiating an object of that class.

When constructs are initialized, they are always defined within the _**scope**_ of another construct and they always have an _**id**_ which must be unique within the same scope.

For example, here's how you would define an SNS Topic in your stack with default configuration:

```ts
new sns.Topic(this, 'MyTopic');
```

The first argument to every construct is always the scope in which it is created, and will almost always be simply `this` because most constructs will be defined within the current scope.

Scopes enable constructs to be composed together to form higher-level abstractions by enabling the framework to group them together into logical units, allocate globally unique identifiers and allow them to consult __context information__ such as the region in which it is going to be deployed into, which AZs are available for my account, etc.

In most cases, the construct initializer will have a third `props` argument that can be used to define the construct's initial configuration. For example:

```ts
new BeautifulConstruct(this, 'Foo', {
  favoriteColor: 'green',
  timeout: 300
});
```

The `construct.node` property can be used to interact with the construct's node representation:

* `construct.node.scope` The scope in which this construct was defined.

* `construct.node.id` Returns the "id" of the construct passed upon creation.

* `construct.node.uniqueId` Returns an app-wide unique safe ID of this construct. This ID encodes the construct's path into a human readable portion as well as a hash of the full path to ensure global uniqueness.

* `construct.node.path` Gets the path of this construct from the root of scope (the `App`)\.

## Construct IDs<a name="constructs_ids"></a>

Every construct in a CDK app must have an **id** unique within the scope in which the construct is defined. IDs are used to find constructs in the construct hierarchy, and to allocate logical IDs so that AWS CloudFormation can keep track of the generated resources\.

When a construct is created, its ID is specified as the second initializer argument:

```
const c1 = new MyBeautifulConstruct(this, 'OneBeautiful');
const c2 = new MyBeautifulConstruct(this, 'TwoBeautiful');
assert(c1.node.id === 'OneBeautiful');
assert(c2.node.id === 'TwoBeautiful');
```

Note that the ID of a construct does not directly map onto the physical name of the resource when it is created\. If you want to give a physical name to a bucket or table, specify the physical name using use the appropriate property, such as `bucketName` or `tableName`, as shown in the following example:

```
new s3.Bucket(this, 'MyBucket', {
  bucketName: 'physical-bucket-name'
});
```

In general it's recommended to avoid specifying physical names\. Instead, let AWS CloudFormation generate names for you\. Use attributes, such as `bucket.bucketName`, to discover the generated names\.

When you synthesize an AWS CDK app into an AWS CloudFormation template, the AWS CloudFormation logical ID for each resource in the template is allocated according to the path of that resource in the scope hierarchy\. For more information, see [Logical IDs](logical_ids.md)\.

## Construct Properties<a name="constructs_properties"></a>

Customize constructs by passing a property object as the third parameter \(*props*\)\. Every construct has its own set of parameters, defined as an interface\. You can pass a property object to your construct in two ways:

```
// Inline (recommended)
new sqs.Queue(this, 'MyQueue', {
  visibilityTimeout: 300
});
```

## Construct Metadata<a name="constructs_metadata"></a>

You can attach metadata to a construct using the `aws-cdk.Construct.node.addMetadata` operation\. Metadata entries automatically include the stack trace from which the metadata entry was added to allow tracing back to your code, even if the entry was was defined by a lower\-level library that you don't own\.
