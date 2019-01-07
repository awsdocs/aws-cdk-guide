--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# AWS Construct Library<a name="cdk_aws_construct_lib"></a>

The AWS Construct Library is a set of modules which expose APIs for defining AWS resources in AWS CDK apps\. The AWS Construct Library is organized in modules based on the AWS service to which the resource belongs\. For example, the [@aws\-cdk/aws\-ec2](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#module-@aws-cdk/aws-ec2) module includes the [@aws\-cdk/aws\-ec2\.VpcNetwork](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#@aws-cdk/aws-ec2.VpcNetwork) construct which makes it easy to define an [Amazon VPC](https://aws.amazon.com/vpc) in your AWS CDK app\.

The AWS Construct Library includes many common patterns and capabilities which are designed to allow developers to focus on their application\-specific architectures and reduces the boilerplate and glue logic needed when working with AWS\.

## Least\-Privilege IAM Policies<a name="cdk_aws_construct_lib_least-privilege"></a>

IAM policies are automatically defined based on intent\. For example, when subscribing an Amazon SNS [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) to a Lambda [Function](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.Function) the function's IAM permission policy is automatically modified to allow the specific topic to invoke the function\.

Furthermore, most AWS Constructs expose `grant*` methods which allow intent\-based permission definitions\. For example, the Amazon S3 [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct has a [grantRead\(principal\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.grantRead) method, which accepts an IAM [Principal](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#iprincipal-interface) such as a [User](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#user) or a [Role](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#role) which modifies the policy to allow the principal to read objects from the bucket\.

## Event\-Driven APIs<a name="cdk_aws_construct_lib_event_driven"></a>

Many AWS constructs include `on*` methods, which can be used to react to events emitted by the construct\. For example, the AWS CodeCommit [Repository](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#Repository) construct has an [onCommit](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#@aws-cdk/aws-codecommit.RepositoryRef.onCommit) method\. AWS Constructs that can be used as targets for various event providers implement interfaces such as [IEventRuleTarget](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html#ieventruletarget-interface) \(for CloudWatch Event Rule target\), [IAlarmAction](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#ialarmaction-interface) \(for AWS CloudWatch Alarm actions\), etc\.

For more information see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html) and [@aws\-cdk/aws\-events](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html)\.

## Security Groups<a name="cdk_aws_construct_lib_security_groups"></a>

Amazon EC2 network entities such as the [Elastic Load Balancing](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-elasticloadbalancingv2.html) and [Auto Scaling Group](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-autoscaling.html#auto-scaling-group) instances can connect to each other based on definitions of security groups\.

The AWS CDK provides APIs for defining security group connections\. For more information, see [Allowing Connections](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#allowing-connections)\.

## Metrics<a name="cdk_aws_construct_lib_metrics"></a>

Many AWS resources emit AWS CloudWatch metrics as part of their normal operation\. Metrics can be used to setup an [Alarm](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Alarm) or included in a [Dashboard](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Dashboard)\.

[Metric](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Metric) objects for AWS Constructs can be obtained using `metricXxx()` methods\. For example, the [metricDuration\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.FunctionRef.metricDuration) method reports the execution time of an AWS Lambda function\.

For more information see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html)\.

## Imports<a name="cdk_aws_construct_imports"></a>

If you need to reference a resource, such as an Amazon S3 bucket or VPC, which is defined outside of your AWS CDK app, you can use the `Xxxx.import(...)` static methods that are available on AWS Constructs\. For example, the [Bucket\.import\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.import) method can be used to obtain a [BucketRef](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef) object, which can be used in most places where a bucket is required\. This patterns allows treating resources defined outside your app as if they were part of your app\.