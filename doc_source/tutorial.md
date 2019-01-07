--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# AWS CDK Tutorial<a name="tutorial"></a>

This topic walks you through creating and deploying an AWS CDK app, from initializing the project to deploying the resulting AWS CloudFormation template\.

## Creating the Project<a name="tutorial_create_directory"></a>

Create a directory for your project with an empty Git repository\.

```
mkdir hello-cdk
cd hello-cdk
```

## Initializing the Project<a name="tutorial_init_project"></a>

Initialize an empty project, where *LANGUAGE* is one of the supported programming languages: **csharp** \(C\#\), **java** \(Java\), or **typescript** \(TypeScript\)\.

```
cdk init --language LANGUAGE
```

## Compiling the Project<a name="tutorial_compile_project"></a>

Compile your program:

------
#### [ C\# ]

Since configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command:

```
cdk
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ Java ]

```
mvn compile
```

------

## Listing the Stacks in the App<a name="tutorial_list_stacks"></a>

List the stacks in the app\.

```
cdk ls
```

The result is just the name of the stack:

```
HelloCdkStack
```

**Note**  
There is a known issue on Windows with the AWS CDK \.NET environment\. Whenever you use a cdk command, it issues a node warning similar to the following:  

```
(node:27508) UnhandledPromiseRejectionWarning: Unhandled promise rejection
(rejection id: 1): Error: EPIPE: broken pipe, write
(node:27508) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated.
In the future, promise rejections that are not handled will terminate the
Node.js process with a non-zero exit code.
```

You can safely ignore this warning\.

## Adding an Amazon S3 Bucket<a name="tutorial_add_bucket"></a>

What can you do with this app? Nothing\. Since the stack is empty, there's nothing to deploy\. Let's define an Amazon S3 bucket\.

Install the `@aws-cdk/aws-s3` package:

------
#### [ C\# ]

```
dotnet add package Amazon.CDK.AWS.S3
```

------
#### [ JavaScript ]

```
npm install @aws-cdk/aws-s3
```

------
#### [ TypeScript ]

```
npm install @aws-cdk/aws-s3
```

------
#### [ Java ]

Add the following to `pom.xml`, where *CDK\-VERSION* is the version of the AWS CDK:

```
<dependency>
  <groupId>software.amazon.awscdk</groupId>
  <artifactId>s3</artifactId>
  <version>CDK-VERSION</version>
</dependency>
```

------

Next, define an Amazon S3 bucket in the stack\. Amazon S3 buckets are represented by the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.Bucket) class:

------
#### [ C\# ]

Create `MyStack.cs`:

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class MyStack : Stack
    {
        public MyStack(App parent, string name) : base(parent, name, null)
        {
            new Bucket(this, "MyFirstBucket", new BucketProps
            {
                Versioned = true
            });
        }
    }
}
```

------
#### [ JavaScript ]

In `index.js`:

```
const cdk = require('@aws-cdk/cdk');
const s3 = require('@aws-cdk/aws-s3');

class MyStack extends cdk.Stack {
    constructor(parent, id, props) {
        super(parent, id, props);

        new s3.Bucket(this, 'MyFirstBucket', {
            versioned: true
        });
    }
}
```

------
#### [ TypeScript ]

In `lib/hello-cdk-stack.ts`:

```
import cdk = require('@aws-cdk/cdk');
import s3 = require('@aws-cdk/aws-s3');

export class HelloCdkStack extends cdk.Stack {
    constructor(parent: cdk.App, id: string, props?: cdk.StackProps) {
        super(parent, id, props);

        new s3.Bucket(this, 'MyFirstBucket', {
            versioned: true
        });
    }
}
```

------
#### [ Java ]

In `src/main/java/com/acme/MyStack.java`:

```
package com.acme;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.s3.BucketProps;

public class MyStack extends Stack {
    public MyStack(final App parent, final String name) {
        this(parent, name, null);
    }

    public MyStack(final App parent, final String name, final StackProps props) {
        super(parent, name, props);

        new Bucket(this, "MyFirstBucket", BucketProps.builder()
            .withVersioned(true)
            .build());
    }
}
```

------

A few things to notice:
+ [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.Bucket) is a construct\. This means it's initialization signature has **parent**, **id**, and **props**\. In this case, the bucket is an immediate child of **MyStack**\.
+ `MyFirstBucket` is the **logical id** of the bucket construct, not the physical name of the Amazon S3 bucket\. The logical ID is used to uniquely identify resources in your stack across deployments\. See [Logical IDs](logical_ids.md) for information on logical IDs\. To specify a physical name for your bucket, set the [bucketName](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketProps.bucketName) property when you define your bucket\.
+ Since the bucket's [versioned](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.versioned) property is `true`, [versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) is enabled on the bucket\.

Compile your program:

------
#### [ C\# ]

Since configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command:

```
cdk
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ Java ]

```
mvn compile
```

------

## Synthesizing an AWS CloudFormation Template<a name="tutorial_synth_template"></a>

Synthesize a AWS CloudFormation template for the stack:

```
cdk synth HelloCdkStack
```

**Note**  
Since the AWS CDK app only contains a single stack, you can omit `HelloCdkStack`\.

This command executes the AWS CDK app and synthesize an AWS CloudFormation template for the **HelloCdkStack** stack\. You should see something similar to the following, where *VERSION* is the version of the AWS CDK\.

```
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: "@aws-cdk/aws-codepipeline-api=VERSION,@aws-cdk/aws-events=VERSION,@aws-c\
        dk/aws-iam=VERSION,@aws-cdk/aws-kms=VERSION,@aws-cdk/aws-s3=VERSION,@aws-c\
        dk/aws-s3-notifications=VERSION,@aws-cdk/cdk=VERSION,@aws-cdk/cx-api=VERSION\
        .0,hello-cdk=0.1.0"
```

You can see that the stack contains an **AWS::S3::Bucket** resource with the desired versioning configuration\.

**Note**  
The **AWS::CDK::Metadata** resource was automatically added to your template by the toolkit\. This allows us to learn which libraries were used in your stack\. See [Version Reporting](tools.md#version_reporting) for details, including how to [opt out](tools.md#version_reporting_opt_out)\.

## Deploying the Stack<a name="tutorial_deploy_stack"></a>

Deploy the stack:

```
cdk deploy
```

The deploy command synthesizes an AWS CloudFormation template from the stack and then invokes the AWS CloudFormation create/update API to deploy it into your AWS account\. The command displays information as it progresses\.

## Modifying the Code<a name="tutorial_modify_code"></a>

Configure the bucket to use KMS managed encryption:

------
#### [ C\# ]

Update `MyStack.cs`:

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true,
    Encryption = BucketEncryption.KmsManaged
});
```

------
#### [ JavaScript ]

Update `index.js`:

```
new s3.Bucket(this, 'MyFirstBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KmsManaged
});
```

------
#### [ TypeScript ]

Update `lib/hello-cdk-stack.ts`

```
new s3.Bucket(this, 'MyFirstBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KmsManaged
});
```

------
#### [ Java ]

Update `src/main/java/com/acme/MyStack.java`

```
new Bucket(this, "MyFirstBucket", BucketProps.builder()
    .withVersioned(true)
    .withEncryption(BucketEncryption.KmsManaged)
    .build());
```

------

Compile your program:

------
#### [ C\# ]

Since configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command:

```
cdk
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ Java ]

```
mvn compile
```

------

## Preparing for Deployment<a name="tutorial_prep_deployment"></a>

Before you deploy the updated stack, evaluate the difference between the AWS CDK app and the deployed stack:

```
cdk diff
```

The toolkit queries your AWS account for the current AWS CloudFormation template for the **hello\-cdk** stack, and compares the result with the template synthesized from the app\. The output should look like the following:

```
[~] 🛠 Updating MyFirstBucketB8884501 (type: AWS::S3::Bucket)
└─ [+] .BucketEncryption:
    └─ New value: {"ServerSideEncryptionConfiguration":[{"ServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}
```

As you can see, the diff indicates that the `ServerSideEncryptionConfiguration` property of the bucket is now set to enable server\-side encryption\.

You can also see that the bucket is not going to be replaced but rather updated \(**Updating MyFirstBucketB8884501**\)\.

Update the stack:

```
cdk deploy
```

The toolkit updates the bucket configuration to enable server\-side KMS encryption for the bucket:

```
⏳  Starting deployment of stack hello-cdk...
 [ 0/2 ] UPDATE_IN_PROGRESS   [AWS::S3::Bucket ] MyFirstBucketB8884501
 [ 1/2 ] UPDATE_COMPLETE      [AWS::S3::Bucket ] MyFirstBucketB8884501
 [ 1/2 ] UPDATE_COMPLETE_CLEANUP_IN_PROGRESS   [AWS::CloudFormation::Stack ] hello-cdk
 [ 2/2 ] UPDATE_COMPLETE      [AWS::CloudFormation::Stack ] hello-cdk
✅  Deployment of stack hello-cdk completed successfully
```

## What Next?<a name="tutorial_what_next"></a>
+ Learn more about [AWS CDK Concepts](concepts.md)
+ Check out the [examples directory](https://github.com/awslabs/aws-cdk/tree/master/examples) in your GitHub repository
+ Learn about the rich APIs offered by the [AWS Construct Library](aws_construct_lib.md)
+ Work directly with CloudFormation using the [AWS AWS CloudFormation Library](cloudformation.md)
+ Come talk to us on [Gitter](https://gitter.im/awslabs/aws-cdk)