# AWS CDK Constructs<a name="constructs"></a>

Constructs are the basic building blocks of AWS Cloud Development Kit \(AWS CDK\) applications\. A construct is a component within your application that represents one or more AWS CloudFormation resources and their configuration\. You build your application, piece by piece, by importing and configuring constructs\.

## Import and use constructs<a name="constructs-import"></a>

Constructs are classes that you import into your CDK applications from the [AWS Construct Library](libraries.md#libraries-construct)\. You can also create and distribute your own constructs, or use constructs created by third\-party developers\.

Constructs are part of the Construct Programming Model \(CPM\)\. They are available to use with other tools such as CDK for Terraform \(CDKtf\), CDK for Kubernetes \(CDK8s\), and Projen\.

Numerous third parties have also published constructs compatible with the AWS CDK\. Visit [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&offset=0) to explore the AWS CDK construct partner ecosystem\.

## Construct levels<a name="constructs_lib_levels"></a>

Constructs from the AWS Construct Library are categorized into three levels\. Each level offers an increasing level of abstraction\. The higher the abstraction, the easier to configure, requiring less expertise\. The lower the abstraction, the more customization available, requiring more expertise\.

**Level 1 \(L1\) constructs**  <a name="constructs_lib_levels_one"></a>
L1 constructs, also known as *CFN resources*, are the lowest\-level construct and offer no abstraction\. Each L1 construct maps directly to a single AWS CloudFormation resource\. With L1 constructs, you import a construct that represents a specific AWS CloudFormation resource\. You then define the resource’s properties within your construct instance\.  
L1 constructs are great to use when you are familiar with AWS CloudFormation and need complete control over defining your AWS resource properties\.  
In the AWS Construct Library, L1 constructs are named starting with `Cfn`, followed by an identifier for the AWS CloudFormation resource that it represents\. For example, the `[CfnBucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.CfnBucket.html)` construct is an L1 construct that represents an `[AWS::S3::Bucket](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-s3-bucket.html)` AWS CloudFormation resource\.  
L1 constructs are generated from the [AWS CloudFormation resource specification](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-resource-specification.html)\. If a resource exists in AWS CloudFormation, it'll be available in the AWS CDK as an L1 construct\. New resources or properties may take up to a week to become available in the AWS Construct Library\. For more information, see [AWS resource and property types reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) in the *AWS CloudFormation User Guide*\.

**Level 2 \(L2\) constructs**  <a name="constructs_lib_levels_two"></a>
L2 constructs, also known as *curated* constructs, are thoughtfully developed by the CDK team and are usually the most widely used construct type\. L2 constructs map directly to single AWS CloudFormation resources, similar to L1 constructs\. Compared to L1 constructs, L2 constructs provide a higher\-level abstraction through an intuitive intent\-based API\. L2 constructs include sensible default property configurations, best practice security policies, and generate a lot of the boilerplate code and glue logic for you\.  
L2 constructs also provide helper methods for most resources that make it simpler and quicker to define properties, permissions, event\-based interactions between resources, and more\.  
The `[s3\.Bucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html)` class is an example of an L2 construct for an Amazon Simple Storage Service \(Amazon S3\) bucket resource\.  
The AWS Construct Library contains L2 constructs that are designated stable and ready for production use\. For L2 constructs under development, they are designated as experimental and offered in a separate module\.

**Level 3 \(L3\) constructs**  <a name="constructs_lib_levels_three"></a>
L3 constructs, also known as *patterns*, are the highest\-level of abstraction\. Each L3 construct can contain a collection of resources that are configured to work together to accomplish a specific task or service within your application\. L3 constructs are used to create entire AWS architectures for particular use cases in your application\.  
To provide complete system designs, or substantial parts of a larger system, L3 constructs offer opinionated default property configurations\. They are built around a particular approach toward solving a problem and providing a solution\. With L3 constructs, you can create and configure multiple resources quickly, with the fewest amount of input and code\.  
The `[ecsPatterns\.ApplicationLoadBalancedFargateService](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ecs_patterns.ApplicationLoadBalancedFargateService.html)` class is an example of an L3 construct that represents an AWS Fargate service running on an Amazon Elastic Container Service \(Amazon ECS\) cluster and fronted by an application load balancer\.  
Similar to L2 constructs, L3 constructs that are ready for production use are included in the AWS Construct Library\. Those under development are offered in separate modules\.

## Defining constructs<a name="constructs_define"></a>

### Composition<a name="constructs_composition"></a>

*Composition* is the key pattern for defining higher\-level abstractions through constructs\. A high\-level construct can be composed from any number of lower\-level constructs\. From a bottom\-up perspective, you use constructs to organize the individual AWS resources that you want to deploy\. You use whatever abstractions are convenient for your purpose, with as many levels as you need\.

With composition, you define reusable components and share them like any other code\. For example, a team can define a construct that implements the company’s best practice for an Amazon DynamoDB table, including backup, global replication, automatic scaling, and monitoring\. The team can share the construct internally with other teams, or publicly\.

Teams can use constructs like any other library package\. When the library is updated, developers get access to the new version’s improvements and bug fixes, similar to any other code library\.

### Initialization<a name="constructs_init"></a>

Constructs are implemented in classes that extend the [https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html) base class\. You define a construct by instantiating the class\. All constructs take three parameters when they are initialized:
+ **scope** – The construct's parent or owner\. This can either be a stack or another construct\. Scope determines the construct's place in the [construct tree](apps.md#apps-tree)\. You should usually pass `this` \(`self` in Python\), which represents the current object, for the scope\.
+ **id** – An [identifier](identifiers.md) that must be unique within the scope\. The identifier serves as a namespace for everything that’s defined within the construct\. It’s used to generate unique identifiers, such as [resource names](resources.md#resources_physical_names) and AWS CloudFormation logical IDs\.

  Identifiers need only be unique within a scope\. This lets you instantiate and reuse constructs without concern for the constructs and identifiers they might contain, and enables composing constructs into higher\-level abstractions\. In addition, scopes make it possible to refer to groups of constructs all at once\. Examples include for [tagging](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tag.html), or specifying where the constructs will be deployed\.
+ **props** – A set of properties or keyword arguments, depending on the language, that define the construct’s initial configuration\. Higher\-level constructs provide more defaults, and if all prop elements are optional, you can omit the props parameter completely\.

### Configuration<a name="constructs_config"></a>

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
#### [ Go ]

```
	awss3.NewBucket(stack, jsii.String("MyEncryptedBucket"), &awss3.BucketProps{
		Encryption: awss3.BucketEncryption_KMS,
		WebsiteIndexDocument: jsii.String("index.html"),
	})
```

------

### Interacting with constructs<a name="constructs_interact"></a>

Constructs are classes that extend the base [Construct](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html) class\. After you instantiate a construct, the construct object exposes a set of methods and properties that let you interact with the construct and pass it around as a reference to other parts of the system\.

The AWS CDK framework doesn't put any restrictions on the APIs of constructs\. Authors can define any API they want\. However, the AWS constructs that are included with the AWS Construct Library, such as `s3.Bucket`, follow guidelines and common patterns\. This provides a consistent experience across all AWS resources\.

Most AWS constructs have a set of [grant](permissions.md#permissions_grants) methods that you can use to grant AWS Identity and Access Management \(IAM\) permissions on that construct to a principal\. The following example grants the IAM group `data-science` permission to read from the Amazon S3 bucket `raw-data`\.

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
#### [ Go ]

```
	rawData := awss3.NewBucket(stack, jsii.String("raw-data"), nil)
	dataScience := awsiam.NewGroup(stack, jsii.String("data-science"), nil)
	rawData.GrantRead(dataScience, nil)
```

------

Another common pattern is for AWS constructs to set one of the resource's attributes from data supplied elsewhere\. Attributes can include Amazon Resource Names \(ARNs\), names, or URLs\.

The following code defines an AWS Lambda function and associates it with an Amazon Simple Queue Service \(Amazon SQS\) queue through the queue's URL in an environment variable\.

------
#### [ TypeScript ]

```
const jobsQueue = new sqs.Queue(this, 'jobs');
const createJobLambda = new lambda.Function(this, 'create-job', {
  runtime: lambda.Runtime.NODEJS_18_X,
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
  runtime: lambda.Runtime.NODEJS_18_X,
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
    runtime=lambda_.Runtime.NODEJS_18_X,
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
                .environment(java.util.Map.of(   // Map.of is Java 9 or later
                    "QUEUE_URL", jobsQueue.getQueueUrl())
                .build();
```

------
#### [ C\# ]

```
var jobsQueue = new Queue(this, "jobs");
var createJobLambda = new Function(this, "create-job", new FunctionProps
{
    Runtime = Runtime.NODEJS_18_X,
    Handler = "index.handler",
    Code = Code.FromAsset(@".\create-job-lambda-code"),
    Environment = new Dictionary<string, string>
    {
        ["QUEUE_URL"] = jobsQueue.QueueUrl
    }
});
```

------
#### [ Go ]

```
	createJobLambda := awslambda.NewFunction(stack, jsii.String("create-job"), &awslambda.FunctionProps{
		Runtime: awslambda.Runtime_NODEJS_18_X(),
		Handler: jsii.String("index.handler"),
		Code:    awslambda.Code_FromAsset(jsii.String(".\\create-job-lambda-code"), nil),
		Environment: &map[string]*string{
			"QUEUE_URL": jsii.String(*jobsQueue.QueueUrl()),
		},
	})
```

------

For information about the most common API patterns in the AWS Construct Library, see [Resources and the AWS CDK](resources.md)\.

### The app and stack construct<a name="constructs_apps_stacks"></a>

The `[App](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html)` and `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` classes from the AWS Construct Library are unique constructs\. Compared to other constructs, they don't configure AWS resources on their own\. Instead, they are used to provide context for your other constructs\. All constructs that represent AWS resources must be defined, directly or indirectly, within the scope of a `Stack` construct\. `Stack` constructs are defined within the scope of an `App` construct\.

To learn more about CDK apps, see [AWS CDK apps](apps.md)\. To learn more about CDK stacks, see [Introduction to AWS CDK stacks](stacks.md)\.

The following example defines an app with a single stack\. Within the stack, an L2 construct is used to configure an Amazon S3 bucket resource\.

------
#### [ TypeScript ]

```
import { App, Stack, StackProps } from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';

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
const { App , Stack } = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');

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
from aws_cdk import App, Stack
import aws_cdk.aws_s3 as s3
from constructs import Construct

class HelloCdkStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        s3.Bucket(self, "MyFirstBucket", versioned=True)

app = App()
HelloCdkStack(app, "HelloCdkStack")
```

------
#### [ Java ]

Stack defined in `HelloCdkStack.java` file:

```
import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
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

App defined in `HelloCdkApp.java` file:

```
import software.amazon.awscdk.App;
import software.amazon.awscdk.StackProps;

public class HelloCdkApp {
    public static void main(final String[] args) {
        App app = new App();

        new HelloCdkStack(app, "HelloCdkStack", StackProps.builder()
                .build());

        app.synth();
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
#### [ Go ]

```
func NewHelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	awss3.NewBucket(stack, jsii.String("MyFirstBucket"), &awss3.BucketProps{
		Versioned: jsii.Bool(true),
	})

	return stack
}
```

------

## Working with constructs<a name="constructs-work"></a>

### Working with L1 constructs<a name="constructs_l1_using"></a>

L1 constructs map directly to individual AWS CloudFormation resources\. You must provide the resource's required configuration\.

In this example, we create a `bucket` object using the `CfnBucket` L1 construct:

------
#### [ TypeScript ]

```
const bucket = new s3.CfnBucket(this, "amzn-s3-demo-bucket", {
  bucketName: "amzn-s3-demo-bucket"
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.CfnBucket(this, "amzn-s3-demo-bucket", {
  bucketName: "amzn-s3-demo-bucket"
});
```

------
#### [ Python ]

```
bucket = s3.CfnBucket(self, "amzn-s3-demo-bucket", bucket_name="amzn-s3-demo-bucket")
```

------
#### [ Java ]

```
CfnBucket bucket = new CfnBucket.Builder().bucketName("amzn-s3-demo-bucket").build();
```

------
#### [ C\# ]

```
var bucket = new CfnBucket(this, "amzn-s3-demo-bucket", new CfnBucketProps
{
    BucketName= "amzn-s3-demo-bucket"
});
```

------
#### [ Go ]

```
	awss3.NewCfnBucket(stack, jsii.String("amzn-s3-demo-bucket"), &awss3.CfnBucketProps{
		BucketName: jsii.String("amzn-s3-demo-bucket"),
	})
```

------

Construct properties that aren't simple Booleans, strings, numbers, or containers are handled differently in the supported languages\.

------
#### [ TypeScript ]

```
const bucket = new s3.CfnBucket(this, "amzn-s3-demo-bucket", {
  bucketName: "amzn-s3-demo-bucket",
  corsConfiguration: {
    corsRules: [{
          allowedOrigins: ["*"],
          allowedMethods: ["GET"]
    }]
  }
});
```

------
#### [ JavaScript ]

```
const bucket = new s3.CfnBucket(this, "amzn-s3-demo-bucket", {
  bucketName: "amzn-s3-demo-bucket",
  corsConfiguration: {
    corsRules: [{
          allowedOrigins: ["*"],
          allowedMethods: ["GET"]
    }]
  }
});
```

------
#### [ Python ]

In Python, these properties are represented by types defined as inner classes of the L1 construct\. For example, the optional property `cors_configuration` of a `CfnBucket` requires a wrapper of type `CfnBucket.CorsConfigurationProperty`\. Here we are defining `cors_configuration` on a `CfnBucket` instance\.

```
bucket = CfnBucket(self, "amzn-s3-demo-bucket", bucket_name="amzn-s3-demo-bucket",
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

In Java, these properties are represented by types defined as inner classes of the L1 construct\. For example, the optional property `corsConfiguration` of a `CfnBucket` requires a wrapper of type `CfnBucket.CorsConfigurationProperty`\. Here we are defining `corsConfiguration` on a `CfnBucket` instance\.

```
CfnBucket bucket = CfnBucket.Builder.create(this, "amzn-s3-demo-bucket")
                        .bucketName("amzn-s3-demo-bucket")
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

In C\#, these properties are represented by types defined as inner classes of the L1 construct\. For example, the optional property `CorsConfiguration` of a `CfnBucket` requires a wrapper of type `CfnBucket.CorsConfigurationProperty`\. Here we are defining `CorsConfiguration` on a `CfnBucket` instance\.

```
var bucket = new CfnBucket(this, "amzn-s3-demo-bucket", new CfnBucketProps
{
    BucketName = "amzn-s3-demo-bucket",
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
#### [ Go ]

In Go, these types are named using the name of the L1 construct, an underscore, and the property name\. For example, the optional property `CorsConfiguration` of a `CfnBucket` requires a wrapper of type `CfnBucket_CorsConfigurationProperty`\. Here we are defining `CorsConfiguration` on a `CfnBucket` instance\.

```
	awss3.NewCfnBucket(stack, jsii.String("amzn-s3-demo-bucket"), &awss3.CfnBucketProps{
		BucketName: jsii.String("amzn-s3-demo-bucket"),
		CorsConfiguration: &awss3.CfnBucket_CorsConfigurationProperty{
			CorsRules: []awss3.CorsRule{
				awss3.CorsRule{
					AllowedOrigins: jsii.Strings("*"),
					AllowedMethods: &[]awss3.HttpMethods{"GET"},
				},
			},
		},
	})
```

------

**Important**  
You can't use L2 property types with L1 constructs, or vice versa\. When working with L1 constructs, always use the types defined for the L1 construct you're using\. Do not use types from other L1 constructs \(some may have the same name, but they are not the same type\)\.  
Some of our language\-specific API references currently have errors in the paths to L1 property types, or don't document these classes at all\. We hope to fix this soon\. In the meantime, remember that such types are always inner classes of the L1 construct they are used with\.

### Working with L2 constructs<a name="constructs_using"></a>

In the following example, we define an Amazon S3 bucket by creating an object from the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) L2 construct:

------
#### [ TypeScript ]

```
import * as s3 from 'aws-cdk-lib/aws-s3';

// "this" is HelloCdkStack
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

------
#### [ JavaScript ]

```
const s3 = require('aws-cdk-lib/aws-s3');

// "this" is HelloCdkStack
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true
});
```

------
#### [ Python ]

```
import aws_cdk.aws_s3 as s3

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
#### [ Go ]

```
import (
	"github.com/aws/aws-cdk-go/awscdk/v2/awss3"
	"github.com/aws/jsii-runtime-go"
)

// stack is HelloCdkStack
awss3.NewBucket(stack, jsii.String("MyFirstBucket"), &awss3.BucketProps{
		Versioned: jsii.Bool(true),
	})>
```

------

`MyFirstBucket` is not the name of the bucket that AWS CloudFormation creates\. It is a logical identifier given to the new construct within the context of your CDK app\. The [physicalName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Resource.html#physicalname) value will be used to name the AWS CloudFormation resource\.

## Working with third\-party constructs<a name="constructs-work-third"></a>

[Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0) is a resource to help you discover additional constructs from AWS, third parties, and the open\-source CDK community\.

### Writing your own constructs<a name="constructs_author"></a>

In addition to using existing constructs, you can also write your own constructs and let anyone use them in their apps\. All constructs are equal in the AWS CDK\. Constructs from the AWS Construct Library are treated the same as a construct from a third\-party library published via NPM, Maven, or PyPI\. Constructs published to your company's internal package repository are also treated in the same way\.

To declare a new construct, create a class that extends the [Construct](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html) base class, in the `constructs` package, then follow the pattern for initializer arguments\.

The following example shows how to declare a construct that represents an Amazon S3 bucket\. The S3 bucket sends an Amazon Simple Notification Service \(Amazon SNS\) notification every time someone uploads a file into it\.

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
class NotifyingBucket(Construct):

    def __init__(self, scope: Construct, id: str, *, prefix=None):
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
#### [ Go ]

```
type NotifyingBucketProps struct {
	awss3.BucketProps
	Prefix *string
}

func NewNotifyingBucket(scope constructs.Construct, id *string, props *NotifyingBucketProps) awss3.Bucket {
	var bucket awss3.Bucket
	if props == nil {
		bucket = awss3.NewBucket(scope, jsii.String(*id+"Bucket"), nil)
	} else {
		bucket = awss3.NewBucket(scope, jsii.String(*id+"Bucket"), &props.BucketProps)
	}
	topic := awssns.NewTopic(scope, jsii.String(*id+"Topic"), nil)
	if props == nil {
		bucket.AddObjectCreatedNotification(awss3notifications.NewSnsDestination(topic))
	} else {
		bucket.AddObjectCreatedNotification(awss3notifications.NewSnsDestination(topic), &awss3.NotificationKeyFilter{
			Prefix: props.Prefix,
		})
	}
	return bucket
}
```

------

**Note**  
Our `NotifyingBucket` construct inherits not from `Bucket` but rather from `Construct`\. We are using composition, not inheritance, to bundle an Amazon S3 bucket and an Amazon SNS topic together\. In general, composition is preferred over inheritance when developing AWS CDK constructs\.

The `NotifyingBucket` constructor has a typical construct signature: `scope`, `id`, and `props`\. The last argument, `props`, is optional \(gets the default value `{}`\) because all props are optional\. \(The base `Construct` class does not take a `props` argument\.\) You could define an instance of this construct in your app without `props`, for example:

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
#### [ Go ]

```
NewNotifyingBucket(stack, jsii.String("MyNotifyingBucket"), nil)
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
#### [ Go ]

```
NewNotifyingBucket(stack, jsii.String("MyNotifyingBucket"), &NotifyingBucketProps{
	Prefix: jsii.String("images/"),
})
```

------

Typically, you would also want to expose some properties or methods on your constructs\. It's not very useful to have a topic hidden behind your construct, because users of your construct aren't able to subscribe to it\. Adding a `topic` property lets consumers access the inner topic, as shown in the following example:

------
#### [ TypeScript ]

```
export class NotifyingBucket extends Construct {
  public readonly topic: sns.Topic;

  constructor(scope: Construct, id: string, props: NotifyingBucketProps) {
    super(scope, id);
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
    super(scope, id);
    const bucket = new s3.Bucket(this, 'bucket');
    this.topic = new sns.Topic(this, 'topic');
    bucket.addObjectCreatedNotification(new s3notify.SnsDestination(this.topic), { prefix: props.prefix });
  }
}

module.exports = { NotifyingBucket };
```

------
#### [ Python ]

```
class NotifyingBucket(Construct):

    def __init__(self, scope: Construct, id: str, *, prefix=None, **kwargs):
        super().__init__(scope, id)
        bucket = s3.Bucket(self, "bucket")
        self.topic = sns.Topic(self, "topic")
        bucket.add_object_created_notification(s3notify.SnsDestination(self.topic),
            s3.NotificationKeyFilter(prefix=prefix))
```

------
#### [ Java ]

```
public class NotifyingBucket extends Construct {

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
        super(scope, id);

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

    public NotifyingBucket(Construct scope, string id, NotifyingBucketProps props = null) : base(scope, id)
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
#### [ Go ]

To do this in Go, we'll need a little extra plumbing\. Our original `NewNotifyingBucket` function returned an `awss3.Bucket`\. We'll need to extend `Bucket` to include a `topic` member by creating a `NotifyingBucket` struct\. Our function will then return this type\.

```
type NotifyingBucket struct {
	awss3.Bucket
	topic awssns.Topic
}

func NewNotifyingBucket(scope constructs.Construct, id *string, props *NotifyingBucketProps) NotifyingBucket {
	var bucket awss3.Bucket
	if props == nil {
		bucket = awss3.NewBucket(scope, jsii.String(*id+"Bucket"), nil)
	} else {
		bucket = awss3.NewBucket(scope, jsii.String(*id+"Bucket"), &props.BucketProps)
	}
	topic := awssns.NewTopic(scope, jsii.String(*id+"Topic"), nil)
	if props == nil {
		bucket.AddObjectCreatedNotification(awss3notifications.NewSnsDestination(topic))
	} else {
		bucket.AddObjectCreatedNotification(awss3notifications.NewSnsDestination(topic), &awss3.NotificationKeyFilter{
			Prefix: props.Prefix,
		})
	}
	var nbucket NotifyingBucket
	nbucket.Bucket = bucket
	nbucket.topic = topic
	return nbucket
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
#### [ Go ]

```
	queue := awssqs.NewQueue(stack, jsii.String("NewImagesQueue"), nil)
	images := NewNotifyingBucket(stack, jsii.String("MyNotifyingBucket"), &NotifyingBucketProps{
		Prefix: jsii.String("/images"),
	})
	images.topic.AddSubscription(awssnssubscriptions.NewSqsSubscription(queue, nil))
```

------

## Learn more<a name="constructs-learn"></a>

The following video provides a comprehensive overview of CDK constructs, and explains how you can use them in your CDK apps\.