--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS CloudFormation<a name="cloudformation"></a>

The [AWS Construct Library](aws_construct_lib.md) includes constructs with APIs for defining AWS infrastructure\. For example, you can use the [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) construct to define Amazon Simple Storage Service \(Amazon S3\) buckets, and the [Topic](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-sns/topic.html) construct to define Amazon Simple Notification Service \(Amazon SNS\) topics\.

Under the hood, these constructs are implemented using AWS CloudFormation resources, which are available in the `CfnXxx` classes in each library\. For example, the [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) construct uses the [CfnBucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/cfnbucket.html) resource \(as well as other resources, depending on what bucket APIs are used\)\.

**Important**  
Avoid interacting directly with AWS CloudFormation unless absolutely necessary, such as when an AWS Construct Library construct does not provide the needed functionality\.