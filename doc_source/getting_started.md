# Getting Started With the AWS CDK<a name="getting_started"></a>

This topic describes how to install and configure the AWS CDK and create your first AWS CDK app\.

**Note**  
Want to dig deeper? Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour of a real\-world project\.

**Tip**  
The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open source plug\-in for Visual Studio Code that makes it easier to create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications, including the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. [Install the AWS Toolkit](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html) and learn more about [using the AWS CDK Explorer](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Prerequisites<a name="getting_started_prerequisites"></a>

All CDK developers need to install [Node\.js](https://nodejs.org/en/download) >= 10\.3\.0, even those working in languages other than TypeScript or JavaScript\. The AWS CDK Toolkit \(`cdk` command\-line tool\) and the AWS Construct Library are developed in TypeScript and run on Node\.js\. The bindings for other supported languages use this backend and toolset\.

You must provide your credentials and an AWS Region to use the AWS CDK CLI, as described in [Specifying Your Credentials and Region](#getting_started_credentials)\.

Other prerequisites depend on your development language, as follows\.

------
#### [ TypeScript ]

TypeScript >= 2\.7

------
#### [ JavaScript ]

No additional prerequisites

------
#### [ Python ]
+ Python >= 3\.6

------
#### [ Java ]
+ Maven <= 3\.5 and Java >= 8
+ A Java IDE is preferred \(the examples in this guide may refer to Eclipse\)\. `cdk init` creates a Maven project, which most IDEs can import\. Some IDEs may need to be configured to use Java 8 \(also known as 1\.8\)\.
+ Set the `JAVA_HOME` environment variable to the path to where you have installed the JDK on your machine

------
#### [ C\# ]

\.NET standard 2\.1 compatible implementation:
+ \.NET Core >= 3\.1
+ \.NET Framework >= 4\.6\.1, or
+ Mono >= 5\.4

Visual Studio 2019 \(any edition\) recommended\.

------

## Installing the AWS CDK<a name="getting_started_install"></a>

Install the AWS CDK using the following command\.

```
npm install -g aws-cdk
```

Run the following command to verify correct installation and print the version number of the AWS CDK\.

```
cdk --version
```

## Updating Your Language Dependencies<a name="getting_started_update"></a>

If you get an error message that your language framework is out of date, use one of the following commands to update the components that the AWS CDK needs to support the language\.

------
#### [ TypeScript ]

```
npx npm-check-updates -u
```

------
#### [ JavaScript ]

```
npx npm-check-updates -u
```

------
#### [ Python ]

```
pip install --upgrade aws-cdk.core
```

You might have to issue this command multiple times to update all dependencies\.

------
#### [ Java ]

```
mvn versions:use-latest-versions
```

------
#### [ C\# ]

```
nuget update
```

Or **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio

------

## Using the env Property to Specify Account and Region<a name="getting_started_credentials_prop"></a>

You can use the `env` property on a stack to specify the account and region used when deploying a stack, as shown in the following example, where *REGION* is the region and *ACCOUNT* is the account ID\.

------
#### [ TypeScript ]

```
new MyStack(app, 'MyStack', {
    env: {
        region: 'REGION',
        account: 'ACCOUNT' 
    } 
});
```

------
#### [ JavaScript ]

```
new MyStack(app, 'MyStack', {
    env: {
        region: 'REGION',
        account: 'ACCOUNT' 
    } 
});
```

------
#### [ Python ]

```
MyStack(app, "MyStack", env=core.Environment(region="REGION",account="ACCOUNT")
```

------
#### [ Java ]

```
new MyStack(app, "MyStack", StackProps.builder().env(
        Environment.builder()
            .account("ACCOUNT")
            .region("REGION")
            .build()).build());
```

------
#### [ C\# ]

```
new MyStack(app, "MyStack", new StackProps
{
    Env = new Amazon.CDK.Environment
    {
        Account = "ACCOUNT",
        Region = "REGION"
    }
});
```

------

**Note**  
The AWS CDK team recommends that you explicitly set your account and region using the `env` property on a stack when you deploy stacks to production\.

Since you can create any number of stacks in any of your accounts in any region that supports all of the stack's resources, the AWS CDK team recommends that you create your production stacks in one AWS CDK app, and deploy them as necessary\. For example, if you own three accounts, with account IDs *ONE*, *TWO*, and *THREE* and want to be able to deploy each one in **us\-west\-2** and **us\-east\-1**, you might declare them as: 

------
#### [ TypeScript ]

```
new MyStack(app, 'Stack-One-W',   { env: { account: 'ONE',   region: 'us-west-2' }});
new MyStack(app, 'Stack-One-E',   { env: { account: 'ONE',   region: 'us-east-1' }});
new MyStack(app, 'Stack-Two-W',   { env: { account: 'TWO',   region: 'us-west-2' }});
new MyStack(app, 'Stack-Two-E',   { env: { account: 'TWO',   region: 'us-east-1' }});
new MyStack(app, 'Stack-Three-W', { env: { account: 'THREE', region: 'us-west-2' }});
new MyStack(app, 'Stack-Three-E', { env: { account: 'THREE', region: 'us-east-1' }});
```

------
#### [ JavaScript ]

```
new MyStack(app, 'Stack-One-W',   { env: { account: 'ONE',   region: 'us-west-2' }});
new MyStack(app, 'Stack-One-E',   { env: { account: 'ONE',   region: 'us-east-1' }});
new MyStack(app, 'Stack-Two-W',   { env: { account: 'TWO',   region: 'us-west-2' }});
new MyStack(app, 'Stack-Two-E',   { env: { account: 'TWO',   region: 'us-east-1' }});
new MyStack(app, 'Stack-Three-W', { env: { account: 'THREE', region: 'us-west-2' }});
new MyStack(app, 'Stack-Three-E', { env: { account: 'THREE', region: 'us-east-1' }});
```

------
#### [ Python ]

```
MyStack(app, "Stack-One-W",   env=core.Environment(account="ONE",   region="us-west-2"))
MyStack(app, "Stack-One-E",   env=core.Environment(account="ONE",   region="us-east-1"))
MyStack(app, "Stack-Two-W",   env=core.Environment(account="TWO",   region="us-west-2"))
MyStack(app, "Stack-Two-E",   env=core.Environment(account="TWO",   region="us-east-1"))
MyStack(app, "Stack-Three-W", env=core.Environment(account="THREE", region="us-west-2"))
MyStack(app, "Stack-Three-E", env=core.Environment(account="THREE", region="us-east-1"))
```

------
#### [ Java ]

```
public class MyApp {
	
    private static App app;

    // Helper method to declare MyStacks in specific accounts/regions
    private static MyStack makeMyStack(final String name, final String account,
            final String region) {

        return new MyStack(app, name, StackProps.builder()
                .env(Environment.builder()
                    .account(account)
                    .region(region)
                    .build())
                .build());
    }

    public static void main(final String argv[]) {
        app = new App();

        makeMyStack("Stack-One-W",   "ONE",   "us-west-2");
        makeMyStack("Stack-One-E",   "ONE",   "us-east-1");
        makeMyStack("Stack-Two-W",   "TWO",   "us-west-2");
        makeMyStack("Stack-Two-E",   "TWO",   "us-east-1");
        makeMyStack("Stack-Three-W", "THREE", "us-west-2");
        makeMyStack("Stack-Three-E", "THREE", "us-east-1");

        app.synth();
    }
}
```

------
#### [ C\# ]

```
// Helper func to declare MyStacks in specific accounts/regions
Stack makeMyStack(string name, string account, string region)
{
    return new MyStack(app, name, new StackProps
    {
        Env = new Amazon.CDK.Environment
        {
            Account = account,
            Region = region
        }
    });
}

makeMyStack("Stack-One-W",   account: "ONE",   region: "us-west-2");
makeMyStack("Stack-One-E",   account: "ONE",   region: "us-east-1");
makeMyStack("Stack-Two-W",   account: "TWO",   region: "us-west-2");
makeMyStack("Stack-Two-E",   account: "TWO",   region: "us-east-1");
makeMyStack("Stack-Three-W", account: "THREE", region: "us-west-2");
makeMyStack("Stack-Three-E", account: "THREE", region: "us-east-1");
```

------

And deploy the stack for account *TWO* in **us\-east\-1** with:

```
cdk deploy Stack-Two-E
```

**Note**  
If the existing credentials do not have permission to create resources within the account you specify, the AWS CDK returns an AWS CloudFormation error when you attempt to deploy the stack\.

## Specifying Your Credentials and Region<a name="getting_started_credentials"></a>

You must specify your credentials and an AWS Region to use the AWS CDK CLI\. The CDK looks for credentials and region in the following order:
+ Using the \-\-profile option to cdk commands\.
+ Using environment variables\.
+ Using the default profile as set by the AWS Command Line Interface \(AWS CLI\)\.

### Using the \-\-profile Option to Specify Credentials and Region<a name="getting_started_credentials_profile"></a>

Use the \-\-profile *PROFILE* option to a cdk command to use a specific profile when executing the command\.

For example, if the `~/.aws/config` \(Linux or Mac\) or `%USERPROFILE%\.aws\config` \(Windows\) file contains the following profile:

```
[profile test]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
region=us-west-2
```

You can deploy your app using the **test** profile with the following command\.

```
cdk deploy --profile test
```

**Note**  
The profile must contain the access key, secret access key, and region\.

See [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) in the AWS CLI documentation for details\.

### Using Environment Variables to Specify Credentials and a Region<a name="getting_started_credentials_env_vars"></a>

Use environment variables to specify your credentials and region\.
+ `AWS_ACCESS_KEY_ID` – Specifies your access key\.
+ `AWS_SECRET_ACCESS_KEY` – Specifies your secret access key\.
+ `AWS_DEFAULT_REGION` – Specifies your default Region\.

For example, to set the region to `us-east-2` on Linux or macOS:

```
export AWS_DEFAULT_REGION=us-east-2
```

To do the same on Windows:

```
set AWS_DEFAULT_REGION=us-east-2
```

See [Environment Variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html) in the *AWS Command Line Interface User Guide* for details\.

### Using the AWS CLI to Specify Credentials and a Region<a name="getting_started_credentials_config"></a>

Use the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide) aws configure command to specify your default credentials and a region\.

## Hello World Tutorial<a name="hello_world_tutorial"></a>

The typical workflow for creating a new app is:

1. Create the app directory\.

1. Initialize the app\.

1. Add code to the app\.

1. Compile the app, if necessary\.

1. Deploy the resources defined in the app\.

1. Test the app\.

1. If there are any issues, loop through modify, compile \(if necessary\), deploy, and test again\.

And of course, keep your code under version control\.

This tutorial walks you through how to create and deploy a simple AWS CDK app, from initializing the project to deploying the resulting AWS CloudFormation template\. The app contains one resource, an Amazon S3 bucket\.

### Creating the App Directory<a name="hello_world_tutorial_create_directory"></a>

Create a directory for your app with an empty Git repository\.

```
mkdir hello-cdk
cd hello-cdk
```

**Note**  
 Be sure to use the name `hello-cdk` for your project directory\. The AWS CDK project template uses the directory name to name things in the generated code, so if you use a different name, you'll need to change some of the code in this article\. 

### Initializing the App<a name="tutorial_init_project"></a>

To initialize your new AWS CDK app, you use the `cdk init` command\.

```
cdk init --language LANGUAGE [TEMPLATE]
```

Where:
+ *LANGUAGE* is one of the supported programming languages: **csharp** \(C\#\), **java** \(Java\), **javascript** \(JavaScript\), **python** \(Python\), or **typescript** \(TypeScript\)
+ *TEMPLATE* is an optional template\. If the desired template is *app*, the default, you may omit it\.

The following table describes the templates available with the supported languages\.

|  |  | 
| --- |--- |
| app \(default\) |  Creates an empty AWS CDK app\.  | 
| sample\-app |  Creates an AWS CDK app with a stack containing an Amazon SQS queue and an Amazon SNS topic\.  | 

For Hello World, you can just use the default template\.

------
#### [ TypeScript ]

```
cdk init --language typescript
```

------
#### [ JavaScript ]

```
cdk init ‐-language javascript
```

------
#### [ Python ]

```
cdk init --language python
```

Once the init command finishes, your prompt should show **\(\.env\)**, indicating you are running under virtualenv\. If not, activate the virtual environment by issuing the command below\.

```
source .env/bin/activate
```

Once you've got your virtualenv running, run the following command to install the required dependencies\.

```
pip install -r requirements.txt
```

Change the instantiation of **HelloCdkStack** in `app.py` to the following\.

```
HelloCdkStack(app, "HelloCdkStack")
```

------
#### [ Java ]

```
cdk init --language java
```

------
#### [ C\# ]

```
cdk init --language csharp
```

------

### Compiling the App<a name="hello_world_tutorial_compile_project"></a>

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
#### [ Python ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

or press Control\-B in Eclipse

**Tip**  
You can suppress the **\[INFO\]** messages in the build log by adding the **\-q** option to your `mvn` commands\. \(Don"t forget the one in `cdk.json`\.\)

------
#### [ C\# ]

```
dotnet build src
```

or press F6 in Visual Studio

------

### Listing the Stacks in the App<a name="hello_world_tutorial_list_stacks"></a>

List the stacks in the app\.

```
cdk ls
```

The result is just the name of the stack\.

```
HelloCdkStack
```

### Adding an Amazon S3 Bucket<a name="hello_world_tutorial_add_bucket"></a>

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
#### [ Python ]

```
pip install aws-cdk.aws-s3
```

You might have to execute this command multiple times to resolve dependencies\.

------
#### [ Java ]

If necessary, add the following to `pom.xml`, where *CDK\-VERSION* is the version of the AWS CDK\.

```
<dependency>
  <groupId>software.amazon.awscdk</groupId>
  <artifactId>s3</artifactId>
  <version>CDK-VERSION</version>
</dependency>
```

------
#### [ C\# ]

Run the following command in the `src/HelloCdk` directory\.

```
dotnet add package Amazon.CDK.AWS.S3
```

Or **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio, then locate and install the `Amazon.CDK.AWS.S3` package

------

Next, define an Amazon S3 bucket in the stack\. Amazon S3 buckets are represented by the [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) class\.

------
#### [ TypeScript ]

In `lib/hello-cdk-stack.ts`:

```
import * as core = from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

export class HelloCdkStack extends core.Stack {
  constructor(scope: core.App, id: string, props?: core.StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}
```

------
#### [ JavaScript ]

In `lib/hello-cdk-stack.js`:

```
const core = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

class HelloCdkStack extends core.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket', {
      versioned: true
    });
  }
}

module.exports = { HelloCdkStack }
```

------
#### [ Python ]

Replace the first import statement in `hello_cdk_stack.py` in the `hello_cdk` directory with the following code\.

```
from aws_cdk import (
    aws_s3 as s3,
    core
)
```

Replace the comment with the following code\.

```
bucket = s3.Bucket(self, 
    "MyFirstBucket", 
    versioned=True,)
```

------
#### [ Java ]

In `src/main/java/com/myorg/HelloStack.java`:

```
package com.myorg;

import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.s3.Bucket;

public class HelloStack extends Stack {
    public HelloStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
            .versioned(true).build();
    }
}
```

------
#### [ C\# ]

Update `HelloStack.cs` to include a Amazon S3 bucket with versioning enabled\.

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class HelloStack : Stack
    {
        public HelloStack(Construct scope, string id, IStackProps props) : base(scope, id, props)
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
+ [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) is a construct\. This means its initialization signature has `scope`, `id`, and `props` and it is a child of the stack\.
+ `MyFirstBucket` is the **id** of the bucket construct, not the physical name of the Amazon S3 bucket\. The logical ID is used to uniquely identify resources in your stack across deployments\.  To specify a physical name for your bucket, set the [bucketName](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html#bucketname) property \(`bucket_name` in Python\) when you define your bucket\.
+ Because the bucket's [versioned](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html#versioned) property is `true`, [versioning](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html) is enabled on the bucket\.

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
#### [ Python ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

or press Control\-B in Eclipse

**Tip**  
You can suppress the **\[INFO\]** messages in the build log by adding the **\-q** option to your `mvn` commands\. \(Don"t forget the one in `cdk.json`\.\)

------
#### [ C\# ]

```
dotnet build src
```

or press F6 in Visual Studio

------

### Synthesizing an AWS CloudFormation Template<a name="hello_world_tutorial_synth_template"></a>

Synthesize an AWS CloudFormation template for the app, as follows\. If you get an error like "\-\-app is required\.\.\.", it's because you are running the command from a subdirectory of your project directory\. Navigate to the project directory and try again\.

```
cdk synth
```

This command executes the AWS CDK app and synthesizes an AWS CloudFormation template for the `HelloCdkStack` stack\. You should see something similar to the following, where *VERSION* is the version of the AWS CDK\.

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
The AWS CDK CLI automatically adds the **AWS::CDK::Metadata** resource to your template\. The AWS CDK uses metadata to gain insight into how the AWS CDK is used\. One possible benefit is that the CDK team could notify users if a construct is going to be deprecated\. For details, including how to [opt out](tools.md#version_reporting_opt_out) of version reporting, see [Version Reporting](tools.md#version_reporting) \.

### Deploying the Stack<a name="hello_world_tutorial_deploy_stack"></a>

Deploy the stack, as follows\.

```
cdk deploy
```

The deploy command synthesizes an AWS CloudFormation template from the app's stack, and then invokes AWS CloudFormation to deploy it in your AWS account\. If your code would change your infrastructure's security posture, the command displays information about those changes and requires you to confirm them before your stack is deployed\. The command displays information as it completes various steps in the process\.

### Modifying the App<a name="hello_world_tutorial_modify_code"></a>

Configure the bucket to use AWS Key Management Service \(AWS KMS\) managed encryption\.

------
#### [ TypeScript ]

Update `lib/hello-cdk-stack.ts`

```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true,
  encryption: s3.BucketEncryption.KMS_MANAGED
});
```

------
#### [ JavaScript ]

Update `lib/hello-cdk-stack.js`\.

```
new s3.Bucket(this, 'MyFirstBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KMS_MANAGED
});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, 
    "MyFirstBucket", 
    versioned=True,
    encryption=s3.BucketEncryption.KMS_MANAGED,)
```

------
#### [ Java ]

Update `src/main/java/com/myorg/HelloStack.java`\.

```
import software.amazon.awscdk.services.s3.BucketEncryption;
```

```
Bucket.Builder.create(this, "MyFirstBucket")
        .versioned(true)
        .encryption(BucketEncryption.KMS_MANAGED)
        .build();
```

------
#### [ C\# ]

Update `HelloStack.cs`\.

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true,
    Encryption = BucketEncryption.KMS_MANAGED
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
#### [ Python ]

Nothing to compile\.

------
#### [ Java ]

```
mvn compile
```

or press Control\-B in Eclipse

**Tip**  
You can suppress the **\[INFO\]** messages in the build log by adding the **\-q** option to your `mvn` commands\. \(Don"t forget the one in `cdk.json`\.\)

------
#### [ C\# ]

```
dotnet build src
```

or press F6 in Visual Studio

------

### Preparing for Deployment<a name="hello_world_tutorial_prep_deployment"></a>

Before you deploy the updated app, evaluate the difference between the AWS CDK app and the deployed app\.

```
cdk diff
```

The AWS CDK CLI queries your AWS account for the current AWS CloudFormation template for the `hello-cdk` stack, and compares the result with the template synthesized from the app\. The Resources section of the output should look like the following\.

```
Stack HelloCdkStack
Resources
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 |- [+] BucketEncryption
     |- {"ServerSideEncryptionConfiguration":[{"ServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}
```

As you can see, the diff indicates that the `ServerSideEncryptionConfiguration` property of the bucket is now set to enable server\-side encryption\.

You can also see that the bucket isn't going to be replaced, but will be updated instead \(**Updating MyFirstBucket\.\.\.**\)\.

Deploy the changes\.

```
cdk deploy
```

Enter y to approve the changes and deploy the updated stack\. The AWS CDK CLI updates the bucket configuration to enable server\-side AWS KMS encryption for the bucket\. The final output is the ARN of the stack, where *REGION* is your default region, *ACCOUNT\-ID* is your account ID, and *ID* is a unique identifier for the bucket or stack\.

```
HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...
 0/2 | 10:55:30 AM | UPDATE_IN_PROGRESS   | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketID) 
 1/2 | 10:55:50 AM | UPDATE_COMPLETE      | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketID) 

HelloCdkStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/HelloCdkStack/ID
```

### Destroying the App's Resources<a name="hello_world_tutorial_delete_stack"></a>

Destroy the app's resources to avoid incurring any costs from the resources created in this tutorial, as follows\.

```
cdk destroy
```

Enter y to approve the changes and delete any stack resources\. In some cases this command fails, such as when a resource isn't empty and must be empty before it can be destroyed\. See [Delete Stack Fails](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/troubleshooting.html#troubleshooting-errors-delete-stack-fails) in the *AWS CloudFormation User Guide* for details\.