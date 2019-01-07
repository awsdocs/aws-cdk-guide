--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Constructs<a name="cdk_constructs"></a>

Constructs are the building blocks of AWS CDK applications\. Constructs can have child constructs, which in turn can have child constructs, forming a hierarchical tree structure\.

The AWS CDK includes two different levels of constructs:

CloudFormation Resource  
These constructs are low\-level constructs that provide a direct, one\-to\-one, mapping to an AWS CloudFormation resource, as listed in the AWS CloudFormation topic [ AWS Resource Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.   
All CloudFormation Resource members are found in the `@aws-cdk/resources` package\.

AWS Construct Library  
These constructs have been handwritten by AWS and come with convenient defaults and additional knowledge about the inner workings of the AWS resources they represent\. In general, you will be able to express your intent without worrying about the details too much, and the correct resources will automatically be defined for you\.  
AWS Construct Library members are found in the `@aws-cdk/aws-NAMESPACE` packages, where NAMESPACE is the short name for the associated service, such as SQS for the AWS Construct Library for the Amazon SQS service\. See the [Reference](https://awslabs.github.io/aws-cdk/reference.html#reference) section for descriptions of the AWS CDK packages and constructs\.

## Construct Structure<a name="cdk_constructs_structure"></a>

The construct tree structure is a powerful design pattern for composing high\-level abstractions\. For example, you can define a `StorageLayer` construct that represents your application's storage layer and include all the AWS resources, such as DynamoDB tables and Amazon S3 buckets, needed to implement your storage layer in this construct\. When your higher\-level code uses this construct, it only needs to instantiate the `StorageLayer` construct\.

When you initialize a construct, add the construct to the construct tree by specifying the parent construct as the first initializer parameter, an identifier for the construct as the second parameter, and a set of properties for the final parameter, as shown in the following example\.

```
new SomeConstruct(parent, name[, props]);
```

In almost all cases, you should pass the keyword `this` for the `parent` argument, because you will generally initialize new constructs in the context of the parent construct\. Any descriptive string will do for the `name` argument, and an in\-line object for the set of properties\.

```
new BeautifulConstruct(this, 'Foo', {
    applicationName: 'myApp',
    timeout: 300
});
```

**Note**  
Associating the construct to its parent as part of initialization is necessary because the construct occasionally needs contextual information from its parent, such as to which the region the stack is deployed\.

Use the following operations to inspect the construct tree\.

 aws\-cdk\.Construct\.parent   
Gets the path of this construct from the root of the tree\.

 aws\-cdk\.Construct\.getChildren   
Gets an array of all of the contruct's children\.

 aws\-cdk\.Construct\.getChild   
Gets the child construct with the specified ID\.

 aws\-cdk\.Construct\.toTreeString\(\)   
Gets a string representing the construct's tree\.

## Construct Names<a name="cdk_constructs_EVER"></a>

Every construct in a CDK app must have a **name** unique among its siblings\. Names are used to track constructs in the construct hierarchy, and to allocate logical IDs so that AWS CloudFormation can keep track of the generated resources\.

When a construct is created, its name is specified as the second initializer argument:

```
const c1 = new MyBeautifulConstruct(this, 'OneBeautiful');
const c2 = new MyBeautifulConstruct(this, 'TwoBeautiful');
assert(c1.name === 'OneBeautiful');
assert(c2.name === 'TwoBeautiful');
```

Use the `aws-cdk.Construct.path` property to get the path of this construct from the root of the tree\.

Note that the name of a construct does not directly map onto the physical name of the resource when it is created\. If you want to give a physical name to a bucket or table, specify the physical name using use the appropriate property, such as `bucketName` or `tableName`, as shown in the following example:

```
new Bucket(this, 'MyBucket', {
    bucketName: 'physical-bucket-name'
});
```

Avoid specifying physical names\. Instead, let AWS CloudFormation generate names for you\. Use attributes, such as `bucket.bucketName`, to discover the generated names\.

When you synthesize an AWS CDK tree into an AWS CloudFormation template, the AWS CloudFormation logical ID for each resource in the template is allocated according to the path of that resource in the construct tree\. For more information, see [Logical IDs](cdk_logical_ids.md)\.

## Construct Properties<a name="cdk_constructs_properties"></a>

Customize constructs by passing a property object as the third parameter \(*props*\)\. Every construct has its own set of parameters, defined as an interface\. You can pass a property object to your construct in two ways:

```
// Inline (recommended)
new Queue(this, 'MyQueue', {
  visibilityTimeout: 300
});

// Instantiate separate property object
const props: QueueProps = {
  visibilityTimeout: 300
};

new Queue(this, 'MyQueue', props);
```

## Construct Metadata<a name="cdk_constructs_metadata"></a>

You can attach metadata to a construct using the `aws-cdk.Construct.addMetadata` operation\. Metadata entries automatically include the stack trace from which the metadata entry was added\. Therefore, at any level of a construct you can find the code location, even if metadata was created by a lower\-level library that you don't own\.