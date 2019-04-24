--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS Construct Library<a name="aws_construct_lib"></a>

The AWS Construct Library is a set of modules that expose APIs for defining AWS resources in AWS CDK apps\. Each module is based on the AWS service to which the resource belongs\. For example, the [@aws\-cdk/aws\-ec2](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#module-@aws-cdk/aws-ec2) module includes the [@aws\-cdk/aws\-ec2\.VpcNetwork](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#@aws-cdk/aws-ec2.VpcNetwork) construct, which makes it easy to define an [Amazon VPC](https://aws.amazon.com/vpc) in your CDK app\.

The AWS Construct Library modules are described in the [CDK Reference](https://awslabs.github.io/aws-cdk/)\.

## Versioning<a name="aws_construct_lib_versioning"></a>

The CDK follows the semantic versioning model\. This means that breaking changes are limited to major releases, such as 2\.0\.

Minor releases, such as 2\.4, guarantee that any code written in a previous minor version, such as 2\.1, will build, run, and produce the exact same results when built, run, and producing results, as before\.

## CDK Patterns<a name="aws_construct_lib_patterns"></a>

The AWS Construct Library includes many common patterns and capabilities that are designed to enable developers to focus on their application\-specific architectures and reduce the boilerplate and glue logic needed when working with AWS services\.

### Roles<a name="aws_construct_lib_roles"></a>

Roles \.\.\.

### Grants<a name="aws_construct_lib_grants"></a>

AWS Identity and Access Management \(IAM\) policies are automatically defined based on intent\. For example, when subscribing an Amazon Simple Notification Service \(Amazon SNS\) [Topic](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-sns.html#@aws-cdk/aws-sns.Topic) to an AWS Lambda [Function](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.Function), the function's IAM permission policy is automatically modified to allow the specific topic to invoke the function\.

Also, most AWS constructs expose `grant*` methods that allow intent\-based permission definitions\. For example, the Amazon S3 [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#bucket) construct has a [grantRead\(principal\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.grantRead) method\. This method accepts an IAM [Principal](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#iprincipal-interface) such as a [User](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#user) or a [Role](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-iam.html#role), which modifies the policy to allow the principal to read objects from the bucket\.

### Resource Policies<a name="aws_construct_lib_resource_policies"></a>

Resource policies \.\.\.

### Principals<a name="aws_construct_lib_principals"></a>

Principals \.\.\.

## Metrics<a name="aws_construct_lib_metrics"></a>

Many AWS resources emit [Amazon CloudWatch metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/working_with_metrics.html) as part of their normal operation\. Metrics can be used to set up an [Alarm](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Alarm) or can be included in a [Dashboard](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Dashboard)\.

You can obtain [Metric](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#@aws-cdk/aws-cloudwatch.Metric) objects for AWS constructs by using `metricXxx()` methods\. For example, the [metricDuration\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-lambda.html#@aws-cdk/aws-lambda.FunctionRef.metricDuration) method reports the execution time of an AWS Lambda function\.

For more information, see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html)\.

## Events<a name="aws_construct_lib_events"></a>

Many AWS constructs include `on*` methods, which you can to react to events emitted by the construct\. For example, the CodeCommit [Repository](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#Repository) construct has an [onCommit](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-codecommit.html#@aws-cdk/aws-codecommit.RepositoryRef.onCommit) method\. You can use AWS constructs as targets for various event provider interfaces, such as [IEventRuleTarget](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html#ieventruletarget-interface) \(for the CloudWatch event rule target\), [IAlarmAction](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html#ialarmaction-interface) \(for Amazon CloudWatch alarm actions\), and so on\.

For more information, see [@aws\-cdk/aws\-cloudwatch](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-cloudwatch.html) and [@aws\-cdk/aws\-events](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-events.html)\.

## Referencing Constructs<a name="aws_construct_lib_referencing"></a>

This section contains information about how to reference other constructs, either from within the same app, or across apps\.

### Get a Value from Another Stack<a name="get_stack_value"></a>

You can get a value from another stack in the same app by using the `export` method in one stack and the `import` method in the other stack\.

The following example creates a bucket on one stack and passes a reference to that bucket to the other stack through an interface\.

First create a stack with a bucket\. The stack includes a property we use to pass the bucket's properties to the other stack\. Notice how we use the `export` method on the bucket to get its properties and save them in the stack property\.

```
class HelloCdkStack extends cdk.Stack {
  // Property that defines the stack you are exporting from
  public readonly myBucketImportProps: s3.BucketImportProps;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
      super(scope, id, props);

      const mybucket = new s3.Bucket(this, "MyFirstBucket");

      // Save bucket's BucketImportProps
      this.myBucketImportProps = mybucket.export();
  }
}
```

Create an interface for the second stack's properties\. We use this interface to pass the bucket properties between the two stacks\.

```
// Interface we'll use to pass the bucket's properties to another stack
interface MyCdkStackProps {
    theBucketImportProps: s3.BucketImportProps;
}
```

Create the second stack that gets a reference to the other bucket from the properties passed in through the constructor\.

```
// The class for the other stack
class MyCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyCdkStackProps) {
    super(scope, id);

    const myOtherBucket = s3.Bucket.import(this, "MyOtherBucket", props.theBucketImportProps);

    // Do something with myOtherBucket
  }
}
```

Finally, connect the dots in your app\.

```
const app = new cdk.App();

const myStack = new HelloCdkStack(app, "HelloCdkStack");

new MyCdkStack(app, "MyCdkStack", {
  theBucketImportProps: myStack.myBucketImportProps
});

app.run();
```

### Referencing Constructs Across All App<a name="aws_construct_lib_referencing_across"></a>

This section contains information about how to reference constructs from other apps\.

To reference a resource, such as an Amazon S3 bucket or VPC, that's defined outside of your CDK app, use the `Xxxx.import(...)` static methods that are available on AWS constructs\. For example, use the [Bucket\.import\(\)](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef.import) method to obtain a [BucketRef](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketRef) object, which can be used in most places where a bucket is required\. This pattern enables treating resources defined outside of your app as if they are part of your app\.

## Get a Value from Another Stack<a name="get_stack_value"></a>

You can get a value from another stack in the same app by using the `export` method in one stack and the `import` method in the other stack\.

The following example creates a bucket on one stack and passes a reference to that bucket to the other stack through an interface\.

First create a stack with a bucket\. The stack includes a property we use to pass the bucket's properties to the other stack\. Notice how we use the `export` method on the bucket to get its properties and save them in the stack property\.

```
class HelloCdkStack extends cdk.Stack {
  // Property that defines the stack you are exporting from
  public readonly myBucketImportProps: s3.BucketImportProps;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
      super(scope, id, props);

      const mybucket = new s3.Bucket(this, "MyFirstBucket");

      // Save bucket's BucketImportProps
      this.myBucketImportProps = mybucket.export();
  }
}
```

Create an interface for the second stack's properties\. We use this interface to pass the bucket properties between the two stacks\.

```
// Interface we'll use to pass the bucket's properties to another stack
interface MyCdkStackProps {
    theBucketImportProps: s3.BucketImportProps;
}
```

Create the second stack that gets a reference to the other bucket from the properties passed in through the constructor\.

```
// The class for the other stack
class MyCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyCdkStackProps) {
    super(scope, id);

    const myOtherBucket = s3.Bucket.import(this, "MyOtherBucket", props.theBucketImportProps);

    // Do something with myOtherBucket
  }
}
```

Finally, connect the dots in your app\.

```
const app = new cdk.App();

const myStack = new HelloCdkStack(app, "HelloCdkStack");

new MyCdkStack(app, "MyCdkStack", {
  theBucketImportProps: myStack.myBucketImportProps
});

app.run();
```

## Get a Value from Another App<a name="get_app_value"></a>

You can get a value from a stack in another app by ???

## Security Groups<a name="aws_construct_lib_security_groups"></a>

Amazon Elastic Compute Cloud \(Amazon EC2\) network entities, such as the [Elastic Load Balancing](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-elasticloadbalancingv2.html) and [Auto Scaling Group](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-autoscaling.html#auto-scaling-group) instances, can connect to each other based on definitions of security groups\.

The CDK provides APIs for defining security group connections\. For more information, see [Allowing Connections](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#allowing-connections)\.