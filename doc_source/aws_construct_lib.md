--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS Construct Library<a name="aws_construct_lib"></a>

The AWS Construct Library is a set of modules that expose APIs for defining AWS resources in AWS CDK apps\. Each module is based on the AWS service to which the resource belongs\. For example, [EC2](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ec2-readme.html) includes the [Vpc](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-ec2/vpc.html) construct, which makes it easy to define an Amazon VPC in your AWS CDK app\.

The AWS Construct Library modules are described in the [AWS CDK Reference](https://awslabs.github.io/aws-cdk/)\.

## Versioning<a name="aws_construct_lib_versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and adhere to the [Semantic Versioning](https://semver.org) model\. This means that breaking changes to stable APIs \(see “The AWS CDK Stability Index”\) are limited to major releases\. Minor and patch releases are backwards\-compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version and be expected to continue to build and run, producing the same output\.

### AWS CDK Stability Index<a name="aws_construct_lib_versioning_stability"></a>

However, certain APIs do not adher to the semantic versioning model\. The AWS CDK team has added a stability index to help developers determine which APIs deviate from the semantic versioning model\.

There are three levels of stability in the AWS CDK Construct Library:

Stable  
The API is subject to the semantic versioning model\. We will not introduce non\-backward compatible changes or remove the API in a subsequent patch or feature release\.

CloudFormation Only  
These APIs are automatically built from the AWS CloudFormation resource specification and are subject to any changes introduced by AWS CloudFormation\.

Experimental  
The API is still under active development and subject to non\-backward compatible changes or removal in any future version\. We recommend that you do not use this API in production environments\. Experimental APIs are not subject to the semantic versioning model\.

Deprecated  
The API may emit warnings\. We do not guarantee backward compatibility\.

Experimental and stable modules receive the same level of support from AWS\. The only difference is that we might change experimental APIs within a major version\. While we don not recommend using experimental APIs in production, we vet them the same way as we vet stable APIs before we include them in an AWS CDK release\.

### Identifying the Support Level of an API<a name="aws_construct_lib_versioning_support"></a>

The AWS CDK Developer guide and API Reference for each module starts with a section outlining the module's stability index\. The libraries that include only AWS CloudFormation Resources, and no hand\-curated constructs, are labelled with the maturity indicator **CloudFormation\-only**\.

The module\-level gives an indication of the stability of the majority of the APIs included in the module, however individual APIs within the module can be annotated with different stability levels:


| Stability | TypeScript | JavaScript | Python | C\#/\.NET | Java | 
| --- |--- |--- |--- |--- |--- |
| Experimental | @experimental | @stability Experimental | @experimental | Stability: Experimental | Stability: Experimental | 
| Stable | @stable | @stability Stable | @stable | Stability: Stable | Stability: Stable | 
| Deprecated | @deprecated | @Deprecated | @deprecated | \[Obsolete\] | Stability: Deprecated | 

## AWS CDK Patterns<a name="aws_construct_lib_patterns"></a>

The AWS Construct Library includes many common patterns and capabilities that are designed to enable developers to focus on their application\-specific architectures and reduce the boilerplate and glue logic needed when working with AWS services\.

### Grants<a name="aws_construct_lib_grants"></a>

AWS Identity and Access Management \(IAM\) policies are automatically defined based on intent\. For example, when subscribing an Amazon Simple Notification Service \(Amazon SNS\) [Topic](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-sns/topic.html) to an AWS Lambda [Function](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-lambda/function.html), the function's IAM permission policy is automatically modified to allow the specific topic to invoke the function\.

Also, most AWS constructs expose `grant*` methods that allow intent\-based permission definitions\. For example, the Amazon S3 [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) construct has a [grantRead](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grantreadidentity-objectskeypattern) method\. This method accepts an IAM [IPrincipal](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-iam/iprincipal.html) such as a [User](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-iam/user.html) or a [Role](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-iam/role.html), which modifies the policy to allow the principal to read objects from the bucket\.

## Metrics<a name="aws_construct_lib_metrics"></a>

Many AWS resources emit [Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) as part of their normal operation\. Metrics can be used to set up an [Alarm](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-cloudwatch/alarm.html) or can be included in a [Dashboard](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-cloudwatch/dashboard.html)\.

You can obtain [Metric](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-cloudwatch/metric.html) objects for AWS constructs by using `metricXxx()` methods\. For example, the [metricAllDuration](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-lambda/function.html#aws_lambda_Function_metricAllDuration) method reports the execution time of an AWS Lambda function\.

For more information, see [CloudWatch](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cloudwatch-readme.html)\.

## Events<a name="aws_construct_lib_events"></a>

Many AWS constructs include `on*` methods, which you can to react to events emitted by the construct\. For example, the CodeCommit [Repository](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-codecommit/repository.html) construct implements the [onCommit](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-codecommit/irepository.html#aws_codecommit_IRepository_onCommit) method\. You can use AWS constructs as targets for various event provider interfaces, such as [IEventRuleTarget](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-events/ieventruletarget.html) \(for the CloudWatch event rule target\), [IAlarmAction](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-cloudwatch/ialarmaction.html) \(for Amazon CloudWatch alarm actions\), and so on\.

For more information, see [CloudWatch](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cloudwatch-readme.html) and [Events](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-events-readme.html)\.

## Referencing Resources<a name="aws_construct_lib_referencing"></a>

This section contains information about how to reference other resources, either from within the same app, or across apps\. The only caveat is that the resource must be in the same account and region\.

Many CDK classes have required properties that are CDK resource objects \(resources\)\. To satisfy these requirements, you can refer to a resource in one of two ways:
+ By passing the resource directly\.
+ By passing the resource's unique identifier\. This is typically an ARN, but it could also be an ID or a name\.

For example, an Amazon ECS service requires a reference to the cluster on which it runs; a CloudFront distribution requires a reference to the bucket containing source code\.

In the AWS CDK, all AWS Construct Library resources that expect another resource take a property that is of the interface type of that resource\. For example, the Amazon ECS service takes a property of type `cluster: ICluster;` the CloudFront distribution takes a property of type `sourceBucket: IBucket`\.

Since every resource implements its corresponding interface, you can directly pass any resources you're defining in the same AWS CDK application, as shown in the following example, which creates a new Amazon ECS cluster\.

```
const cluster = new ecs.Cluster(this, 'Cluster', { /* ... */ });

const service = new ecs.Service(this, 'Service', {
  cluster: cluster,
  /* ... */
});
```

### Passing Resources from a Different Stack<a name="aws_construct_lib_referencing_stack"></a>

You can refer to resources in a different stack, but in the same account and region\. If you need to refer to a resource in a different account or region, see the next section\.  

```
const account = '123456789012';
const region = 'us-east-1';

const stack1 = new StackThatProvidesABucket(app, 'Stack1');

// stack2 takes a property of type IBucket
const stack2 = new StackThatExpectsABucket(app, 'Stack2', {
  bucket: stack1.bucket
});
```

If the resource is in the same account and region, but in a different stack, the AWS CDK creates the relevant information, such as the bucket name, that is necessary to transfer that information from one stack to the other\.

### Passing Resources from a Different Account or Region<a name="aws_construct_lib_referencing_other"></a>

You can refer to a resource in a different account or region by using the resource's unique indentifier, such as:
+ `bucketName` for `bucket`
+ `functionArn` for `lambda`
+ `securityGroupId` for `securityGroup`

The following example shows how to pass a generated bucket name to a Lambda function\.

```
const bucket = new s3.Bucket(this, 'Bucket');

new lambda.Function(this, 'MyLambda', {
  /* ... */,
  environment: {
    BUCKET_NAME: bucket.bucketName,
  },
});
```

### Turning Unique Identifiers into Objects<a name="aws_construct_lib_referencing_ids"></a>

If you have the unique identifier for a resource, such as a bucket ARN, but you need to pass it to a AWS CDK construct that expects an object, you can turn the ARN into a AWS CDK object in the current stack by calling a static factory method on the resource's class\. The following example shows how to create a bucket from an existing bucket ARN\.

```
// Construct a resource (bucket) by its full ARN (can be cross account)
Bucket.fromBucketArn(this, 'Bucket', 'arn:aws:s3:::my-bucket-name');
```