# Constructs<a name="constructs"></a>

Constructs are the basic building blocks of AWS CDK apps\. A construct represents a "cloud component" and encapsulates everything AWS CloudFormation needs to create the component\. 

A construct can represent a single resource, such as an Amazon Simple Storage Service \(Amazon S3\) bucket, or it can represent a higher\-level component consisting of multiple AWS CDK resources\. Examples of such components include a worker queue with its associated compute capacity, a cron job with monitoring resources and a dashboard, or even an entire app spanning multiple AWS accounts and regions\.

## AWS Construct library<a name="constructs_lib"></a>

The AWS CDK includes the [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html), which contains constructs representing AWS resources\.

This library includes constructs that represent all the resources available on AWS\. For example, the `[s3\.Bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html)` class represents an Amazon S3 bucket, and the `[dynamodb\.Table](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-dynamodb.Table.html)` class represents an Amazon DynamoDB table\.

There are different levels of constructs in this library, beginning with low\-level constructs, which we call *CFN Resources* \(or L1, short for "level 1"\)\. These constructs directly represent all of the AWS resources that are available in AWS CloudFormation\. CFN Resources are periodically generated from the [AWS CloudFormation Resource Specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html)\. They are named **Cfn***Xyz*, where *Xyz* is name of the resource\. For example, [s3\.CfnBucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.CfnBucket.html) represents the [AWS::S3::Bucket](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html) CFN Resource\. When you use CFN resources, you must explicitly configure all resource properties, which requires a complete understanding of the details of the underlying AWS CloudFormation resource model\.

The next level of constructs, L2, also represent AWS resources, but with a higher\-level, intent\-based API\. They provide similar functionality, but provide the defaults, boilerplate, and glue logic you'd be writing yourself with a CFN Resource construct\. AWS constructs offer convenient defaults and reduce the need to know all the details about the AWS resources they represent, while providing convenience methods that make it simpler to work with the resource\. For example, the [s3\.Bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) class represents an Amazon S3 bucket with additional properties and methods, such as [bucket\.addLifeCycleRule\(\)](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#add-wbr-lifecycle-wbr-rulerule), which adds a lifecycle rule to the bucket\.

Finally, the AWS Construct Library includes even higher\-level constructs, which we call *patterns*\. These constructs are designed to help you complete common tasks in AWS, often involving multiple kinds of resources\. For example, the [aws\-ecs\-patterns\.ApplicationLoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecs-patterns.ApplicationLoadBalancedFargateService.html) construct represents an architecture that includes an AWS Fargate container cluster employing an Application Load Balancer \(ALB\)\. The [aws\-apigateway\.LambdaRestApi](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigateway.LambdaRestApi.html) construct represents an Amazon API Gateway API that's backed by an AWS Lambda function\.

For more information about how to navigate the library and discover constructs that can help you build your apps, see the [API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html)\.

## Composition<a name="constructs_composition"></a>

The key pattern for defining higher\-level abstractions through constructs is called *composition*\. A high\-level construct can be composed from any number of lower\-level constructs, and in turn, those could be composed from even lower\-level constructs\. To enable this pattern, constructs are always defined within the scope of another construct\. This scoping pattern results in a hierarchy of constructs known as a *construct tree*\. In the AWS CDK, the root of the tree represents your entire **[AWS CDK app](apps.md)**\. Within the app, you typically define one or more **[stacks](stacks.md)**, which are the unit of deployment, analogous to AWS CloudFormation stacks\. Within stacks, you define resources, or other constructs that eventually contain resources\.

Composition of constructs means that you can define reusable components and share them like any other code\. For example, a central team can define a construct that implements the company's best practice for a DynamoDB table with backup, global replication, auto\-scaling, and monitoring, and share it with teams across a company or publicly\. Teams can now use this construct as they would any other library package in their favorite programming language to define their tables and comply with their team's best practices\. When the library is updated, developers can pick up the updates and enjoy any bug fixes and improvements through the workflows they already have for their other types of code\.

## Initialization<a name="constructs_init"></a>

Constructs are implemented in classes that extend the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html) base class\. You define a construct by instantiating the class\. All constructs take three parameters when they are initialized:
+ **Scope** – The construct within which this construct is defined\. You should almost always pass `this` for the scope, because it represents the current scope in which you are defining the construct\.
+ **id** – An [identifier](identifiers.md) that must be unique within this scope\. The identifier serves as a namespace for everything that's encapsulated within the scope's subtree and is used to allocate unique identities such as [resource names](resources.md#resources_physical_names) and AWS CloudFormation logical IDs\.
+ **Props** – A set of properties or keyword arguments, depending upon the supported language, that define the construct's initial configuration\. In most cases, constructs provide sensible defaults, and if all props elements are optional, you can leave out the **props** parameter completely\.

Identifiers need only be unique within a scope\. This lets you instantiate and reuse constructs without concern for the constructs and identifiers they might contain, and enables composing constructs into higher level abstractions\. In addition, scopes make it possible to refer to groups of constructs all at once, for example for [tagging](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html) or for specifying where the constructs will be deployed\.

## Apps and stacks<a name="constructs_apps_stacks"></a>

We call your CDK application an *app*, which is represented by the AWS CDK class [App](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.App.html)\. The following example defines an app with a single stack that contains a single Amazon S3 bucket with versioning enabled:

------
#### [ TypeScript ]

```
import { App, Stack, StackProps } from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

class HelloCdkStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}

const app = new App();
new HelloCdkStack(app, "HelloCdkStack");
```

------
#### [ JavaScript ]

```
const { App , Stack } = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}

const app = new App();
new HelloCdkStack(app, "HelloCdkStack");
```

------
#### [ Python ]

```
from aws_cdk.core import App, Stack
from aws_cdk import aws_s3 as s3

class HelloCdkStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        s3.Bucket(self, "MyFirstBucket", versioned=True)

app = core.App()
HelloCdkStack(app, "HelloCdkStack")
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.*;
import software.amazon.awscdk.services.s3.*;

public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
                .versioned(true).build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdkApp
{
    internal static class Program
    {
        public static void Main(string[] args)
        {
            var app = new App();
            new HelloCdkStack(app, "HelloCdkStack");
            app.Synth();
        }
    }
    
    public class HelloCdkStack : Stack
    {
        public HelloCdkStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
        {
            new Bucket(this, "MyFirstBucket", new BucketProps { Versioned = true });
        }
    }
}
```

------

As you can see, you need a scope within which to define your bucket\. Since resources eventually need to be deployed as part of a AWS CloudFormation stack into an *AWS [environment](environments.md)*, which covers a specific AWS account and AWS region\. AWS constructs, such as `s3.Bucket`, must be defined within the scope of a [Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/stack.html)\.

Stacks in AWS CDK apps extend the **Stack** base class, as shown in the previous example\. This is a common pattern when creating a stack within your AWS CDK app: extend the **Stack** class, define a constructor that accepts **scope**, **id**, and **props**, and invoke the base class constructor via `super` with the received **scope**, **id**, and **props**, as shown in the following example\.

------
#### [ TypeScript ]

```
class HelloCdkStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

    //...
  }
}
```

------
#### [ JavaScript ]

```
class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    //...
  }
}
```

------
#### [ Python ]

```
class HelloCdkStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # ...
```

------
#### [ Java ]

```
public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // ...
    }
}
```

------
#### [ C\# ]

```
public class HelloCdkStack : Stack
{
    public HelloCdkStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
    {
        //...
    }
}
```

------

## Using L1 constructs<a name="constructs_l1_using"></a>

Once you have defined a stack, you can populate it with resources by instantiating constructs\. First, we'll do it with an L1 construct\.

L1 constructs are exactly the resources defined by AWS CloudFormation—no more, no less\. You must provide the resource's required configuration yourself\. Here, for example, is how to create an Amazon S3 bucket using the `CfnBucket` class\. \(You'll see a similar definition using the `Bucket` class in the next section\.\) 

------
#### [ TypeScript ]

```
const bucket = new s3.CfnBucket(this, "MyBucket", {
  bucketName: "MyBucket"
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.CfnBucket(this, "MyBucket", {
  bucketName: "MyBucket"
});
```

------
#### [ Python ]

```
bucket = s3.CfnBucket(self, "MyBucket", bucket_name="MyBucket")
```

------
#### [ Java ]

```
CfnBucket bucket = new CfnBucket.Builder().bucketName("MyBucket").build();
```

------
#### [ C\# ]

```
var bucket = new CfnBucket(this, "MyBucket", new CfnBucketProps
{
    BucketName= "MyBucket"
});
```

------

In Python, Java, and C\#, L1 construct properties that aren't simple Booleans, strings, numbers, or containers are represented by types defined as inner classes of the L1 construct\. For example, the optional property `corsConfiguration` of a `CfnBucket` requires a wrapper of type `Cfn.CorsConfigurationProperty`\. Here we are defining `corsConfiguration` on a `CfnBucket` instance\. 

------
#### [ TypeScript ]

```
const bucket = new s3.CfnBucket(this, "MyBucket", {
  bucketName: "MyBucket",
  corsConfiguration: {
    corsRules: [{
          allowedOrigins: ["*"],
          allowedMethods: ["*"]
    }]
  }
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.CfnBucket(this, "MyBucket", {
  bucketName: "MyBucket",
  corsConfiguration: {
    corsRules: [{
          allowedOrigins: ["*"],
          allowedMethods: ["*"]
    }]
  }
});
```

------
#### [ Python ]

```
bucket = CfnBucket(self, "MyBucket", bucket_name="MyBucket",
    cors_configuration=CfnBucket.CorsConfigurationProperty(
        cors_rules=[CfnBucket.CorsRuleProperty(
            allowed_origins=["*"],
            allowed_methods=["GET"]
        )]
    )
)
```

------
#### [ Java ]

```
CfnBucket bucket = CfnBucket.Builder.create(this, "MyBucket")
                        .bucketName("MyBucket")
                        .corsConfiguration(new CfnBucket.CorsConfigurationProperty.Builder()
                            .corsRules(Arrays.asList(new CfnBucket.CorsRuleProperty.Builder()
                                .allowedOrigins(Arrays.asList("*"))
                                .allowedMethods(Arrays.asList("GET"))
                                .build()))
                            .build())
                        .build();
```

------
#### [ C\# ]

```
var bucket = new CfnBucket(this, "MyBucket", new CfnBucketProps
{
    BucketName = "MyBucket",
    CorsConfiguration = new CfnBucket.CorsConfigurationProperty
    {
        CorsRules = new object[] {
            new CfnBucket.CorsRuleProperty
            {
                AllowedOrigins = new string[] { "*" },
                AllowedMethods = new string[] { "GET" },
            }
        }
    }
});
```

------

**Important**  
You can't use L2 property types with L1 constructs, or vice versa\. When working with L1 constructs, always use the types defined inside the L1 construct you're using\. Do not use types from other L1 constructs \(some may have the same name, but they are not the same type\)\.   
Some of our language\-specific API references currently have errors in the paths to L1 property types, or don't document these classes at all\. We hope to fix this soon\. In the meantime, just remember that such types are always inner classes of the L1 construct they are used with\.

## Using L2 constructs<a name="constructs_using"></a>

The following example defines an Amazon S3 bucket by creating an instance of the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) class, an L2 construct\.

------
#### [ TypeScript ]

```
import * as s3 from '@aws-cdk/aws-s3';

// "this" is HelloCdkStack
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

------
#### [ JavaScript ]

```
const s3 = require('@aws-cdk/aws-s3');

// "this" is HelloCdkStack
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

------
#### [ Python ]

```
from aws_cdk import aws_s3 as s3

# "self" is HelloCdkStack
s3.Bucket(self, "MyFirstBucket", versioned=True)
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.s3.*;

public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
                .versioned(true).build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.S3;

// "this" is HelloCdkStack
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true
});
```

------

The [AWS Construct Library](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) includes constructs that represent many AWS resources\.

**Note**  
`MyFirstBucket` is not the name of the bucket that AWS CloudFormation creates\. It is a logical identifier given to the new construct\. See [Physical Names](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Resource.html#physicalname-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) for details\.

## Configuration<a name="constructs_config"></a>

Most constructs accept `props` as their third argument \(or in Python, keyword arguments\), a name/value collection that defines the construct's configuration\. The following example defines a bucket with AWS Key Management Service \(AWS KMS\) encryption and static website hosting enabled\. Since it does not explicitly specify an encryption key, the `Bucket` construct defines a new `kms.Key` and associates it with the bucket\.

------
#### [ TypeScript ]

```
new s3.Bucket(this, 'MyEncryptedBucket', {
  encryption: s3.BucketEncryption.KMS,
  websiteIndexDocument: 'index.html'
});
```

------
#### [ JavaScript ]

```
new s3.Bucket(this, 'MyEncryptedBucket', {
  encryption: s3.BucketEncryption.KMS,
  websiteIndexDocument: 'index.html'
});
```

------
#### [ Python ]

```
s3.Bucket(self, "MyEncryptedBucket", encryption=s3.BucketEncryption.KMS,
    website_index_document="index.html")
```

------
#### [ Java ]

```
Bucket.Builder.create(this, "MyEncryptedBucket")
        .encryption(BucketEncryption.KMS_MANAGED)
        .websiteIndexDocument("index.html").build();
```

------
#### [ C\# ]

```
new Bucket(this, "MyEncryptedBucket", new BucketProps
{
    Encryption = BucketEncryption.KMS_MANAGED,
    WebsiteIndexDocument = "index.html"
});
```

------

 AWS constructs are designed around the concept of "sensible defaults\." Most constructs have a minimal required configuration, enabling you to quickly get started while also providing full control over the configuration when you need it\. 

## Interacting with constructs<a name="constructs_interact"></a>

Constructs are classes that extend the base [Construct](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html) class\. After you instantiate a construct, the construct object exposes a set of methods and properties that enable you to interact with the construct and pass it around as a reference to other parts of the system\. The AWS CDK framework doesn't put any restrictions on the APIs of constructs; authors can define any API they wish\. However, the AWS constructs that are included with the AWS Construct Library, such as `s3.Bucket`, follow guidelines and common patterns in order to provide a consistent experience across all AWS resources\.

For example, almost all AWS constructs have a set of [grant](permissions.md#permissions_grants) methods that you can use to grant AWS Identity and Access Management \(IAM\) permissions on that construct to a principal\. The following example grants the IAM group `data-science` permission to read from the Amazon S3 bucket `raw-data`\.

------
#### [ TypeScript ]

```
const rawData = new s3.Bucket(this, 'raw-data');
const dataScience = new iam.Group(this, 'data-science');
rawData.grantRead(dataScience);
```

------
#### [ JavaScript ]

```
const rawData = new s3.Bucket(this, 'raw-data');
const dataScience = new iam.Group(this, 'data-science');
rawData.grantRead(dataScience);
```

------
#### [ Python ]

```
raw_data = s3.Bucket(self, 'raw-data')
data_science = iam.Group(self, 'data-science')
raw_data.grant_read(data_science)
```

------
#### [ Java ]

```
Bucket rawData = new Bucket(this, "raw-data");
Group dataScience = new Group(this, "data-science");
rawData.grantRead(dataScience);
```

------
#### [ C\# ]

```
var rawData = new Bucket(this, "raw-data");
var dataScience = new Group(this, "data-science");
rawData.GrantRead(dataScience);
```

------

Another common pattern is for AWS constructs to set one of the resource's attributes, such as its Amazon Resource Name \(ARN\), name, or URL from data supplied elsewhere\. For example, the following code defines an AWS Lambda function and associates it with an Amazon Simple Queue Service \(Amazon SQS\) queue through the queue's URL in an environment variable\.

------
#### [ TypeScript ]

```
const jobsQueue = new sqs.Queue(this, 'jobs');
const createJobLambda = new lambda.Function(this, 'create-job', {
  runtime: lambda.Runtime.NODEJS_10_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('./create-job-lambda-code'),
  environment: {
    QUEUE_URL: jobsQueue.queueUrl
  }
});
```

------
#### [ JavaScript ]

```
const jobsQueue = new sqs.Queue(this, 'jobs');
const createJobLambda = new lambda.Function(this, 'create-job', {
  runtime: lambda.Runtime.NODEJS_10_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('./create-job-lambda-code'),
  environment: {
    QUEUE_URL: jobsQueue.queueUrl
  }
});
```

------
#### [ Python ]

```
jobs_queue = sqs.Queue(self, "jobs")
create_job_lambda = lambda_.Function(self, "create-job",
    runtime=lambda_.Runtime.NODEJS_10_X,
    handler="index.handler",
    code=lambda_.Code.from_asset("./create-job-lambda-code"),
    environment=dict(
        QUEUE_URL=jobs_queue.queue_url
    )
)
```

------
#### [ Java ]

```
final Queue jobsQueue = new Queue(this, "jobs");
Function createJobLambda = Function.Builder.create(this, "create-job")
                .handler("index.handler")
                .code(Code.fromAsset("./create-job-lambda-code"))
                .environment(new HashMap<String, String>() {{
                        put("QUEUE_URL", jobsQueue.getQueueUrl());
                }}).build();
```

------
#### [ C\# ]

```
var jobsQueue = new Queue(this, "jobs");
var createJobLambda = new Function(this, "create-job", new FunctionProps
{
    Runtime = Runtime.NODEJS_10_X,
    Handler = "index.handler",
    Code = Code.FromAsset(@".\create-job-lambda-code"),
    Environment = new Dictionary<string, string>
    {
        ["QUEUE_URL"] = jobsQueue.QueueUrl
    }
});
```

------

For information about the most common API patterns in the AWS Construct Library, see [Resources](resources.md)\.

## Authoring constructs<a name="constructs_author"></a>

In addition to using existing constructs like `s3.Bucket`, you can also author your own constructs, and then anyone can use them in their apps\. All constructs are equal in the AWS CDK\. An AWS CDK construct such as `s3.Bucket` or `sns.Topic` behaves the same as a construct imported from a third\-party library that someone published on npm or Maven or PyPI—or to your company's internal package repository\.

To declare a new construct, create a class that extends the [Construct](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Construct.html) base class, then follow the pattern for initializer arguments\.

For example, you could declare a construct that represents an Amazon S3 bucket which sends an Amazon Simple Notification Service \(Amazon SNS\) notification every time someone uploads a file into it:

------
#### [ TypeScript ]

```
export interface NotifyingBucketProps {
  prefix?: string;
}

export class NotifyingBucket extends Construct {
  constructor(scope: Construct, id: string, props: NotifyingBucketProps = {}) {
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    const topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(topic),
      { prefix: props.prefix });
  }
}
```

------
#### [ JavaScript ]

```
class NotifyingBucket extends Construct {
  constructor(scope, id, props = {}) {
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    const topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(topic),
      { prefix: props.prefix });
  }
}

module.exports = { NotifyingBucket }
```

------
#### [ Python ]

```
class NotifyingBucket(core.Construct):

    def __init__(self, scope: core.Construct, id: str, *, prefix=None):
        super().__init__(scope, id)
        bucket = s3.Bucket(self, "bucket")
        topic = sns.Topic(self, "topic")
        bucket.add_object_created_notification(s3notify.SnsDestination(topic),
            s3.NotificationKeyFilter(prefix=prefix))
```

------
#### [ Java ]

```
public class NotifyingBucket extends Construct {

    public NotifyingBucket(final Construct scope, final String id) {
        this(scope, id, null, null);
    }
    
    public NotifyingBucket(final Construct scope, final String id, final BucketProps props) {
        this(scope, id, props, null);
    }
    
    public NotifyingBucket(final Construct scope, final String id, final String prefix) {
        this(scope, id, null, prefix);
    }

    public NotifyingBucket(final Construct scope, final String id, final BucketProps props, final String prefix) {
        super(scope, id);

        Bucket bucket = new Bucket(this, "bucket");
        Topic topic = new Topic(this, "topic");
        if (prefix != null)
            bucket.addObjectCreatedNotification(new SnsDestination(topic),
                NotificationKeyFilter.builder().prefix(prefix).build());
     }
}
```

------
#### [ C\# ]

```
public class NotifyingBucketProps : BucketProps
{
    public string Prefix { get; set; }
}

public class NotifyingBucket : Construct
{
    public NotifyingBucket(Construct scope, string id, NotifyingBucketProps props = null) : base(scope, id)
    {
        var bucket = new Bucket(this, "bucket");
        var topic = new Topic(this, "topic");
        bucket.AddObjectCreatedNotification(new SnsDestination(topic), new NotificationKeyFilter
        {
            Prefix = props?.Prefix
        });
    }
}
```

------

The `NotifyingBucket` constructor has a signature compatible with the base `Construct` class: `scope`, `id`, and `props`\. The last argument, `props`, is optional \(gets the default value `{}`\) because all props are optional\. This means that you could define an instance of this construct in your app without `props`, for example: 

------
#### [ TypeScript ]

```
new NotifyingBucket(this, 'MyNotifyingBucket');
```

------
#### [ JavaScript ]

```
new NotifyingBucket(this, 'MyNotifyingBucket');
```

------
#### [ Python ]

```
NotifyingBucket(self, "MyNotifyingBucket")
```

------
#### [ Java ]

```
new NotifyingBucket(this, "MyNotifyingBucket");
```

------
#### [ C\# ]

```
new NotifyingBucket(this, "MyNotifyingBucket");
```

------

Or you could use `props` \(in Java, an additional parameter\) to specify the path prefix to filter on, for example:

------
#### [ TypeScript ]

```
new NotifyingBucket(this, 'MyNotifyingBucket', { prefix: 'images/' });
```

------
#### [ JavaScript ]

```
new NotifyingBucket(this, 'MyNotifyingBucket', { prefix: 'images/' });
```

------
#### [ Python ]

```
NotifyingBucket(self, "MyNotifyingBucket", prefix="images/")
```

------
#### [ Java ]

```
new NotifyingBucket(this, "MyNotifyingBucket", "/images");
```

------
#### [ C\# ]

```
new NotifyingBucket(this, "MyNotifyingBucket", new NotifyingBucketProps
{
    Prefix = "/images"
});
```

------

Typically, you would also want to expose some properties or methods on your constructs\. For example, it's not very useful to have a topic hidden behind your construct, because it wouldn't be possible for users of your construct to subscribe to it\. Adding a `topic` property allows consumers to access the inner topic, as shown in the following example:

------
#### [ TypeScript ]

```
export class NotifyingBucket extends Construct {
  public readonly topic: sns.Topic;

  constructor(scope: Construct, id: string, props: NotifyingBucketProps) {
    super(scope, id, props);
    const bucket = new s3.Bucket(this, 'bucket');
    this.topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(this.topic), { prefix: props.prefix });
  }
}
```

------
#### [ JavaScript ]

```
class NotifyingBucket extends Construct {

  constructor(scope, id, props) {
    super(scope, id, props);
    const bucket = new s3.Bucket(this, 'bucket');
    this.topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(this.topic), { prefix: props.prefix });
  }
}

module.exports = { NotifyingBucket }
```

------
#### [ Python ]

```
class NotifyingBucket(core.Construct):

    def __init__(self, scope: core.Construct, id: str, *, prefix=None, **kwargs):
        super().__init__(scope, id, **kwargs)
        bucket = s3.Bucket(self, "bucket")
        self.topic = sns.Topic(self, "topic")
        bucket.add_object_created_notification(s3notify.SnsDestination(self.topic),
            s3.NotificationKeyFilter(prefix=prefix))
```

------
#### [ Java ]

```
public class NotifyingBucket extends Bucket {

    public Topic topic = null;
    
    public NotifyingBucket(final Construct scope, final String id) {
        this(scope, id, null, null);
    }
    
    public NotifyingBucket(final Construct scope, final String id, final BucketProps props) {
        this(scope, id, props, null);
    }
    
    public NotifyingBucket(final Construct scope, final String id, final String prefix) {
        this(scope, id, null, prefix);
    }

    public NotifyingBucket(final Construct scope, final String id, final BucketProps props, final String prefix) {
        super(scope, id, props);

        Bucket bucket = new Bucket(this, "bucket");
        topic = new Topic(this, "topic");
        if (prefix != null)
            bucket.addObjectCreatedNotification(new SnsDestination(topic),
                NotificationKeyFilter.builder().prefix(prefix).build());
     }
}
```

------
#### [ C\# ]

```
public class NotifyingBucket : Construct
{
    public readonly Topic topic;

    public NotifyingBucket(Construct scope, string id, NotifyingBucketProps props = null) : base(scope, id, props)
    {
        var bucket = new Bucket(this, "bucket");
        topic = new Topic(this, "topic");
        bucket.AddObjectCreatedNotification(new SnsDestination(topic), new NotificationKeyFilter
        {
            Prefix = props?.Prefix
        });
    }
}
```

------

Now, consumers can subscribe to the topic, for example:

------
#### [ TypeScript ]

```
const queue = new sqs.Queue(this, 'NewImagesQueue');
const images = new NotifyingBucket(this, '/images');
images.topic.addSubscription(new sns_sub.SqsSubscription(queue));
```

------
#### [ JavaScript ]

```
const queue = new sqs.Queue(this, 'NewImagesQueue');
const images = new NotifyingBucket(this, '/images');
images.topic.addSubscription(new sns_sub.SqsSubscription(queue));
```

------
#### [ Python ]

```
queue = sqs.Queue(self, "NewImagesQueue")
images = NotifyingBucket(self, prefix="Images")
images.topic.add_subscription(sns_sub.SqsSubscription(queue))
```

------
#### [ Java ]

```
NotifyingBucket images = new NotifyingBucket(this, "MyNotifyingBucket", "/images");
images.topic.addSubscription(new SqsSubscription(queue));
```

------
#### [ C\# ]

```
var queue = new Queue(this, "NewImagesQueue");
var images = new NotifyingBucket(this, "MyNotifyingBucket", new NotifyingBucketProps
{
    Prefix = "/images"
});
images.topic.AddSubscription(new SqsSubscription(queue));
```

------