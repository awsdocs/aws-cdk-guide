--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS CloudFormation<a name="cloudformation"></a>

The [AWS Construct Library](aws_construct_lib.md)includes constructs with APIs for defining AWS infrastructure\. For example, you can use the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct to define Amazon Simple Storage Service \(Amazon S3\) buckets, and the [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) construct to define Amazon Simple Notification Service \(Amazon SNS\) topics\.

Under the hood, these constructs are implemented using AWS CloudFormation resources, which are available in the `CfnXxx` classes in each library\. For example, the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct uses the [@aws\-cdk/aws\-s3\.cloudformation\.BucketResource](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.cloudformation.BucketResource) resource \(as well as other resources, depending on what bucket APIs are used\)\.

**Important**  
Avoid interacting directly with AWS CloudFormation unless absolutely necessary, such as when a CDK AWS Construct Library does not provide the needed functionality\.