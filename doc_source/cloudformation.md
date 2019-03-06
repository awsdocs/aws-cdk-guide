--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS CloudFormation Library<a name="cloudformation"></a>

The [AWS Construct Library](aws_construct_lib.md)includes constructs with APIs for defining AWS infrastructure\. For example, you can use the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct to define Amazon Simple Storage Service \(Amazon S3\) buckets, and the [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) construct to define Amazon Simple Notification Service \(Amazon SNS\) topics\.

Under the hood, these constructs are implemented using AWS CloudFormation resources, which are available in the `CfnXxx` classes in each library\. For example, the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct uses the [@aws\-cdk/aws\-s3\.cloudformation\.BucketResource](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.cloudformation.BucketResource) resource \(as well as other resources, depending on what bucket APIs are used\)\.

**Important**  
Generally, when building CDK apps, you shouldn't need to interact with AWS CloudFormation directly\. However, there might be advanced use cases and migration scenarios where this might be required\. We are also aware that there might be gaps in capabilities in the AWS Construct Library over time\.

## Resources<a name="cloudformation_resources"></a>

The CDK creates the low\-level resources from the [AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html) on a regular basis\. The classes are available under the `CfnXxx` classes of each AWS library\. Their API matches 1:1 with how you would use these resources in AWS CloudFormation\.

When defining AWS CloudFormation resources, the `props` argument of the class initializer matches 1:1 to the resource's properties in AWS CloudFormation\.

For example, to define an [AWS::SQS::Queue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html) resource encrypted with an AWS managed key, you can directly specify the [KmsMasterKeyId](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-sqs-queue-kmsmasterkeyid) property\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.CfnQueue(this, 'MyQueueResource', {
    kmsMasterKeyId: 'alias/aws/sqs'
});
```

For reference, if you use the [Queue](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sqs.html#@aws-cdk/aws-sqs.Queue) construct, you can define managed queue encryption as follows\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KmsManaged
});
```

## Resource Object Properties<a name="cloudformation_resource_object"></a>

Use resource object properties to get a runtime attribute of an AWS CloudFormation resource\.

The following example configures the dead letter queue of an AWS Lambda function to use the Amazon Resource Name \(ARN\) of an Amazon SQS queue resource\.

```
import sqs = require('@aws-cdk/aws-sqs');
import lambda = require('@aws-cdk/aws-lambda');
      
const dlq = new sqs.CfnQueue(this, { name: 'DLQ' });
      
new lambda.CfnFunction(this, {
    deadLetterConfig: {
        targetArn: dlq.queueArn
    }
});
```

The [cdk\.Resource\.ref](@cdk-class-url;#@aws-cdk/cdk.Resource.ref) attribute represents the AWS CloudFormation resource's intrinsic reference \(or *return value*\)\. For example, *dlq\.ref* also [refers](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-properties-sqs-queues-ref) to the queue's ARN\. When possible, use an explicitly named attribute instead of *ref*\.

## Resource Options<a name="cloudformation_resource_options"></a>

For resources, the [cdk\.Resource\.options](@cdk-class-url;#@aws-cdk/cdk.Resource.options) object includes AWS CloudFormation options, such as `condition`, `updatePolicy`, `createPolicy`, and `metadata`\.