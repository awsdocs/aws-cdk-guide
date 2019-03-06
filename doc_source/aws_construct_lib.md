--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS Construct Library<a name="aws_construct_lib"></a>

The AWS Construct Library is a set of modules that expose APIs for defining AWS resources in AWS CDK apps\. Each module is based on the AWS service to which the resource belongs\. For example, the [@aws\-cdk/aws\-ec2](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#module-@aws-cdk/aws-ec2) module includes the [@aws\-cdk/aws\-ec2\.VpcNetwork](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#@aws-cdk/aws-ec2.VpcNetwork) construct, which makes it easy to define an [Amazon VPC](https://aws.amazon.com/vpc) in your CDK app\.

The AWS Construct Library modules are described in the [CDK Reference](https://awslabs.github.io/aws-cdk/)\.

The AWS Construct Library includes many common patterns and capabilities that are designed to enable developers to focus on their application\-specific architectures and reduce the boilerplate and glue logic needed when working with AWS services\.

## Grants<a name="aws_construct_lib_grants"></a>

AWS Identity and Access Management \(IAM\) policies are automatically defined based on intent\. For example, when subscribing an Amazon Simple Notification Service \(Amazon SNS\) [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) to an AWS Lambda [Function](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.Function), the function's IAM permission policy is automatically modified to allow the specific topic to invoke the function\.

Also, most AWS constructs expose `grant*` methods that allow intent\-based permission definitions\. For example, the Amazon S3 [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct has a [grantRead\(principal\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.grantRead) method\. This method accepts an IAM [Principal](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#iprincipal-interface) such as a [User](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#user) or a [Role](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#role), which modifies the policy to allow the principal to read objects from the bucket\.

## Security Groups<a name="aws_construct_lib_security_groups"></a>

Amazon Elastic Compute Cloud \(Amazon EC2\) network entities, such as the [Elastic Load Balancing](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-elasticloadbalancingv2.html) and [Auto Scaling Group](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-autoscaling.html#auto-scaling-group) instances, can connect to each other based on definitions of security groups\.

The CDK provides APIs for defining security group connections\. For more information, see [Allowing Connections](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#allowing-connections)\.

## Metrics<a name="aws_construct_lib_metrics"></a>

Many AWS resources emit [Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) as part of their normal operation\. Metrics can be used to set up an [Alarm](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Alarm) or can be included in a [Dashboard](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Dashboard)\.

You can obtain [Metric](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Metric) objects for AWS constructs by using `metricXxx()` methods\. For example, the [metricDuration\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.FunctionRef.metricDuration) method reports the execution time of an AWS Lambda function\.

For more information, see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html)\.

## Exports and Imports<a name="aws_construct_exports_and_imports"></a>

If you need to reference a resource, such as an Amazon S3 bucket or VPC, that's defined outside of your CDK app, you can use the `Xxxx.import(...)` static methods that are available on AWS constructs\. For example, you can use the [Bucket\.import\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.import) method to obtain a [BucketRef](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef) object, which can be used in most places where a bucket is required\. This pattern enables treating resources defined outside of your app as if they are part of your app\.

## Event\-Driven APIs<a name="aws_construct_lib_event_driven"></a>

Many AWS constructs include `on*` methods, which can be used to react to events emitted by the construct\. For example, the CodeCommit [Repository](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#Repository) construct has an [onCommit](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#@aws-cdk/aws-codecommit.RepositoryRef.onCommit) method\. You can use AWS constructs as targets for various event provider interfaces, such as [IEventRuleTarget](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html#ieventruletarget-interface) \(for the CloudWatch event rule target\), [IAlarmAction](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#ialarmaction-interface) \(for Amazon CloudWatch alarm actions\), and so on\.

For more information, see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html) and [@aws\-cdk/aws\-events](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html)\.