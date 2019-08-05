# Resources<a name="resources"></a>

As described in [Constructs](constructs.md), the AWS CDK provides a rich class library of constructs, called *AWS constructs*, that represent all AWS resources\. This section describes some common patterns and best practices for how to use these constructs\.

Defining AWS resources in your CDK app is exactly like defining any other construct\. You create an instance of the construct class, pass in the scope as the first argument, the logical ID of the construct, and a set of configuration properties \(props\)\. For example, here's how to create an Amazon SQS queue with KMS encryption using the [sqs\.Queue](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-sqs.Queue.html) construct from the AWS Construct Library\.

```
import sqs = require('@aws-cdk/aws-sqs');
            
new sqs.Queue(this, 'MyQueue', {
    encryption: sqs.QueueEncryption.KmsManaged
});
```

Some configuration props are optional, and in many cases have default values\. In some cases, all props are optional, and the last argument can be omitted entirely\.

## Resource Attributes<a name="resources_attributes"></a>

Most resources in the AWS Construct Library expose attributes, which are resolved at deployment time by AWS CloudFormation\. Attributes are exposed in the form of properties on the resource classes with the type name as a prefix\. The following example shows how to get the URL of an Amazon SQS queue using the `queueUrl` property\.

```
import sqs = require('@aws-cdk/aws-sqs');

const queue = new sqs.Queue(this, 'MyQueue');

queue.queueUrl; // => A string representing a deploy-time value
```

See [Tokens](tokens.md) for information about how the AWS CDK encodes deploy\-time attributes as strings\.

## Referencing Resources<a name="resources_referencing"></a>

Many AWS CDK classes require properties that are AWS CDK resource objects \(resources\)\. To satisfy these requirements, you can refer to a resource in one of two ways:
+ By passing the resource directly
+ By passing the resource's unique identifier, which is typically an ARN, but it could also be an ID or a name

For example, an Amazon ECS service requires a reference to the cluster on which it runs; an Amazon CloudFront distribution requires a reference to the bucket containing source code\.

If a construct property represents another AWS construct, its type is that of the interface type of that construct\. For example, the Amazon ECS service takes a property `cluster` of type `ecs.ICluster`; the CloudFront distribution takes a property `sourceBucket` of type `s3.IBucket`\.

Because every resource implements its corresponding interface, you can directly pass any resource object you're defining in the same AWS CDK app\. The following example defines an Amazon ECS cluster and then uses it to define an Amazon ECS service\.

```
const cluster = new ecs.Cluster(this, 'Cluster', { /* ... */ });

const service = new ecs.Service(this, 'Service', {
  cluster: cluster,
  /* ... */
});
```

## Accessing Resources in a Different Stack<a name="resource_stack"></a>

You can access resources in a different stack, as long as they are in the same account and AWS Region\. The following example defines the stack `stack1`, which defines an Amazon S3 bucket\. Then it defines a second stack, `stack2`, which takes the bucket from `stack1` as a constructor property\.

```
const prod = { account: '123456789012', region: 'us-east-1' };

const stack1 = new StackThatProvidesABucket(app, 'Stack1' , { env: prod });

// stack2 will take a property { bucket: IBucket }
const stack2 = new StackThatExpectsABucket(app, 'Stack2', {
  bucket: stack1.bucket, 
  env: prod
});
```

If the AWS CDK determines that the resource is in the same account and Region, but in a different stack, it automatically synthesizes AWS CloudFormation [exports](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) in the producing stack and an [Fn::ImportValue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html) in the consuming stack to transfer that information from one stack to the other\.

## Physical Names<a name="resources_physical_names"></a>

The logical names of resources in AWS CloudFormation are different from the names of resources that are shown in the AWS Management Console after AWS CloudFormation has deployed the resources\. The AWS CDK calls these final names *physical names*\.

For example, AWS CloudFormation might create the Amazon S3 bucket with the logical ID **Stack2MyBucket4DD88B4F** from the previous example with the physical name **stack2mybucket4dd88b4f\-iuv1rbv9z3to**\.

You can specify a physical name when creating constructs that represent resources by using the property <resourceType>Name\. The following example creates an Amazon S3 bucket with the physical name **my\-bucket\-name**\.

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket-name',
});
```

Assigning physical names to resources has some disadvantages in AWS CloudFormation\. Most importantly, any changes to deployed resources that require a resource replacement, such as changes to a resource's properties that are immutable after creation, will fail if a resource has a physical name assigned\. If you end up in a state like that, the only solution is to delete the AWS CloudFormation stack, then deploy the AWS CDK app again\. See the [AWS CloudFormation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-name.html) for details\.

In some cases, such as when creating an AWS CDK app with cross\-environment references, physical names are required for the AWS CDK to function correctly\. In those cases, if you don't want to bother with coming up with a physical name yourself, you can let the AWS CDK name it for you by using the special value `PhysicalName.GENERATE_IF_NEEDED,`, as follows\.

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: PhysicalName.GENERATE_IF_NEEDED,
});
```

## Passing Unique Identifiers<a name="resources_identifiers"></a>

Whenever possible, you should pass resources by reference, as described in the previous section\. However, there are cases where you have no other choice but to refer to a resource by one of its attributes\. For example, when you are using the low\-level AWS CloudFormation resources, or need to expose resources to the runtime components of an AWS CDK application, such as when referring to Λ functions through environment variables\.

These identifiers are available as attributes on the resources, such as the following\.

```
bucket.bucketName
lambda.functionArn
securityGroup.securityGroupId
```

The following example shows how to pass a generated bucket name to an AWS Lambda function\.

```
const bucket = new s3.Bucket(this, 'Bucket');

new lambda.Function(this, 'MyLambda', {
  /* ... */,
  environment: {
    BUCKET_NAME: bucket.bucketName,
  },
});
```

## Importing Existing External Resources<a name="resources_importing"></a>

Sometimes you already have a resource in your AWS account and want to use it in your AWS CDK app, for example, a resource that was defined through the console, the AWS SDK, directly with AWS CloudFormation, or in a different AWS CDK application\. You can turn the resource's ARN \(or another identifying attribute, or group of attributes\) into an AWS CDK object in the current stack by calling a static factory method on the resource's class\. 

The following example shows how to define a bucket based on the existing bucket with the ARN **arn:aws:s3:::my\-bucket\-name**, and a VPC based on the existing VPC with the resource name `booh`\.

```
// Construct a resource (bucket) just by its name (must be same account)
Bucket.fromName(this, 'Bucket', 'my-bucket-name');

// Construct a resource (bucket) by its full ARN (can be cross account)
Bucket.fromArn(this, 'Bucket', 'arn:aws:s3:::my-bucket-name');

// Construct a resource by giving more than one attribute (complex resources)
Resource.fromAttributes(this, 'MyResource', {
  resourceName: 'booh',
  vpc: vpc
});
```

Because the `ec2.Vpc` construct is composed of many AWS resources, such as the VPC itself, subnets, security groups, and routing tables\), the complexity makes it difficult to import those resources using attributes\. To address this, the VPC construct contains a `fromLookup` method that uses a [context method](context.md#context_methods) to resolve all the required attributes at synthesis time, and cache the values for future usage in `cdk.context.json`\. 

The following example shows how to get the default VPC of a stack's environment\.

```
ec2.Vpc.fromLookup(this, 'DefaultVpc', { 
  isDefault: true 
});
```

Note that this approach works only for stacks that are defined with an explicit **account** and **region** in their `env` property\. If this is called from an [environment\-agnostic stack](stacks.md#stack_api), the CLI does not know which environment to query for the VPC\.

Although you can use an imported resource anywhere, you cannot modify the imported resource\. For example, calling `addToResourcePolicy` on an imported `s3.IBucket` does nothing\.

## Permission Grants<a name="resources_grants"></a>

AWS constructs make least\-privilege permissions easy to achieve by offering simple, intent\-based APIs to express permission requirements\. Many AWS constructs offer grant methods that enable you to easily grant an entity, such as an IAM role or a user, permission to work with the resource without having to manually craft one or more IAM permission statements\.

The following example creates the permissions to allow a Lambda function's execution role to read and write objects to a particular Amazon S3 bucket\. If the Amazon S3 bucket is encrypted using an AWS KMS key, this method also grants the Lambda function's execution role permissions to decrypt using this key\.

```
bucket.grantReadWrite(lambda);
```

The grant methods return an `iam.Grant` object\. Use the success attribute of the `Grant` object to determine whether the grant was effectively applied \(for example, it may not have been applied on [imported resources](#resources_referencing)\)\. You can also use the `assertSuccess` method of the `Grant` object to enforce that the grant was successfully applied\.

If a specific grant method isn't available for the particular use case, you can use a generic grant method to define a new grant with a specified list of actions\. 

The following example shows how to grant a Lambda function access to the Amazon DynamoDB `CreateBackup` action\.

```
table.grant(lambda, 'dynamodb:CreateBackup');
```

Many resources, such as Lambda functions, require a role to be assumed when executing code\. A configuration property enables you to specify an `iam.IRole`\. If no role is specified, the function automatically creates a role specifically for this use\. You can then use grant methods on the resources to add statements to the role\.

The grant methods are built using lower\-level APIs for handling with IAM policies\. Policies are modeled as [PolicyDocument](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-iam-readme.html) objects\. Add statements directly to roles \(or a construct's attached role\) using the `addToRolePolicy` method, or to a resource's policy \(such as a `Bucket` policy\) using the `addToResourcePolicy` method\. 

## Metrics and Alarms<a name="resources_metrics"></a>

Many resources emit CloudWatch metrics that can be used to set up monitoring dashboards and alarms\. AWS constructs have metric methods that allow easy access to the metrics without having to look up the correct name to use\.

The following example shows how to define an alarm when the `ApproximateNumberOfMessagesNotVisible` of an Amazon SQS queue exceeds 100\.

```
import cw = require('@aws-cdk/aws-cloudwatch');
import sqs = require('@aws-cdk/aws-sqs');
import { Duration } from '@aws-cdk/core';

const queue = new sqs.Queue(this, 'MyQueue');

const metric = queue.metricApproximateNumberOfMessagesNotVisible({
  label: 'Messages Visible (Approx)',
  period: Duration.minutes(5),
  // ...
});
metric.createAlarm(this, 'TooManyMessagesAlarm', {
  comparisonOperator: cw.ComparisonOperator.GreaterThan,
  threshold: 100,
  // ...
});
```

If there is no method for a particular metric, you can use the general metric method to specify the metric name manually\.

Metrics can also be added to CloudWatch dashboards\. See [CloudWatch](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cloudwatch-readme.html)\.

## Network Traffic<a name="resources_traffic"></a>

In many cases, you must enable permissions on a network for an application to work, such as when the compute infrastructure needs to access the persistence layer\. Resources that establish or listen for connections expose methods that enable traffic flows, including setting security group rules or network ACLs\.

[IConnectable](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ec2.IConnectable.html) resources have a `connections` property that is the gateway to network traffic rules configuration\.

You enable data to flow on a given network path by using `allow` methods\. The following example enables HTTPS connections to the web and incoming connections from the Amazon EC2 Auto Scaling group `fleet2`\.

```
import asg = require('@aws-cdk/aws-autoscaling');
import ec2 = require('@aws-cdk/aws-ec2');

const fleet: asg.AutoScalingGroup = /*…*/

// Allow surfing the (secure) web
fleet.connections.allowTo(new ec2.AnyIpv4(), new ec2.Port(443));

const fleet2: asg.AutoScalingGroup = this.newASG();
fleet.connections.allowFrom(fleet2);
```

Certain resources have default ports associated with them, for example, the listener of a load balancer on the public port, and the ports on which the database engine accepts connections for instances of an Amazon RDS database\. In such cases, you can enforce tight network control without having to manually specify the port by using the `allowDefaultPortFrom` and `allowToDefaultPort` methods\. 

The following example shows how to enable connections from any IPV4 address, and a connection from an Auto Scaling group to access a database\.

```
listener.connections.allowDefaultPortFromAnyIpv4('Allow public access');

fleet.connections.allowToDefaultPort(rdsDatabase, 'Fleet can access database');
```

## Amazon CloudWatch Events<a name="resources_cloudwatch"></a>

Some resources can act as event sources\. Use the `addEventNotification` method to register an event target to a particular event type emitted by the resource\. In addition to this, `addXxxNotification` methods offer a simplified way to register a handler for a common event type\. 

The following example shows how to trigger a Lambda function when an object is added to an Amazon S3 bucket\.

```
Import targets = require('@aws-cdk/aws-events-targets');
const handler = new lambda.Function(this, 'Handler', { /*…*/ });
const bucket = new s3.Bucket(this, 'Bucket');
bucket.addObjectCreatedNotification(new targets.LambdaFunction(handler));
```