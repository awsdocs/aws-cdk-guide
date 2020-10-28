# Resources<a name="resources"></a>

As described in [Constructs](constructs.md), the AWS CDK provides a rich class library of constructs, called *AWS constructs*, that represent all AWS resources\. This section describes some common patterns and best practices for how to use these constructs\.

Defining AWS resources in your CDK app is exactly like defining any other construct\. You create an instance of the construct class, pass in the scope as the first argument, the logical ID of the construct, and a set of configuration properties \(props\)\. For example, here's how to create an Amazon SQS queue with KMS encryption using the [sqs\.Queue](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-sqs.Queue.html) construct from the AWS Construct Library\.

------
#### [ TypeScript ]

```
import * as sqs from '@aws-cdk/aws-sqs';
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KMS_MANAGED
});
```

------
#### [ JavaScript ]

```
const sqs = require('@aws-cdk/aws-sqs');
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KMS_MANAGED
});
```

------
#### [ Python ]

```
import aws_cdk.aws_sqs as sqs
      
sqs.Queue(self, "MyQueue", encryption=sqs.QueueEncryption.KMS_MANAGED)
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.sqs.*;

Queue.Builder.create(this, "MyQueue").encryption(
        QueueEncryption.KMS_MANAGED).build();
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.SQS;

new Queue(this, "MyQueue", new QueueProps
{
    Encryption = QueueEncryption.KMS_MANAGED
});
```

------

Some configuration props are optional, and in many cases have default values\. In some cases, all props are optional, and the last argument can be omitted entirely\.

## Resource attributes<a name="resources_attributes"></a>

Most resources in the AWS Construct Library expose attributes, which are resolved at deployment time by AWS CloudFormation\. Attributes are exposed in the form of properties on the resource classes with the type name as a prefix\. The following example shows how to get the URL of an Amazon SQS queue using the `queueUrl` \(Python: `queue_url`\) property\.

------
#### [ TypeScript ]

```
import * as sqs from '@aws-cdk/aws-sqs';
      
const queue = new sqs.Queue(this, 'MyQueue');
const url = queue.queueUrl; // => A string representing a deploy-time value
```

------
#### [ JavaScript ]

```
const sqs = require('@aws-cdk/aws-sqs');
      
const queue = new sqs.Queue(this, 'MyQueue');
const url = queue.queueUrl; // => A string representing a deploy-time value
```

------
#### [ Python ]

```
from aws_cdk.aws_sqs as sqs

queue = sqs.Queue(self, "MyQueue")
url = queue.queue_url # => A string representing a deploy-time value
```

------
#### [ Java ]

```
Queue queue = new Queue(this, "MyQueue");
String url = queue.getQueueUrl();    // => A string representing a deploy-time value
```

------
#### [ C\# ]

```
var queue = new Queue(this, "MyQueue");
var url = queue.QueueUrl; // => A string representing a deploy-time value
```

------

See [Tokens](tokens.md) for information about how the AWS CDK encodes deploy\-time attributes as strings\.

## Referencing resources<a name="resources_referencing"></a>

Many AWS CDK classes require properties that are AWS CDK resource objects \(resources\)\. To satisfy these requirements, you can refer to a resource in one of two ways:
+ By passing the resource directly
+ By passing the resource's unique identifier, which is typically an ARN, but it could also be an ID or a name

For example, an Amazon ECS service requires a reference to the cluster on which it runs; an Amazon CloudFront distribution requires a reference to the bucket containing source code\.

If a construct property represents another AWS construct, its type is that of the interface type of that construct\. For example, the Amazon ECS service takes a property `cluster` of type `ecs.ICluster`; the CloudFront distribution takes a property `sourceBucket` \(Python: `source_bucket`\) of type `s3.IBucket`\.

Because every resource implements its corresponding interface, you can directly pass any resource object you're defining in the same AWS CDK app\. The following example defines an Amazon ECS cluster and then uses it to define an Amazon ECS service\.

------
#### [ TypeScript ]

```
const cluster = new ecs.Cluster(this, 'Cluster', { /*...*/ });

const service = new ecs.Ec2Service(this, 'Service', { cluster: cluster });
```

------
#### [ JavaScript ]

```
const cluster = new ecs.Cluster(this, 'Cluster', { /*...*/ });

const service = new ecs.Ec2Service(this, 'Service', { cluster: cluster });
```

------
#### [ Python ]

```
cluster = ecs.Cluster(self, "Cluster")

service = ecs.Ec2Service(self, "Service", cluster=cluster)
```

------
#### [ Java ]

```
Cluster cluster = new Cluster(this, "Cluster");
Ec2Service service = new Ec2Service(this, "Service",
        new Ec2ServiceProps.Builder().cluster(cluster).build());
```

------
#### [ C\# ]

```
var cluster = new Cluster(this, "Cluster");
var service = new Ec2Service(this, "Service", new Ec2ServiceProps { Cluster = cluster });
```

------

## Accessing resources in a different stack<a name="resource_stack"></a>

You can access resources in a different stack, as long as they are in the same account and AWS Region\. The following example defines the stack `stack1`, which defines an Amazon S3 bucket\. Then it defines a second stack, `stack2`, which takes the bucket from `stack1` as a constructor property\.

------
#### [ TypeScript ]

```
const prod = { account: '123456789012', region: 'us-east-1' };

const stack1 = new StackThatProvidesABucket(app, 'Stack1' , { env: prod });

// stack2 will take a property { bucket: IBucket }
const stack2 = new StackThatExpectsABucket(app, 'Stack2', {
  bucket: stack1.bucket, 
  env: prod
});
```

------
#### [ JavaScript ]

```
const prod = { account: '123456789012', region: 'us-east-1' };

const stack1 = new StackThatProvidesABucket(app, 'Stack1' , { env: prod });

// stack2 will take a property { bucket: IBucket }
const stack2 = new StackThatExpectsABucket(app, 'Stack2', {
  bucket: stack1.bucket, 
  env: prod
});
```

------
#### [ Python ]

```
prod = core.Environment(account="123456789012", region="us-east-1")

stack1 = StackThatProvidesABucket(app, "Stack1", env=prod)

# stack2 will take a property "bucket"
stack2 = StackThatExpectsABucket(app, "Stack2", bucket=stack1.bucket, env=prod)
```

------
#### [ Java ]

```
// Helper method to build an environment
static Environment makeEnv(String account, String region) {
    return Environment.builder().account(account).region(region)
            .build();
}

App app = new App();

Environment prod = makeEnv("123456789012", "us-east-1");

StackThatProvidesABucket stack1 = new StackThatProvidesABucket(app, "Stack1",
        StackProps.builder().env(prod).build());

// stack2 will take an argument "bucket"
StackThatExpectsABucket stack2 = new StackThatExpectsABucket(app, "Stack,",
        StackProps.builder().env(prod).build(), stack1.getBucket());
```

------
#### [ C\# ]

```
Amazon.CDK.Environment makeEnv(string account, string region)
{
    return new Amazon.CDK.Environment { Account = account, Region = region };
}

var prod = makeEnv(account: "123456789012", region: "us-east-1");

var stack1 = new StackThatProvidesABucket(app, "Stack1", new StackProps { Env = prod });

// stack2 will take an argument "bucket"
var stack2 = new StackThatExpectsABucket(app, "Stack2", new StackProps { Env = prod,
    bucket = stack1.Bucket});
```

------

If the AWS CDK determines that the resource is in the same account and Region, but in a different stack, it automatically synthesizes AWS CloudFormation [exports](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) in the producing stack and an [Fn::ImportValue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html) in the consuming stack to transfer that information from one stack to the other\.

## Physical names<a name="resources_physical_names"></a>

The logical names of resources in AWS CloudFormation are different from the names of resources that are shown in the AWS Management Console after AWS CloudFormation has deployed the resources\. The AWS CDK calls these final names *physical names*\.

For example, AWS CloudFormation might create the Amazon S3 bucket with the logical ID **Stack2MyBucket4DD88B4F** from the previous example with the physical name **stack2mybucket4dd88b4f\-iuv1rbv9z3to**\.

You can specify a physical name when creating constructs that represent resources by using the property <resourceType>Name\. The following example creates an Amazon S3 bucket with the physical name **my\-bucket\-name**\.

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket-name',
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket-name'
});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "MyBucket", bucket_name="my-bucket-name")
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(this, "MyBucket")
        .bucketName("my-bucket-name").build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "MyBucket", new BucketProps { BucketName = "my-bucket-name" });
```

------

Assigning physical names to resources has some disadvantages in AWS CloudFormation\. Most importantly, any changes to deployed resources that require a resource replacement, such as changes to a resource's properties that are immutable after creation, will fail if a resource has a physical name assigned\. If you end up in a state like that, the only solution is to delete the AWS CloudFormation stack, then deploy the AWS CDK app again\. See the [AWS CloudFormation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-name.html) for details\.

In some cases, such as when creating an AWS CDK app with cross\-environment references, physical names are required for the AWS CDK to function correctly\. In those cases, if you don't want to bother with coming up with a physical name yourself, you can let the AWS CDK name it for you by using the special value `PhysicalName.GENERATE_IF_NEEDED`, as follows\.

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: core.PhysicalName.GENERATE_IF_NEEDED,
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: core.PhysicalName.GENERATE_IF_NEEDED
});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "MyBucket",
                         bucket_name=core.PhysicalName.GENERATE_IF_NEEDED)
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(this, "MyBucket")
        .bucketName(PhysicalName.GENERATE_IF_NEEDED).build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "MyBucket", new BucketProps 
    { BucketName = PhysicalName.GENERATE_IF_NEEDED });
```

------

## Passing unique identifiers<a name="resources_identifiers"></a>

Whenever possible, you should pass resources by reference, as described in the previous section\. However, there are cases where you have no other choice but to refer to a resource by one of its attributes\. For example, when you are using the low\-level AWS CloudFormation resources, or need to expose resources to the runtime components of an AWS CDK application, such as when referring to Lambda functions through environment variables\.

These identifiers are available as attributes on the resources, such as the following\.

------
#### [ TypeScript ]

```
bucket.bucketName
lambdaFunc.functionArn
securityGroup.groupArn
```

------
#### [ JavaScript ]

```
bucket.bucketName
lambdaFunc.functionArn
securityGroup.groupArn
```

------
#### [ Python ]

```
bucket.bucket_name
lambda_func.function_arn
security_group_arn
```

------
#### [ Java ]

The Java AWS CDK binding uses getter methods for attributes\.

```
bucket.getBucketName()
lambdaFunc.getFunctionArn()
securityGroup.getGroupArn()
```

------
#### [ C\# ]

```
bucket.BucketName
lambdaFunc.FunctionArn
securityGroup.GroupArn
```

------

The following example shows how to pass a generated bucket name to an AWS Lambda function\.

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'Bucket');

new lambda.Function(this, 'MyLambda', {
  // ...
  environment: {
    BUCKET_NAME: bucket.bucketName,
  },
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'Bucket');

new lambda.Function(this, 'MyLambda', {
  // ...
  environment: {
    BUCKET_NAME: bucket.bucketName
  }
});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "Bucket")
      
lambda.Function(self, "MyLambda", environment=dict(BUCKET_NAME=bucket.bucket_name))
```

------
#### [ Java ]

```
final Bucket bucket = new Bucket(this, "Bucket");

Function.Builder.create(this, "MyLambda")
        .environment(new HashMap<String, String>() {{
                put("BUCKET_NAME", bucket.getBucketName());
            }}).build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "Bucket");

new Function(this, "MyLambda", new FunctionProps
{
    Environment = new Dictionary<string, string>
    {
        ["BUCKET_NAME"] = bucket.BucketName
    }
});
```

------

## Importing existing external resources<a name="resources_importing"></a>

Sometimes you already have a resource in your AWS account and want to use it in your AWS CDK app, for example, a resource that was defined through the console, the AWS SDK, directly with AWS CloudFormation, or in a different AWS CDK application\. You can turn the resource's ARN \(or another identifying attribute, or group of attributes\) into an AWS CDK object in the current stack by calling a static factory method on the resource's class\. 

The following example shows how to define a bucket based on an existing bucket with the ARN **arn:aws:s3:::my\-bucket\-name**, and a Amazon Virtual Private Cloud based on an existing VPC having a specific ID\.

------
#### [ TypeScript ]

```
// Construct a resource (bucket) just by its name (must be same account)
s3.Bucket.fromBucketName(this, 'MyBucket', 'my-bucket-name');

// Construct a resource (bucket) by its full ARN (can be cross account)
s3.Bucket.fromArn(this, 'MyBucket', 'arn:aws:s3:::my-bucket-name');

// Construct a resource by giving attribute(s) (complex resources)
ec2.Vpc.fromVpcAttributes(this, 'MyVpc', {
  vpcId: 'vpc-1234567890abcde',
});
```

------
#### [ JavaScript ]

```
// Construct a resource (bucket) just by its name (must be same account)
s3.Bucket.fromBucketName(this, 'MyBucket', 'my-bucket-name');

// Construct a resource (bucket) by its full ARN (can be cross account)
s3.Bucket.fromArn(this, 'MyBucket', 'arn:aws:s3:::my-bucket-name');

// Construct a resource by giving attribute(s) (complex resources)
ec2.Vpc.fromVpcAttributes(this, 'MyVpc', {
  vpcId: 'vpc-1234567890abcde'
});
```

------
#### [ Python ]

```
# Construct a resource (bucket) just by its name (must be same account)
s3.Bucket.from__bucket_name(self, "MyBucket", "my-bucket-name")

# Construct a resource (bucket) by its full ARN (can be cross account)
s3.Bucket.from_arn(self, "MyBucket", "arn:aws:s3:::my-bucket-name")

# Construct a resource by giving attribute(s) (complex resources)
ec2.Vpc.from_vpc_attributes(self, "MyVpc", vpc_id="vpc-1234567890abcdef")
```

------
#### [ Java ]

```
// Construct a resource (bucket) just by its name (must be same account)
Bucket.fromBucketName(this, "MyBucket", "my-bucket-name");

// Construct a resource (bucket) by its full ARN (can be cross account)
Bucket.fromBucketArn(this, "MyBucket",
        "arn:aws:s3:::my-bucket-name");

// Construct a resource by giving attribute(s) (complex resources)
Vpc.fromVpcAttributes(this, "MyVpc", VpcAttributes.builder()
        .vpcId("vpc-1234567890abcdef").build());
```

------
#### [ C\# ]

```
// Construct a resource (bucket) just by its name (must be same account)
Bucket.FromBucketName(this, "MyBucket", "my-bucket-name");

// Construct a resource (bucket) by its full ARN (can be cross account)
Bucket.FromBucketArn(this, "MyBucket", "arn:aws:s3:::my-bucket-name");

// Construct a resource by giving  attribute(s) (complex resources)
Vpc.FromVpcAttributes(this, "MyVpc", new VpcAttributes
{ 
    VpcId = "vpc-1234567890abcdef" 
});
```

------

Because the `ec2.Vpc` construct is complex, composed of many AWS resources, such as the VPC itself, subnets, security groups, and routing tables\), it can be difficult to import those resources using attributes\. To address this, the VPC construct contains a `fromLookup` method \(Python: `from_lookup`\) that uses a [context method](context.md#context_methods) to resolve all the required attributes at synthesis time, and cache the values for future use in `cdk.context.json`\. 

You must provide attributes sufficient to uniquely identify a VPC in your AWS account\. For example, there can only ever be one default VPC, so specifying that you want to import the VPC marked as the default is sufficient\.

------
#### [ TypeScript ]

```
ec2.Vpc.fromLookup(this, 'DefaultVpc', { 
  isDefault: true 
});
```

------
#### [ JavaScript ]

```
ec2.Vpc.fromLookup(this, 'DefaultVpc', { 
  isDefault: true 
});
```

------
#### [ Python ]

```
ec2.Vpc.from_lookup(self, "DefaultVpc", is_default=True)
```

------
#### [ Java ]

```
Vpc.fromLookup(this, "DefaultVpc", VpcLookupOptions.builder()
        .isDefault(true).build());
```

------
#### [ C\# ]

```
Vpc.FromLookup(this, id = "DefaultVpc", new VpcLookupOptions { IsDefault = true });
```

------

You can use the `tags` property to query by tag\. Tags may be added to the VPC at the time of its creation using AWS CloudFormation or the AWS CDK, and they may be edited at any time after creation using the AWS Management Console, the AWS CLI, or an AWS SDK\. In addition to any tags you have added yourself, the AWS CDK automatically adds the following tags to all VPCs it creates\. 
+ *Name* – The name of the VPC\.
+ *aws\-cdk:subnet\-name* – The name of the subnet\.
+ *aws\-cdk:subnet\-type* – The type of the subnet: Public, Private, or Isolated\.

------
#### [ TypeScript ]

```
ec2.Vpc.fromLookup(this, 'PublicVpc', 
    {tags: {'aws-cdk:subnet-type': "Public"}});
```

------
#### [ JavaScript ]

```
ec2.Vpc.fromLookup(this, 'PublicVpc', 
    {tags: {'aws-cdk:subnet-type': "Public"}});
```

------
#### [ Python ]

```
ec2.Vpc.from_lookup(self, "PublicVpc", 
    tags={"aws-cdk:subnet-type": "Public"})
```

------
#### [ Java ]

```
Vpc.fromLookup(this, "PublicVpc", VpcLookupOptions.builder()
        .tags(new HashMap<String, String> {{ put("aws-cdk:subnet-type", "Public"); }})
        .build());
```

------
#### [ C\# ]

```
Vpc.FromLookup(this, id = "PublicVpc", new VpcLookupOptions 
     { Tags = new Dictionary<string, string> { ["aws-cdk:subnet-type"] = "Public" });
```

------

Note that `Vpc.fromLookup()` works only in stacks that are defined with an explicit **account** and **region** in their `env` property\. If the AWS CDK attempts to look up an Amazon VPC from an [environment\-agnostic stack](stacks.md#stack_api), the CLI does not know which environment to query to find the VPC\.

Although you can use an imported resource anywhere, you cannot modify the imported resource\. For example, calling `addToResourcePolicy` \(Python: `add_to_resource_policy`\) on an imported `s3.Bucket` does nothing\.

## Permission grants<a name="resources_grants"></a>

AWS constructs make least\-privilege permissions easy to achieve by offering simple, intent\-based APIs to express permission requirements\. Many AWS constructs offer grant methods that enable you to easily grant an entity, such as an IAM role or a user, permission to work with the resource without having to manually craft one or more IAM permission statements\.

The following example creates the permissions to allow a Lambda function's execution role to read and write objects to a particular Amazon S3 bucket\. If the Amazon S3 bucket is encrypted using an AWS KMS key, this method also grants the Lambda function's execution role permissions to decrypt using this key\.

------
#### [ TypeScript ]

```
if (bucket.grantReadWrite(func).success) {
  // ...
}
```

------
#### [ JavaScript ]

```
if ( bucket.grantReadWrite(func).success) {
  // ...
}
```

------
#### [ Python ]

```
if bucket.grant_read_write(func).success:
    # ...
```

------
#### [ Java ]

```
if (bucket.grantReadWrite(func).getSuccess()) {
    // ...
}
```

------
#### [ C\# ]

```
if (bucket.GrantReadWrite(func).Success)
{ 
    // ...
}
```

------

The grant methods return an `iam.Grant` object\. Use the `success` attribute of the `Grant` object to determine whether the grant was effectively applied \(for example, it may not have been applied on [imported resources](#resources_referencing)\)\. You can also use the `assertSuccess` \(Python: `assert_success`\) method of the `Grant` object to enforce that the grant was successfully applied\.

If a specific grant method isn't available for the particular use case, you can use a generic grant method to define a new grant with a specified list of actions\. 

The following example shows how to grant a Lambda function access to the Amazon DynamoDB `CreateBackup` action\.

------
#### [ TypeScript ]

```
table.grant(func, 'dynamodb:CreateBackup');
```

------
#### [ JavaScript ]

```
table.grant(func, 'dynamodb:CreateBackup');
```

------
#### [ Python ]

```
table.grant(func, "dynamodb:CreateBackup")
```

------
#### [ Java ]

```
table.grant(func, "dynamodb:CreateBackup");
```

------
#### [ C\# ]

```
table.Grant(func, "dynamodb:CreateBackup");
```

------

Many resources, such as Lambda functions, require a role to be assumed when executing code\. A configuration property enables you to specify an `iam.IRole`\. If no role is specified, the function automatically creates a role specifically for this use\. You can then use grant methods on the resources to add statements to the role\.

The grant methods are built using lower\-level APIs for handling with IAM policies\. Policies are modeled as [PolicyDocument](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-iam-readme.html) objects\. Add statements directly to roles \(or a construct's attached role\) using the `addToRolePolicy` method \(Python: `add_to_role_policy`\), or to a resource's policy \(such as a `Bucket` policy\) using the `addToResourcePolicy` \(Python: `add_to_resource_policy`\) method\. 

## Metrics and alarms<a name="resources_metrics"></a>

Many resources emit CloudWatch metrics that can be used to set up monitoring dashboards and alarms\. AWS constructs have metric methods that allow easy access to the metrics without having to look up the correct name to use\.

The following example shows how to define an alarm when the `ApproximateNumberOfMessagesNotVisible` of an Amazon SQS queue exceeds 100\.

------
#### [ TypeScript ]

```
import * as cw from '@aws-cdk/aws-cloudwatch';
import * as sqs from '@aws-cdk/aws-sqs';
import { Duration } from '@aws-cdk/core';

const queue = new sqs.Queue(this, 'MyQueue');

const metric = queue.metricApproximateNumberOfMessagesNotVisible({
  label: 'Messages Visible (Approx)',
  period: Duration.minutes(5),
  // ...
});
metric.createAlarm(this, 'TooManyMessagesAlarm', {
  comparisonOperator: cw.ComparisonOperator.GREATER_THAN_THRESHOLD,
  threshold: 100,
  // ...
});
```

------
#### [ JavaScript ]

```
const cw = require('@aws-cdk/aws-cloudwatch');
const sqs = require('@aws-cdk/aws-sqs');
const { Duration } = require('@aws-cdk/core');

const queue = new sqs.Queue(this, 'MyQueue');

const metric = queue.metricApproximateNumberOfMessagesNotVisible({
  label: 'Messages Visible (Approx)',
  period: Duration.minutes(5)
  // ...
});
metric.createAlarm(this, 'TooManyMessagesAlarm', {
  comparisonOperator: cw.ComparisonOperator.GREATER_THAN_THRESHOLD,
  threshold: 100
  // ...
});
```

------
#### [ Python ]

```
import aws_cdk.aws_cloudwatch as cw
import aws_cdk.aws_sqs as sqs
from aws_cdk.core import Duration

queue = sqs.Queue(self, "MyQueue")
metric = queue.metric_approximate_number_of_messages_not_visible(
    label="Messages Visible (Approx)",
    period=Duration.minutes(5),
    # ...
)
metric.create_alarm(self, "TooManyMessagesAlarm",
    comparison_operator=cw.ComparisonOperator.GREATER_THAN_THRESHOLD,
    threshold=100,
    # ...
)
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.Duration;
import software.amazon.awscdk.services.sqs.Queue;
import software.amazon.awscdk.services.cloudwatch.Metric;
import software.amazon.awscdk.services.cloudwatch.MetricOptions;
import software.amazon.awscdk.services.cloudwatch.CreateAlarmOptions;
import software.amazon.awscdk.services.cloudwatch.ComparisonOperator;

Queue queue = new Queue(this, "MyQueue");

Metric metric = queue
        .metricApproximateNumberOfMessagesNotVisible(MetricOptions.builder()
                .label("Messages Visible (Approx)")
                .period(Duration.minutes(5)).build());

metric.createAlarm(this, "TooManyMessagesAlarm", CreateAlarmOptions.builder()
                .comparisonOperator(ComparisonOperator.GREATER_THAN_THRESHOLD)
                .threshold(100)
                // ...
                .build());
```

------
#### [ C\# ]

```
using cdk = Amazon.CDK;
using cw = Amazon.CDK.AWS.CloudWatch;
using sqs = Amazon.CDK.AWS.SQS;

var queue = new sqs.Queue(this, "MyQueue");
var metric = queue.MetricApproximateNumberOfMessagesNotVisible(new cw.MetricOptions
{
    Label = "Messages Visible (Approx)",
    Period = cdk.Duration.Minutes(5),
    // ...
});
metric.CreateAlarm(this, "TooManyMessagesAlarm", new cw.CreateAlarmOptions
{
    ComparisonOperator = cw.ComparisonOperator.GREATER_THAN_THRESHOLD,
    Threshold = 100,
    // ..
});
```

------

If there is no method for a particular metric, you can use the general metric method to specify the metric name manually\.

Metrics can also be added to CloudWatch dashboards\. See [CloudWatch](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cloudwatch-readme.html)\.

## Network traffic<a name="resources_traffic"></a>

In many cases, you must enable permissions on a network for an application to work, such as when the compute infrastructure needs to access the persistence layer\. Resources that establish or listen for connections expose methods that enable traffic flows, including setting security group rules or network ACLs\.

[IConnectable](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ec2.IConnectable.html) resources have a `connections` property that is the gateway to network traffic rules configuration\.

You enable data to flow on a given network path by using `allow` methods\. The following example enables HTTPS connections to the web and incoming connections from the Amazon EC2 Auto Scaling group `fleet2`\.

------
#### [ TypeScript ]

```
import * as asg from '@aws-cdk/aws-autoscaling';
import * as ec2 from '@aws-cdk/aws-ec2';

const fleet1: asg.AutoScalingGroup = asg.AutoScalingGroup(/*...*/);

// Allow surfing the (secure) web
fleet1.connections.allowTo(new ec2.Peer.anyIpv4(), new ec2.Port({ fromPort: 443, toPort: 443 }));

const fleet2: asg.AutoScalingGroup = asg.AutoScalingGroup(/*...*/);
fleet1.connections.allowFrom(fleet2, ec2.Port.AllTraffic());
```

------
#### [ JavaScript ]

```
const asg = require('@aws-cdk/aws-autoscaling');
const ec2 = require('@aws-cdk/aws-ec2');

const fleet1 = asg.AutoScalingGroup();

// Allow surfing the (secure) web
fleet1.connections.allowTo(new ec2.Peer.anyIpv4(), new ec2.Port({ fromPort: 443, toPort: 443 }));

const fleet2 = asg.AutoScalingGroup();
fleet1.connections.allowFrom(fleet2, ec2.Port.AllTraffic());
```

------
#### [ Python ]

```
import aws_cdk.aws_autoscaling as asg
import aws_cdk.aws_ec2 as ec2

fleet1 = asg.AutoScalingGroup( ... )

# Allow surfing the (secure) web
fleet1.connections.allow_to(ec2.Peer.any_ipv4(), 
  ec2.Port(PortProps(from_port=443, to_port=443)))

fleet2 = asg.AutoScalingGroup( ... )
fleet1.connections.allow_from(fleet2, ec2.Port.all_traffic())
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.autoscaling.AutoScalingGroup;
import software.amazon.awscdk.services.ec2.Peer;
import software.amazon.awscdk.services.ec2.Port;

AutoScalingGroup fleet1 = AutoScalingGroup.Builder.create(this, "MyFleet")
        /* ... */.build();

// Allow surfing the (secure) Web
fleet1.getConnections().allowTo(Peer.anyIpv4(),
        Port.Builder.create().fromPort(443).toPort(443).build());

AutoScalingGroup fleet2 = AutoScalingGroup.Builder.create(this, "MyFleet2")
        /* ... */.build();
fleet1.getConnections().allowFrom(fleet2, Port.allTraffic());
```

------
#### [ C\# ]

```
using cdk = Amazon.CDK;
using asg = Amazon.CDK.AWS.AutoScaling;
using ec2 = Amazon.CDK.AWS.EC2;

// Allow surfing the (secure) Web
var fleet1 = new asg.AutoScalingGroup(this, "MyFleet", new asg.AutoScalingGroupProps { /* ... */ });
fleet1.Connections.AllowTo(ec2.Peer.AnyIpv4(), new ec2.Port(new ec2.PortProps 
  { FromPort = 443, ToPort = 443 });

var fleet2 = new asg.AutoScalingGroup(this, "MyFleet2", new asg.AutoScalingGroupProps { /* ... */ });
fleet1.Connections.AllowFrom(fleet2, ec2.Port.AllTraffic());
```

------

Certain resources have default ports associated with them, for example, the listener of a load balancer on the public port, and the ports on which the database engine accepts connections for instances of an Amazon RDS database\. In such cases, you can enforce tight network control without having to manually specify the port by using the `allowDefaultPortFrom` and `allowToDefaultPort` methods \(Python: `allow_default_port_from`, `allow_to_default_port`\)\. 

The following example shows how to enable connections from any IPV4 address, and a connection from an Auto Scaling group to access a database\.

------
#### [ TypeScript ]

```
listener.connections.allowDefaultPortFromAnyIpv4('Allow public access');

fleet.connections.allowToDefaultPort(rdsDatabase, 'Fleet can access database');
```

------
#### [ JavaScript ]

```
listener.connections.allowDefaultPortFromAnyIpv4('Allow public access');

fleet.connections.allowToDefaultPort(rdsDatabase, 'Fleet can access database');
```

------
#### [ Python ]

```
listener.connections.allow_default_port_from_any_ipv4("Allow public access")

fleet.connections.allow_to_default_port(rds_database, "Fleet can access database")
```

------
#### [ Java ]

```
listener.getConnections().allowDefaultPortFromAnyIpv4("Allow public access");

fleet.getConnections().AllowToDefaultPort(rdsDatabase, "Fleet can access database");
```

------
#### [ C\# ]

```
listener.Connections.AllowDefaultPortFromAnyIpv4("Allow public access");

fleet.Connections.AllowToDefaultPort(rdsDatabase, "Fleet can access database");
```

------

## Event handling<a name="resources_events"></a>

Some resources can act as event sources\. Use the `addEventNotification` method \(Python: `add_event_notification`\) to register an event target to a particular event type emitted by the resource\. In addition to this, `addXxxNotification` methods offer a simple way to register a handler for common event types\. 

The following example shows how to trigger a Lambda function when an object is added to an Amazon S3 bucket\.

------
#### [ TypeScript ]

```
import * as s3nots from '@aws-cdk/aws-s3-notifications';

const handler = new lambda.Function(this, 'Handler', { /*…*/ });
const bucket = new s3.Bucket(this, 'Bucket');
bucket.addObjectCreatedNotification(new s3nots.LambdaDestination(handler));
```

------
#### [ JavaScript ]

```
const s3nots = require('@aws-cdk/aws-s3-notifications');

const handler = new lambda.Function(this, 'Handler', { /*…*/ });
const bucket = new s3.Bucket(this, 'Bucket');
bucket.addObjectCreatedNotification(new s3nots.LambdaDestination(handler));
```

------
#### [ Python ]

```
import aws_cdk.aws_s3_notifications as s3_nots

handler = lambda_.Function(self, "Handler", ...)
bucket = s3.Bucket(self, "Bucket")
bucket.add_object_created_notification(s3_nots.LambdaDestination(handler))
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.s3.notifications.LambdaDestination;

Function handler = Function.Builder.create(this, "Handler")/* ... */.build();
Bucket bucket = new Bucket(this, "Bucket");
bucket.addObjectCreatedNotification(new LambdaDestination(handler));
```

------
#### [ C\# ]

```
using lambda = Amazon.CDK.AWS.Lambda;
using s3 = Amazon.CDK.AWS.S3;
using s3Nots = Amazon.CDK.AWS.S3.Notifications;

var handler = new lambda.Function(this, "Handler", new lambda.FunctionProps { .. });
var bucket = new s3.Bucket(this, "Bucket");
bucket.AddObjectCreatedNotification(new s3Nots.LambdaDestination(handler));
```

------

## Removal policies<a name="resources_removal"></a>

Resources that maintain persistent data, such as databases and Amazon S3 buckets and even Amazon ECR registries, have a *removal policy* that indicates whether to delete persistent objects when the AWS CDK stack that contains them is destroyed\. The values specifying the removal policy are available through the `RemovalPolicy` enumeration in the AWS CDK `core` module\.

**Note**  
Resources besides those that store data persistently may also have a `removalPolicy` that is used for a different purpose\. For example, a Lambda function version uses a `removalPolicy` attribute to determine whether a given version is retained when a new version is deployed\. These have different meanings and defaults compared to the removal policy on an Amazon S3 bucket or DynamoDB table\.


| Value | meaning | 
| --- |--- |
| RemovalPolicy\.RETAIN | Keep the contents of the resource when destroying the stack \(default\)\. The resource is orphaned from the stack and must be deleted manually\. If you attempt to re\-deploy the stack while the resource still exists, you will receive an error message due to a name conflict\. | 
| RemovalPolicy\.DESTROY | The resource will be destroyed along with the stack\. | 

AWS CloudFormation does not remove Amazon S3 buckets that contain files even if their removal policy is set to `DESTROY`\. Attempting to do so is a AWS CloudFormation error\. Delete the files from the bucket before destroying the stack\. You can automate this using a custom resource; see the third\-party construct [auto\-delete\-bucket](https://github.com/mobileposse/auto-delete-bucket/tree/master/src/lambda) for an example\.

Following is an example of creating an Amazon S3 bucket with `RemovalPolicy.DESTROY`\.

------
#### [ TypeScript ]

```
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';
  
export class CdkTestStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
  
    const bucket = new s3.Bucket(this, 'Bucket', {
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });
  }
}
```

------
#### [ JavaScript ]

```
const cdk = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');
  
class CdkTestStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);
  
    const bucket = new s3.Bucket(this, 'Bucket', {
      removalPolicy: cdk.RemovalPolicy.DESTROY
    });
  }
}

module.exports = { CdkTestStack }
```

------
#### [ Python ]

```
import aws_cdk.core as cdk
import aws_cdk.aws_s3 as s3

class CdkTestStack(cdk.stack):
    def __init__(self, scope: cdk.Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)
        
        bucket = s3.Bucket(self, "Bucket",
            removal_policy=cdk.RemovalPolicy.DESTROY)
```

------
#### [ Java ]

```
software.amazon.awscdk.core.*;
import software.amazon.awscdk.services.s3.*;

public class CdkTestStack extends Stack {
    public CdkTestStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkTestStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "Bucket")
                .removalPolicy(RemovalPolicy.DESTROY).build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

public CdkTestStack(Construct scope, string id, IStackProps props) : base(scope, id, props)
{
    new Bucket(this, "Bucket", new BucketProps {
        RemovalPolicy = RemovalPolicy.DESTROY
    });
}
```

------

You can also apply a removal policy directly to the underlying AWS CloudFormation resource via the `applyRemovalPolicy()` method\.

------
#### [ TypeScript ]

```
const resource = bucket.node.findChild('Resource') as cdk.CfnResource;
resource.applyRemovalPolicy(cdk.RemovalPolicy.DESTROY);
```

------
#### [ JavaScript ]

```
const resource = bucket.node.findChild('Resource');
resource.applyRemovalPolicy(cdk.RemovalPolicy.DESTROY);
```

------
#### [ Python ]

```
resource = bucket.node.find_child('Resource')
resource.apply_removal_policy(cdk.RemovalPolicy.DESTROY);
```

------
#### [ Java ]

```
CfnResource resource = (CfnResource)bucket.node.findChild("Resource");
resource.applyRemovalPolicy(cdk.RemovalPolicy.DESTROY);
```

------
#### [ C\# ]

```
var resource = (CfnResource)bucket.node.findChild('Resource');
resource.ApplyRemovalPolicy(cdk.RemovalPolicy.DESTROY);
```

------

**Note**  
The AWS CDK's `RemovalPolicy` translates to AWS CloudFormation's `DeletionPolicy`, but the default in AWS CDK is to retain the data, which is the opposite of the AWS CloudFormation default\.