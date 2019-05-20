--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Resources<a name="resources"></a>

The CDK creates the low\-level resources from the [AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html) on a regular basis\. The classes are available under the `CfnXxx` classes of each AWS library\. Their API matches 1:1 with how you would use these resources in AWS CloudFormation\.

When defining AWS CloudFormation resources, the `props` argument of the class initializer matches 1:1 to the resource's properties in AWS CloudFormation\.

For example, to define an [AWS::SQS::Queue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html) resource encrypted with an AWS managed key, you can directly specify the [KmsMasterKeyId](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-sqs-queue-kmsmasterkeyid) property\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.CfnQueue(this, 'MyQueueResource', {
    kmsMasterKeyId: 'alias/aws/sqs'
});
```

For reference, if you use the [Queue](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-sqs/queue.html) construct, you can define managed queue encryption as follows\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KmsManaged
});
```

## Resource Object Properties<a name="resources_object"></a>

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

The [cdk\.CfnReference](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/cfnreference.html) attribute represents the AWS CloudFormation resource's intrinsic reference \(or *return value*\)\. For example, *dlq\.ref* also [refers](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-sqs-queues.html#aws-properties-sqs-queues-ref) to the queue's ARN\. When possible, use an explicitly named attribute instead of *ref*\.

## Resource Options<a name="cloudformation_resource_options"></a>

For resources, the [CfnResource\.options](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/cfnresource.html#cdk_CfnResource_options) object includes AWS CloudFormation options, such as `condition`, `updatePolicy`, `createPolicy`, and `metadata`\.