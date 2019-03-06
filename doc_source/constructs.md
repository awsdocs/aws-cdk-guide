--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Constructs<a name="constructs"></a>

You can think of constructs as *cloud components*\. They can represent architectures of any complexity\. They can represent a single resource, such as an Amazon Simple Storage Service \(Amazon S3\) bucket or an Amazon Simple Notification Service \(Amazon SNS\) topic\. They can represent reusable components, such as a static website, a part of a specific application, or complex, multistack applications that span multiple accounts and AWS Regions\. Constructs can also include other constructs\. Everything in the AWS CDK is a construct\.

This composition of constructs means that you can create sharable constructs\. For example, if construct A and construct B use construct C and you make changes to construct C, then and both construct A and construct B get those changes\.

## The AWS CloudFormation Resource Library<a name="constructs_l1"></a>

The AWS CDK provides a class library of constructs called the **AWS CloudFormation Resource Library**\. This library consists of constructs that represent all the resources available on AWS\.

Each module in the AWS Construct Library includes two types of constructs for each resource: low\-level constructs known as an AWS CloudFormation Resource constructs and high\-level constructs known as an AWS Construct Library constructs\.

The CDK creates the low\-level resources from the [AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html) on a regular basis\. Low\-level constructs are named **Cfn***Xyz*, where *Xyz* represents the name of the resource\. These constructs provide direct, one\-to\-one access to how a resource is synthesized in the AWS CloudFormation template produced by your CDK app\. Using low\-level resources requires you to explicitly configure all resource properties, IAM policies, and have a deep understanding of the details\.

High\-level resource constructs are authored by AWS and offer an intent\-based API for using AWS services\. They provide the same functionality as the low\-level resources, but encode much of the details, boilerplate, and glue logic required to use AWS\. High\-level resources offer convenient defaults and additional knowledge about the inner workings of the AWS resources they represent\. 

Similarly to the AWS SDKs and AWS CloudFormation, the AWS Construct Library is organized into modules, one for each AWS service\. For example, the `@aws-cdk/aws-ec2` module includes resources for Amazon EC2 instances and networking\. The `aws-sns` module includes resources such as `Topic` and `Subscription`\. See the [Reference](https://awslabs.github.io/aws-cdk/reference.html) for descriptions of the CDK packages and constructs\.

AWS Construct Library members are found in the `@aws-cdk/aws-NAMESPACE` packages, where `NAMESPACE` is the short name for the associated service, such as `SQS` for the AWS Construct Library for the Amazon Simple Queue Service \(Amazon SQS\) service\.

## Construct Structure<a name="constructs_structure"></a>

Constructs are represented as normal classes in your code and are defined by instantiating an object of that class\.

When constructs are initialized, they are always defined within the *scope* of another construct, and always have an *id* that must be unique within the same scope\.

For example, here's how you would define an Amazon SNS `topic` in your stack with default configuration\.

```
new sns.Topic(this, 'MyTopic');
```

The first argument to every construct is always the scope in which it's created, and is almost always `this`, because most constructs are defined within the current scope\.

Scopes enable constructs to be composed together to form higher\-level abstractions\. This is done by enabling the framework to group them together into logical units, allocate globally unique identifiers, and allow them to consult context information, such as the AWS Region in which it's going to be deployed and which availability Zones are available for your account\.

In most cases, the construct initializer has a third `props` argument that can be used to define the construct's initial configuration\. For example:

```
new MyConstruct(this, 'Foo', {
  favoriteColor: 'green',
  timeout: 300
});
```

Use the `construct.node` property to get the following information about the construct\.

 construct\.node\.scope   
Gets the scope in which the construct was defined\.

 construct\.node\.id   
Gets the `id` of the construct\.

 construct\.node\.uniqueId   
Gets the app\-wide unique, safe ID of the construct\. This ID encodes the construct's path into a human\-readable portion and a hash of the full path to ensure global uniqueness\.

 construct\.node\.path   
Gets the full path of this construct from the root of the scope \(the `App`\)\.

## Construct IDs<a name="constructs_ids"></a>

Every construct in a CDK app must have an `id` that's unique within the scope in which the construct is defined\. The CDK uses IDs to find constructs in the construct hierarchy\. It also uses IDs to allocate logical IDs so that AWS CloudFormation can keep track of the generated resources\.

When a construct is created, its ID is specified as the second initializer argument\.

```
const c1 = new MyConstruct(this, 'OneConstruct');
const c2 = new MyConstruct(this, 'TwoConstruct');
assert(c1.node.id === 'OneConstruct');
assert(c2.node.id === 'TwoConstruct');
```

Notice that the ID of a construct doesn't directly map to the physical name of the resource when it's created\. To give a physical name to a bucket or table, specify the physical name using the appropriate property, such as `bucketName` or `tableName`, as shown in the following example\.

```
new s3.Bucket(this, 'MyBucket', {
  bucketName: 'physical-bucket-name'
});
```

We recommend that you avoid specifying physical names\. Instead, let AWS CloudFormation generate names for you\. Use attributes, such as `bucket.bucketName`, to discover the generated names\.

When you synthesize a CDK app into an AWS CloudFormation template, the AWS CloudFormation logical ID for each resource in the template is allocated according to the path of that resource in the scope hierarchy\. For more information, see [Logical IDs](logical_ids.md)\.

## Construct Properties<a name="constructs_properties"></a>

Customize constructs by passing a property object as the third parameter \(*props*\)\. Every construct has its own set of properties, defined as an interface\. You can pass a property object to your construct in two ways: inline, or instantiated as a separate property object\.

```
// Inline (recommended)
new sqs.Queue(this, 'MyQueue', {
  visibilityTimeout: 300
});

// Instantiate separate property object
const props: QueueProps = {
  visibilityTimeout: 300
};

new Queue(this, 'MyQueue', props);
```

## Construct Metadata<a name="constructs_metadata"></a>

Attach metadata to a construct using the `addMetadata` method\. Metadata is an AWS CDK\-level annotation, and as such, does not appear in the deployed resources\. Metadata entries automatically include the stack trace from which the metadata entry was added to allow tracing back to your code, even if the entry was defined by a lower\-level library that you don't own\.

Use the `addWarning()` method to emit a message when you you synthesis a stack; use the `addError()` method to not only emit a message when you you synthesis a stack, but to also block the deployment of a stack\.

The following example blocks the deployment of `myStack` if it is not in `us-west-2`:

```
if (myStack.region !== 'us-west-2') {
    myStack.node.addError('myStack is not in us-west-2');
}
```

## Tagging Constructs<a name="constructs_tagging"></a>

You can add a tag to any construct to identify the resources you create\. Tags can be applied to any construct\. Tags are inherited, and are based on scope\. If you tag construct `A`, and construct `A` contains construct `B`, construct `B` inherits the tag\.

There are two tag operations\.

Tag  
Adds \(or applies\) a tag to a set of resources, or to all but a set of resources\.

RemoveTag  
Removes a tag from a set of resources, or from all but a set of resources\.

The following example adds the tag key\-value pair *StackType*\-*TheBest* to any resource created within the **theBestStack** stack labeled *MarketingSystem*\.

```
import cdk = require('@aws-cdk/cdk');

const app = new cdk.App();
const theBestStack = new cdk.Stack(app, 'MarketingSystem');
theBestStack.node.apply(new cdk.Tag('StackType', 'TheBest'));
    
// To remove the tag:
theBestStack.node.apply(new cdk.RemoveTag('TheBest'));
    
// To remove the tag from all EXCEPT the subnets:
theBestStack.node.apply(new cdk.RemoveTag('TheBest'), {exludeResourceTypes: ['AWS::EC2::Subnet']}));
```

The tag operations include some properties to fine\-tune how tags are applied to or removed from the resources that the construct creates\.

applyToLaunchedInstances  
Use this Boolean property to set `PropagateAtLaunch` for any Auto Scaling group resource the construct creates\. The default is `true`\.

includeResourceTypes  
Use this array of strings to apply a tag only to those AWS CloudFormation resource types\. The default is an empty array, which means the tag applies to all AWS CloudFormation resource types\.

excludeResourceTypes  
Use this array of strings to exclude a tag from those AWS CloudFormation resource types\. The default is an empty array, which means the tag applies to all AWS CloudFormation resource types\. This property takes precedence over the `includeResourceTypes` property\.

priority  
Set this integer value to control the precedence of tags\. The default is 0 \(zero\) for `Tag` and 1 for `RemoveTag`\. Higher values take precedence over lower values\.