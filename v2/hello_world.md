# Your first AWS CDK app<a name="hello_world"></a>

Get started with using the AWS Cloud Development Kit \(AWS CDK\) by building your first CDK app\.

Before starting this tutorial, we recommend that you complete the following:
+ See [What is the AWS CDK?](home.md) for an introduction to the AWS CDK\.
+ See [AWS CDK concepts](core_concepts.md) to learn core concepts of the AWS CDK\.
+ Go through prerequisites and AWS CDK setup steps at [Getting started with the AWS CDK](getting_started.md)\.

**Topics**
+ [About this tutorial](#hello_world_about)
+ [Step 1: Create the app](#hello_world_tutorial_create_app)
+ [Step 2: Build the app](#hello_world_tutorial_build)
+ [Step 3: List the stacks in the app](#hello_world_tutorial_list_stacks)
+ [Step 4: Add an Amazon S3 bucket](#hello_world_tutorial_add_bucket)
+ [Step 5: Synthesize an AWS CloudFormation template](#hello_world_tutorial_synth)
+ [Step 6: Deploy your stack](#hello_world_tutorial_deploy)
+ [Step 7: Modify your app](#hello_world_tutorial_modify)
+ [Step 8: Destroying the app's resources](#hello_world_tutorial_destroy)
+ [Next steps](#hello_world_next_steps)

## About this tutorial<a name="hello_world_about"></a>

In this tutorial, you will create and deploy a simple AWS CDK app\. This app contains one stack with a single Amazon Simple Storage Service \(Amazon S3\) bucket resource\. Through this tutorial, you will learn the following:
+ The structure of an AWS CDK project\.
+ How to create an AWS CDK app\.
+ How to use the AWS Construct Library to define apps, stacks, and AWS resources\.
+ How to use the CDK CLI to synthesize, diff, deploy, and delete your CDK app\.
+ How to modify and re\-deploy your CDK app to update your deployed resources\.

The standard AWS CDK development workflow consists of the following steps:

1. **Create your AWS CDK app** – Here, you will use a template provided by the AWS CDK CLI\.

1. **Define your stacks and resources** – Use constructs to define your stacks and AWS resources within your app\.

1. **Build your app** – This step is optional\. The AWS CDK CLI automatically performs this step if necessary\. Performing this step is recommended to identify syntax and type errors\.

1. **Synthesize your stacks** – This step creates an AWS CloudFormation template for each stack in your app\. This step is useful to identify logical errors in your defined AWS resources\.

1. **Deploy your app** – Deploy to your AWS environment using AWS CloudFormation to provision your resources\. During deployment, you will identify any permission issues with your app\.

Through a typical workflow, you'll go back and repeat previous steps to modify or debug your app\.

We recommend that you use version control for your AWS CDK projects\.

## Step 1: Create the app<a name="hello_world_tutorial_create_app"></a>

A CDK app should be in its own directory, with its own local module dependencies\. On your development machine, create a new directory\. The following is an example that creates a new `hello-cdk` directory:

```
$ mkdir hello-cdk
$ cd hello-cdk
```

**Important**  
Be sure to name your project directory `hello-cdk`, *exactly as shown here\.* The AWS CDK project template uses the directory name to name things in the generated code\. If you use a different name, the code in this tutorial won't work\.

Next, from your new directory, initialize the app by using the cdk init command\. Specify the `app` template and your preferred programming language with the `--language` option\. The following is an example:

------
#### [ TypeScript ]

```
cdk init app --language typescript
```

------
#### [ JavaScript ]

```
cdk init app --language javascript
```

------
#### [ Python ]

```
cdk init app --language python
```

After the app has been created, also enter the following two commands\. These activate the app's Python virtual environment and install the AWS CDK core dependencies\.

```
source .venv/bin/activate
python -m pip install -r requirements.txt
```

------
#### [ Java ]

```
cdk init app --language java
```

If you are using an IDE, you can now open or import the project\. In Eclipse, for example, choose **File** > **Import** > **Maven** > **Existing Maven Projects**\. Make sure that the project settings are set to use Java 8 \(1\.8\)\.

------
#### [ C\# ]

```
cdk init app --language csharp
```

If you are using Visual Studio, open the solution file in the `src` directory\.

------
#### [ Go ]

```
cdk init app --language go
```

After the app has been created, also enter the following command to install the AWS Construct Library modules that the app requires\.

```
go get
```

------

The cdk init command creates a number of files and folders inside the `hello-cdk` directory to help you organize the source code for your AWS CDK app\. Collectively, this is called your AWS CDK *project*\. Take a moment to explore the CDK project\.

If you have Git installed, each project you create using cdk init is also initialized as a Git repository\.

## Step 2: Build the app<a name="hello_world_tutorial_build"></a>

In most programming environments, you build or compile code after making changes\. This isn't necessary with the AWS CDK since the CDK CLI will automatically perform this step\. However, you can still build manually when you want to catch syntax and type errors\. The following is an example: 

------
#### [ TypeScript ]

```
npm run build
```

------
#### [ JavaScript ]

No build step is necessary\.

------
#### [ Python ]

No build step is necessary\.

------
#### [ Java ]

```
mvn compile -q
```

Or press Control\-B in Eclipse \(other Java IDEs may vary\)

------
#### [ C\# ]

```
dotnet build src
```

Or press F6 in Visual Studio

------
#### [ Go ]

```
go build
```

------

## Step 3: List the stacks in the app<a name="hello_world_tutorial_list_stacks"></a>

Verify your app has been correctly created by listing the stacks in your app\. Run the following:

```
cdk ls
```

The output should display `HelloCdkStack`\. If you don't see this output, verify that you are in the correct working directory of your project and try again\. If you still don't see your stack, repeat [Step 1: Create the app](#hello_world_tutorial_create_app) and try again\.

## Step 4: Add an Amazon S3 bucket<a name="hello_world_tutorial_add_bucket"></a>

At this point, your CDK app contains a single stack\. Next, you will define an Amazon Simple Storage Service \(Amazon S3\) bucket resource within your stack\. To do this, you will import and use the `[Bucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html)` L2 construct from the AWS Construct Library\.

Modify your CDK app by importing the `Bucket` construct and defining your Amazon S3 bucket resource\. The following is an example:

------
#### [ TypeScript ]

In `lib/hello-cdk-stack.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { aws_s3 as s3 } from 'aws-cdk-lib';

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

In `lib/hello-cdk-stack.js`:

```
const cdk = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');

class HelloCdkStack extends cdk.Stack {
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

In `hello_cdk/hello_cdk_stack.py`:

```
import aws_cdk as cdk
import aws_cdk.aws_s3 as s3
            
class HelloCdkStack(cdk.Stack):

    def __init__(self, scope: cdk.App, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        bucket = s3.Bucket(self, "MyFirstBucket", versioned=True)
```

------
#### [ Java ]

In `src/main/java/com/myorg/HelloCdkStack.java`:

```
package com.myorg;

import software.amazon.awscdk.*;
import software.amazon.awscdk.services.s3.Bucket;

public class HelloCdkStack extends Stack {
    public HelloCdkStack(final App scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket.Builder.create(this, "MyFirstBucket")
            .versioned(true).build();
    }
}
```

------
#### [ C\# ]

In `src/HelloCdk/HelloCdkStack.cs`:

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class HelloCdkStack : Stack
    {
        public HelloCdkStack(App scope, string id, IStackProps props=null) : base(scope, id, props)
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
#### [ Go ]

In `hello-cdk.go`:

```
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awss3"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type HelloCdkStackProps struct {
	awscdk.StackProps
}

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

func main() {
	defer jsii.Close()

	app := awscdk.NewApp(nil)

	NewHelloCdkStack(app, "HelloCdkStack", &HelloCdkStackProps{
		awscdk.StackProps{
			Env: env(),
		},
	})

	app.Synth(nil)
}

func env() *awscdk.Environment {
	return nil
}
```

------

Let's take a closer look at the `Bucket` construct\. Like all constructs, the `Bucket` class takes three parameters:
+ **scope** – Defines the `Stack` class as the parent of the `Bucket` construct\. All constructs that define AWS resources are created within the scope of a stack\. You can define constructs inside of constructs, creating a hierarchy \(tree\)\. Here, and in most cases, the scope is `this` \(`self` in Python\)\.
+ **Id** – The logical ID of the `Bucket` within your AWS CDK app\. This ID, plus a hash based on the bucket's location within the stack, uniquely identifies the bucket during deployment\. The AWS CDK also references this ID when you update the construct in your app and re\-deploy to update the deployed resource\. Here, your logical ID is `MyFirstBucket`\. Buckets can also have a name, specified with the `bucketName` property\. This is different from the logical ID\.
+ **props** – A bundle of values that define properties of the bucket\. Here you defined the `versioned` property as `true`, which enables versioning for the files in the bucket\.

  Props are represented differently in the languages supported by the AWS CDK\.
  + In TypeScript and JavaScript, `props` is a single argument and you pass in an object containing the desired properties\.
  + In Python, props are passed as keyword arguments\.
  + In Java, a Builder is provided to pass the props\. There are two: one for `BucketProps`, and a second for `Bucket` to let you build the construct and its props object in one step\. This code uses the latter\.
  + In C\#, you instantiate a `BucketProps` object using an object initializer and pass it as the third parameter\.

  If a construct's props are optional, you can omit the `props` parameter entirely\.

All constructs take these same three arguments, so it's easy to stay oriented as you learn about new ones\. And as you might expect, you can subclass any construct to extend it to suit your needs, or if you want to change its defaults\.

## Step 5: Synthesize an AWS CloudFormation template<a name="hello_world_tutorial_synth"></a>

Synthesize an AWS CloudFormation template for the app, as follows:

```
cdk synth
```

If your app contains more than one stack, you must specify which stacks to synthesize\. Since your app contains a single stack, the CDK CLI automatically detects the stack to synthesize\.

If you don't run `cdk synth`, the CDK CLI will automatically perform this step when you deploy\. However, we recommend that you run this step before each deployment\.

**Tip**  
If you receive an error such as `--app is required ...`, check the directory that you are running CDK CLI commands from\. You should be in your main app directory\.

The `cdk synth` command runs your app\. This creates an AWS CloudFormation template for each stack in your app\. The CDK CLI will display a YAML formatted version of your template at the command line and save a JSON formatted version of your template in the `cdk.out` directory\. The following is a snippet of the command line output that shows the bucket being defined in the AWS CloudFormation template:

```
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:...
```

**Note**  
Every generated template contains an `AWS::CDK::Metadata` resource by default\. The AWS CDK team uses this metadata to gain insight into AWS CDK usage and find ways to improve it\. For details, including how to opt out of version reporting, see [Version reporting](cli.md#version_reporting)\.

The generated template can be deployed through the AWS CloudFormation console or any AWS CloudFormation deployment tool\. You can also use the CDK CLI to deploy\. In the next step, you use the CDK CLI to deploy\.

## Step 6: Deploy your stack<a name="hello_world_tutorial_deploy"></a>

To deploy your CDK stack to AWS CloudFormation using the CDK CLI, run the following:

```
cdk deploy
```

**Important**  
You must perform a one\-time bootstrapping of your AWS environment before deployment\. For instructions, see [Bootstrap your environment](getting_started.md#getting_started_bootstrap)\.

Similar to `cdk synth`, you don't have to specify the AWS CDK stack since the app contains a single stack\.

If your code has security implications, the CDK CLI will output a summary\. You will need to confirm them to continue with deployment\. The app in this tutorial doesn't have these implications\.

After running `cdk deploy`, the CDK CLI displays progress information as your stack is deployed\. When complete, you can go to the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home) to view your `HelloCdkStack` stack\. You can also go to the Amazon S3 console to view your `MyFirstBucket` resource\.

Congratulations\! You've deployed your first stack using the AWS CDK\. Next, you will modify your app and re\-deploy to update your resource\.

## Step 7: Modify your app<a name="hello_world_tutorial_modify"></a>

In this step, you will modify your Amazon S3 bucket by configuring it to be automatically deleted when your stack is deleted\. This modification involves changing the bucket's `RemovalPolicy` property\. You will also configure the `autoDeleteObjects` property to configure the CDK CLI to delete objects from the bucket before destroying it\. By default, AWS CloudFormation doesn't delete Amazon S3 buckets that contain objects\.

Use the following example to modify your resource:

------
#### [ TypeScript ]

Update `lib/hello-cdk-stack.ts`\.

```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true
});
```

------
#### [ JavaScript ]

Update `lib/hello-cdk-stack.js`\.

```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true
});
```

------
#### [ Python ]

Update `hello_cdk/hello_cdk_stack.py`\.

```
bucket = s3.Bucket(self, "MyFirstBucket",
    versioned=True,
    removal_policy=cdk.RemovalPolicy.DESTROY,
    auto_delete_objects=True)
```

------
#### [ Java ]

Update `src/main/java/com/myorg/HelloCdkStack.java`\.

```
Bucket.Builder.create(this, "MyFirstBucket")
        .versioned(true)
        .removalPolicy(RemovalPolicy.DESTROY)
        .autoDeleteObjects(true)
        .build();
```

------
#### [ C\# ]

Update `src/HelloCdk/HelloCdkStack.cs`\.

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true,
    RemovalPolicy = RemovalPolicy.DESTROY,
    AutoDeleteObjects = true
});
```

------
#### [ Go ]

Update `hello-cdk.go`\.

```
	  awss3.NewBucket(stack, jsii.String("MyFirstBucket"), &awss3.BucketProps{
		Versioned:         jsii.Bool(true),
		RemovalPolicy:     awscdk.RemovalPolicy_DESTROY,
		AutoDeleteObjects: jsii.Bool(true),
	})
```

------

Currently, your code changes have not made any direct updates to your deployed Amazon S3 bucket resource\. Your code defines the desired state of your resource\. To modify your deployed resource, you will use the CDK CLI to synthesize the desired state into a new AWS CloudFormation template\. Then, you will deploy your new AWS CloudFormation template as a change set\. Change sets make only the necessary changes to reach your new desired state\.

To see these changes, use the `cdk diff` command\. Run the following:

```
cdk diff
```

The CDK CLI queries your AWS account account for the latest AWS CloudFormation template for the `HelloCdkStack` stack\. Then, it compares the latest template with the template it just synthesized from your app\. The output should look like the following\.

```
Stack HelloCdkStack
IAM Statement Changes
┌───┬──────────────────────────────┬────────┬──────────────────────────────┬──────────────────────────────┬───────────┐
│   │ Resource                     │ Effect │ Action                       │ Principal                    │ Condition │
├───┼──────────────────────────────┼────────┼──────────────────────────────┼──────────────────────────────┼───────────┤
│ + │ ${Custom::S3AutoDeleteObject │ Allow  │ sts:AssumeRole               │ Service:lambda.amazonaws.com │           │
│   │ sCustomResourceProvider/Role │        │                              │                              │           │
│   │ .Arn}                        │        │                              │                              │           │
├───┼──────────────────────────────┼────────┼──────────────────────────────┼──────────────────────────────┼───────────┤
│ + │ ${MyFirstBucket.Arn}         │ Allow  │ s3:DeleteObject*             │ AWS:${Custom::S3AutoDeleteOb │           │
│   │ ${MyFirstBucket.Arn}/*       │        │ s3:GetBucket*                │ jectsCustomResourceProvider/ │           │
│   │                              │        │ s3:GetObject*                │ Role.Arn}                    │           │
│   │                              │        │ s3:List*                     │                              │           │
└───┴──────────────────────────────┴────────┴──────────────────────────────┴──────────────────────────────┴───────────┘
IAM Policy Changes
┌───┬────────────────────────────────────────────────────────┬────────────────────────────────────────────────────────┐
│   │ Resource                                               │ Managed Policy ARN                                     │
├───┼────────────────────────────────────────────────────────┼────────────────────────────────────────────────────────┤
│ + │ ${Custom::S3AutoDeleteObjectsCustomResourceProvider/Ro │ {"Fn::Sub":"arn:${AWS::Partition}:iam::aws:policy/serv │
│   │ le}                                                    │ ice-role/AWSLambdaBasicExecutionRole"}                 │
└───┴────────────────────────────────────────────────────────┴────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Parameters
[+] Parameter AssetParameters/4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392/S3Bucket AssetParameters4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392S3BucketBF7A7F3F: {"Type":"String","Description":"S3 bucket for asset \"4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392\""}
[+] Parameter AssetParameters/4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392/S3VersionKey AssetParameters4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392S3VersionKeyFAF93626: {"Type":"String","Description":"S3 key for asset version \"4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392\""}
[+] Parameter AssetParameters/4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392/ArtifactHash AssetParameters4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392ArtifactHashE56CD69A: {"Type":"String","Description":"Artifact hash for asset \"4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392\""}

Resources
[+] AWS::S3::BucketPolicy MyFirstBucket/Policy MyFirstBucketPolicy3243DEFD
[+] Custom::S3AutoDeleteObjects MyFirstBucket/AutoDeleteObjectsCustomResource MyFirstBucketAutoDeleteObjectsCustomResourceC52FCF6E
[+] AWS::IAM::Role Custom::S3AutoDeleteObjectsCustomResourceProvider/Role CustomS3AutoDeleteObjectsCustomResourceProviderRole3B1BD092
[+] AWS::Lambda::Function Custom::S3AutoDeleteObjectsCustomResourceProvider/Handler CustomS3AutoDeleteObjectsCustomResourceProviderHandler9D90184F
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 ├─ [~] DeletionPolicy
 │   ├─ [-] Retain
 │   └─ [+] Delete
 └─ [~] UpdateReplacePolicy
     ├─ [-] Retain
     └─ [+] Delete
```

This diff has four sections:
+ **IAM Statement Changes** and **IAM Policy Changes** – These permission changes are there because you set the `AutoDeleteObjects` property on your Amazon S3 bucket\. The auto\-delete feature uses a custom resource to delete the objects in the bucket before the bucket itself is deleted\. The IAM objects grant the custom resource's code access to the bucket\.
+ **Parameters** – The AWS CDK uses these entries to locate the AWS Lambda function asset for the custom resource\.
+ **Resources** – The new and changed resources in this stack\. We can see the previously mentioned IAM objects, the custom resource, and its associated Lambda function being added\. We can also see that the bucket's `DeletionPolicy` and `UpdateReplacePolicy` attributes are being updated\. These allow the bucket to be deleted along with the stack, and to be replaced with a new one\.

You may notice that we specified `RemovalPolicy` in our AWS CDK app but got a `DeletionPolicy` property in the resulting AWS CloudFormation template\. This is because the AWS CDK uses a different name for the property\. The AWS CDK default is to retain the bucket when the stack is deleted, while the AWS CloudFormation default is to delete it\. For more information, see [Removal policies](resources.md#resources_removal)\.

To see your new AWS CloudFormation template, you can run cdk synth\. By making a few changes to your CDK app, your new AWS CloudFormation template now includes many additional lines of code compared to the original AWS CloudFormation template\.

Next, deploy your app by running the following:

```
cdk deploy
```

The AWS CDK will inform you about the security policy changes we've already seen in the diff\. Enter y to approve the changes and deploy the updated stack\. The CDK CLI will deploy your stack to make your desired changes\. The following is an example output:

```
HelloCdkStack: deploying...
[0%] start: Publishing 4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392:current
[100%] success: Published 4cd61014b71160e8c66fe167e43710d5ba068b80b134e9bd84508cf9238b2392:current
HelloCdkStack: creating CloudFormation changeset...
 0/5 | 4:32:31 PM | UPDATE_IN_PROGRESS   | AWS::CloudFormation::Stack  | HelloCdkStack User Initiated
 0/5 | 4:32:36 PM | CREATE_IN_PROGRESS   | AWS::IAM::Role              | Custom::S3AutoDeleteObjectsCustomResourceProvider/Role (CustomS3AutoDeleteObjectsCustomResourceProviderRole3B1BD092)
 1/5 | 4:32:36 PM | UPDATE_COMPLETE      | AWS::S3::Bucket             | MyFirstBucket (MyFirstBucketB8884501)
 1/5 | 4:32:36 PM | CREATE_IN_PROGRESS   | AWS::IAM::Role              | Custom::S3AutoDeleteObjectsCustomResourceProvider/Role (CustomS3AutoDeleteObjectsCustomResourceProviderRole3B1BD092) Resource creation Initiated
 3/5 | 4:32:54 PM | CREATE_COMPLETE      | AWS::IAM::Role              | Custom::S3AutoDeleteObjectsCustomResourceProvider/Role (CustomS3AutoDeleteObjectsCustomResourceProviderRole3B1BD092)
 3/5 | 4:32:56 PM | CREATE_IN_PROGRESS   | AWS::Lambda::Function       | Custom::S3AutoDeleteObjectsCustomResourceProvider/Handler (CustomS3AutoDeleteObjectsCustomResourceProviderHandler9D90184F)
 3/5 | 4:32:56 PM | CREATE_IN_PROGRESS   | AWS::S3::BucketPolicy       | MyFirstBucket/Policy (MyFirstBucketPolicy3243DEFD)
 3/5 | 4:32:56 PM | CREATE_IN_PROGRESS   | AWS::Lambda::Function       | Custom::S3AutoDeleteObjectsCustomResourceProvider/Handler (CustomS3AutoDeleteObjectsCustomResourceProviderHandler9D90184F) Resource creation Initiated
 3/5 | 4:32:57 PM | CREATE_COMPLETE      | AWS::Lambda::Function       | Custom::S3AutoDeleteObjectsCustomResourceProvider/Handler (CustomS3AutoDeleteObjectsCustomResourceProviderHandler9D90184F)
 3/5 | 4:32:57 PM | CREATE_IN_PROGRESS   | AWS::S3::BucketPolicy       | MyFirstBucket/Policy (MyFirstBucketPolicy3243DEFD) Resource creation Initiated
 4/5 | 4:32:57 PM | CREATE_COMPLETE      | AWS::S3::BucketPolicy       | MyFirstBucket/Policy (MyFirstBucketPolicy3243DEFD)
 4/5 | 4:32:59 PM | CREATE_IN_PROGRESS   | Custom::S3AutoDeleteObjects | MyFirstBucket/AutoDeleteObjectsCustomResource/Default (MyFirstBucketAutoDeleteObjectsCustomResourceC52FCF6E)
 5/5 | 4:33:06 PM | CREATE_IN_PROGRESS   | Custom::S3AutoDeleteObjects | MyFirstBucket/AutoDeleteObjectsCustomResource/Default (MyFirstBucketAutoDeleteObjectsCustomResourceC52FCF6E) Resource creation Initiated
 5/5 | 4:33:06 PM | CREATE_COMPLETE      | Custom::S3AutoDeleteObjects | MyFirstBucket/AutoDeleteObjectsCustomResource/Default (MyFirstBucketAutoDeleteObjectsCustomResourceC52FCF6E)
 5/5 | 4:33:08 PM | UPDATE_COMPLETE_CLEA | AWS::CloudFormation::Stack  | HelloCdkStack
 6/5 | 4:33:09 PM | UPDATE_COMPLETE      | AWS::CloudFormation::Stack  | HelloCdkStack

 ✅  HelloCdkStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT:stack/HelloCdkStack/UNIQUE-ID
```

## Step 8: Destroying the app's resources<a name="hello_world_tutorial_destroy"></a>

Now that you've completed this tutorial, you can delete the deployed AWS CloudFormation stack and all resources associated with it\. This is a good practice to minimize unnecessary costs and keep your environment clean\. Run the following:

```
cdk destroy
```

Enter y to approve the changes and delete your stack\.

**Note**  
If you didn't change the bucket's `RemovalPolicy`, the stack deletion would complete successfully, but the bucket would become orphaned \(no longer associated with the stack\)\.

## Next steps<a name="hello_world_next_steps"></a>

Congratulations\! You've completed this tutorial and have used the AWS CDK to successfully create, modify, and delete resources in the AWS Cloud\. You're now ready to begin using the AWS CDK\.

To learn more about using the AWS CDK in your preferred programming language, see [Working with the AWS CDK in supported programming languages](work-with.md)\.

For additional resources, see the following:
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ Dive deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Customizing constructs from the AWS Construct Library](cfn_layer.md)\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) to begin exploring the CDK constructs available for your favorite AWS services\.
+ Visit [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0) to discover constructs created by AWS and others\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples) of using the AWS CDK\.

The AWS CDK is an open\-source project\. To contribute, see to [Contributing to the AWS Cloud Development Kit \(AWS CDK\)](https://github.com/aws/aws-cdk/blob/main/CONTRIBUTING.md)\.