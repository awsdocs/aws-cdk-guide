--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# AWS AWS CloudFormation Library<a name="cdk_cloudformation"></a>

The [AWS Construct Library](cdk_aws_construct_lib.md) includes constructs with APIs for defining AWS infrastructure\. For example, the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct can be used to define Amazon S3 Buckets and the [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) construct can be used to define Amazon SNS Topics\.

Under the hood, these constructs are implemented using AWS CloudFormation resources, which are available in the `CfnXxx` classes in each library\. For example, the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct uses the [@aws\-cdk/aws\-s3\.cloudformation\.BucketResource](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.cloudformation.BucketResource) resource \(as well as other resources, depending on what bucket APIs are used\)\.

**Important**  
Generally, when building AWS CDK apps, you shouldn't need to interact with AWS CloudFormation directly\. However, there might be advanced use cases and migration scenarios where this might be required\. We are also aware that there might be gaps in capabilities in the AWS Construct Library over time\.

## Resources<a name="cdk_cloudformation_resources"></a>

AWS CloudFormation resource classes are automatically generated from the [AWS AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html) and available under the `CfnXxx` classes of each AWS library\. Their API matches 1:1 with how you would use these resources in AWS CloudFormation\.

When defining AWS CloudFormation resource, the `props` argument of the class initializer will match 1:1 to the resource's properties in AWS CloudFormation\.

For example, to define an [AWS::SQS::Queue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html) resource encrypted with an AWS managed key you can directly specify the [KmsMasterKeyId](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-sqs-queue-kmsmasterkeyid) property\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.CfnQueue(this, 'MyQueueResource', {
    kmsMasterKeyId: 'alias/aws/sqs'
});
```

For reference, if you use the [Queue](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sqs.html#@aws-cdk/aws-sqs.Queue) construct, you can define managed queue encryption as follows:

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KmsManaged
});
```

## Resource Options<a name="cdk_cloudformation_resource_options"></a>

To reference the runtime attributes of AWS CloudFormation resources, use one of the properties available on the resource object\.

The following example configures a Lambda function's dead letter queue to use the ARN of an Amazon SQS queue resource\.

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

The [cdk\.Resource\.ref](@cdk-class-url;#@aws-cdk/cdk.Resource.ref) attribute represents the AWS CloudFormation resource's intrinsic reference \(or *Return Value*\)\. For example, *dlq\.ref* also [refers](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-properties-sqs-queues-ref) to the queue's ARN\. When possible, it is preferrable to use an explicitly named attribute instead of *ref*\.

## Resource Options<a name="cdk_cloudformation_resource_options"></a>

The [cdk\.Resource\.options](@cdk-class-url;#@aws-cdk/cdk.Resource.options) object includes AWS CloudFormation options, such as `condition`, `updatePolicy`, `createPolicy` and `metadata`, for a resource\.

## Parameters<a name="cdk_cloudformation_parameters"></a>

```
import sns = require('@aws-cdk/aws-sns');
import cdk = require('@aws-cdk/cdk');
      
const p = new cdk.Parameter(this, 'MyParam', { type: 'String' });
new sns.CfnTopic(this, 'MyTopic', { displayName: p.ref });
```

## Outputs<a name="cdk_cloudformation_outputs"></a>

```
import sqs = require('@aws-cdk/aws-sqs');
import cdk = require('@aws-cdk/cdk');
      
const queue = new sqs.CfnQueue(this, 'MyQueue');
const out = new cdk.Output(this, 'MyQueueArn', { value: queue.queueArn });
      
const import = out.makeImportValue();
assert(import === { "Fn::ImportValue": out.exportName }
```

## Conditions<a name="cdk_cloudformation_conditions"></a>

```
import sqs = require('@aws-cdk/aws-sqs');
import cdk = require('@aws-cdk/cdk');
const cond = new cdk.Condition(this, 'MyCondition', {
    expression: new cdk.FnIf(...)
});
      
const queue = new sqs.CfnQueue(this, 'MyQueue');
queue.options.condition = cond;
```

## Intrinsic Functions<a name="cdk_cloudformation_intrinsic_functions"></a>

```
import { Fn } from'@aws-cdk/cdk';
Fn.join(",", [...])
```

## Pseudo Parameters<a name="cdk_cloudformation_pseudo_parameters"></a>

```
import cdk = require('@aws-cdk/cdk');
new cdk.AwsRegion()
      

cdk synch > mytemplate
into a CI/CD pipeline
```