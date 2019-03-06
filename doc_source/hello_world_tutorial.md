--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Hello World Tutorial<a name="hello_world_tutorial"></a>

The typical workflow for creating a new app is:

1. Create a directory and navigate to it\.

1. To create the skeleton of the app in the programming language *LANGUAGE*, call cdk init \-\-language *LANGUAGE*\.

1. Add constructs \(and stacks, if appropriate\) to the skeleton code\.

1. Compile your code, if necessary\.

1. To deploy the resources you've defined, call cdk deploy\.

1. Test your deployment\.

1. If there are any issues, loop through modify, compile \(if necessary\), deploy, and test again\.

And of course, keep your code under version control\.

This tutorial walks you through how to create and deploy a simple AWS CDK app, from initializing the project to deploying the resulting AWS CloudFormation template\. The app contains one resource, an Amazon S3 bucket\.

## Step 1: Create the Project Directory<a name="hello_world_tutorial_create_directory"></a>

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

**Note**  
Remember that every CDK command has a help option\. For example, cdk init help lists the available languages for the \-\-language \(\-l\) option\.

## Compiling the Project<a name="hello_world_tutorial_compile_project"></a>

Compile your program, as follows\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

------
#### [ C\# ]

Because we configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command\.

```
cdk
```

------

## Listing the Stacks in the App<a name="hello_world_tutorial_list_stacks"></a>

List the stacks in the app\.

```
cdk ls
```

The result is just the name of the stack\.

```
HelloCdkStack
```

## Adding an Amazon S3 Bucket<a name="hello_world_tutorial_add_bucket"></a>

At this point, what can you do with this app? Nothing, because the stack is empty, so there's nothing to deploy\. Let's define an Amazon S3 bucket\.

Install the `@aws-cdk/aws-s3` package\.

------
#### [ TypeScript ]

```
npm install @aws-cdk/aws-s3
```

------
#### [ JavaScript ]

```
npm install @aws-cdk/aws-s3
```

------
#### [ Java ]

Add the following to `pom.xml`, where *CDK\-VERSION* is the version of the CDK\.

```
<dependency>
  <groupId>software.amazon.awscdk</groupId>
  <artifactId>s3</artifactId>
  <version>CDK-VERSION</version>
</dependency>
```

------
#### [ C\# ]

```
dotnet add package Amazon.CDK.AWS.S3
```

------

Next, define an Amazon S3 bucket in the stack\. Amazon S3 buckets are represented by the [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.Bucket) class\.

------
#### [ TypeScript ]

In `lib/hello-cdk-stack.ts`:

```
import cdk = require('@aws-cdk/cdk');
import s3 = require('@aws-cdk/aws-s3');

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
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
    constructor(scope, id, props) {
        super(scope, id, props);

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
    public MyStack(final App scopy, final String id) {
        this(scope, id, null);
    }

    public MyStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);

        new Bucket(this, "MyFirstBucket", BucketProps.builder()
            .withVersioned(true)
            .build());
    }
}
```

------
#### [ C\# ]

Create `MyStack.cs`\.

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class MyStack : Stack
    {
        public MyStack(App scope, string id) : base(scope, id, null)
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

Notice a few things:
+ [Bucket](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.Bucket) is a construct\. This means it's initialization signature has `scope`, `id`, and `props`\. In this case, the bucket is an immediate child of **MyStack**\.
+ `MyFirstBucket` is the **id** of the bucket construct, not the physical name of the Amazon S3 bucket\. The logical ID is used to uniquely identify resources in your stack across deployments\. See [Logical IDs](logical_ids.md) for information about logical IDs\. To specify a physical name for your bucket, set the [bucketName](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.BucketProps.bucketName) property when you define your bucket\.
+ Because the bucket's [versioned](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-s3.html#@aws-cdk/aws-s3.versioned) property is `true`, [versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) is enabled on the bucket\.

Compile your program, as follows\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

------
#### [ C\# ]

Because we configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command\.

```
cdk
```

------

## Synthesizing an AWS CloudFormation Template<a name="hello_world_tutorial_synth_template"></a>

Synthesize an AWS CloudFormation template for the stack, as follows\.

```
cdk synth HelloCdkStack
```

**Note**  
Because the CDK app contains only a single stack, you can omit `HelloCdkStack`\.

This command executes the CDK app and synthesizes an AWS CloudFormation template for the `HelloCdkStack` stack\. You should see something similar to the following, where *VERSION* is the version of the CDK\.

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

You can see that the stack contains an `AWS::S3::Bucket` resource with the versioning configuration we want\.

**Note**  
The toolkit automatically added the **AWS::CDK::Metadata** resource to your template\. The CDK uses metadata to gain insight into how the CDK is used\. One possible benefit is that the CDK team could notify users if a construct is going to be deprecated\. For details, including how to [opt out](tools.md#version_reporting_opt_out) of version reporting, see [Version Reporting](tools.md#version_reporting) \.

## Deploying the Stack<a name="hello_world_tutorial_deploy_stack"></a>

Deploy the stack, as follows\.

```
cdk deploy
```

The deploy command synthesizes an AWS CloudFormation template from the stack, and then invokes the AWS CloudFormation create/update API to deploy it into your AWS account\. If your code includes changes to your existing infrastructure, the command displays information about those changes and requires you to confirm them before it deploys the changes\. The command displays information as it completes various steps in the process\. There is no mechanism to detect or react to any specific step in the process\.

## Modifying the Code<a name="hello_world_tutorial_modify_code"></a>

Configure the bucket to use AWS Key Management Service \(AWS KMS\) managed encryption\.

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
#### [ JavaScript ]

Update `index.js`\.

```
new s3.Bucket(this, 'MyFirstBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KmsManaged
});
```

------
#### [ Java ]

Update `src/main/java/com/acme/MyStack.java`\.

```
new Bucket(this, "MyFirstBucket", BucketProps.builder()
    .withVersioned(true)
    .withEncryption(BucketEncryption.KmsManaged)
    .build());
```

------
#### [ C\# ]

Update `MyStack.cs`\.

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true,
    Encryption = BucketEncryption.KmsManaged
});
```

------

Compile your program, as follows\.

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ JavaScript ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

------
#### [ C\# ]

Because we configured `cdk.json` to run dotnet run, which restores dependencies, builds, and runs your application, run the following command\.

```
cdk
```

------

## Preparing for Deployment<a name="hello_world_tutorial_prep_deployment"></a>

Before you deploy the updated stack, evaluate the difference between the CDK app and the deployed stack\.

```
cdk diff
```

The toolkit queries your AWS account for the current AWS CloudFormation template for the `hello-cdk` stack, and compares the result with the template synthesized from the app\. The output should look like the following\.

```
Resources
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501 
 |- [+] BucketEncryption
     |- {"ServerSideEncryptionConfiguration":[{"ServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}
```

As you can see, the diff indicates that the `ServerSideEncryptionConfiguration` property of the bucket is now set to enable server\-side encryption\.

You can also see that the bucket isn't going to be replaced, but will be updated instead \(**Updating MyFirstBucketB8884501**\)\.

Deploy the changes\.

```
cdk deploy
```

The CDK Toolkit updates the bucket configuration to enable server\-side AWS KMS encryption for the bucket\.

```
HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...
 0/2 | 10:55:30 AM | UPDATE_IN_PROGRESS   | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketB8884501) 
 1/2 | 10:55:50 AM | UPDATE_COMPLETE      | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketB8884501) 

HelloCdkStack

Stack ARN:
arn:aws:cloudformation:us-west-2:188580781645:stack/HelloCdkStack/96956990-feff-11e8-9284-50a68a0e3256
```

## Destroying the Stack<a name="hello_world_tutorial_delete_stack"></a>

Destroy the stack to avoid incurring any costs from the resources created in this tutorial, as follows\.

```
cdk destroy
```

In some cases this command fails, such as when a resource isn't empty and must be empty before it can be destroyed\. See [Delete Stack Fails](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-delete-stack-fails) in the *AWS CloudFormation User Guide* for details\.