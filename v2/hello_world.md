# Your first AWS CDK app<a name="hello_world"></a>

You've read [Getting started with the AWS CDK](getting_started.md) and set up your development environment for writing AWS CDK apps? Great\! Now let's see how it feels to work with the AWS CDK by building the simplest possible AWS CDK app\. 

In this tutorial, you'll learn about the structure of a AWS CDK project, how to use the AWS Construct Library to define AWS resources using code, and how to synthesize, diff, and deploy collections of resources using the AWS CDK Toolkit command\-line tool\.

The standard AWS CDK development workflow is similar to the workflow you're already familiar with as a developer, just with a few extra steps\.

1. Create the app from a template provided by the AWS CDK

1. Add code to the app to create resources within stacks

1. Build the app \(optional; the AWS CDK Toolkit will do it for you if you forget\)

1. Synthesize one or more stacks in the app to create an AWS CloudFormation template

1. Deploy one or more stacks to your AWS account

The build step catches syntax and type errors\. The synthesis step catches logical errors in defining your AWS resources\. The deployment may find permission issues\. As always, you go back to the code, find the problem, fix it, then build, synthesize and deploy again\.

**Tip**  
Don't forget to keep your AWS CDK code under version control\!

This tutorial walks you through creating and deploying a simple AWS CDK app, from initializing the project to deploying the resulting AWS CloudFormation template\. The app contains one stack, which contains one resource: an Amazon S3 bucket\. 

We'll also show what happens when you make a change and re\-deploy, and how to clean up when you're done\.

## Create the app<a name="hello_world_tutorial_create_app"></a>

Each AWS CDK app should be in its own directory, with its own local module dependencies\. Create a new directory for your app\. Starting in your home directory, or another directory if you prefer, issue the following commands\.

**Important**  
Be sure to name your project directory `hello-cdk`, *exactly as shown here\.* The AWS CDK project template uses the directory name to name things in the generated code, so if you use a different name, the code in this tutorial won't work\.

```
mkdir hello-cdk
cd hello-cdk
```

Now initialize the app using the cdk init command, specifying the desired template \("app"\) and programming language\. That is:

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

After the app has been created, also enter the following two commands to activate the app's Python virtual environment and install the AWS CDK core dependencies\.

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

After the app has been created, also enter the following command to instll the AWS Construct Library modules required by the app\.

```
go get
```

------

**Tip**  
If you don't specify a template, the default is "app," which is the one we wanted anyway, so technically you can leave it out and save four keystrokes\.

The cdk init command creates a number of files and folders inside the `hello-cdk` directory to help you organize the source code for your AWS CDK app\. Take a moment to explore\. The structure of a basic app is all there; you'll fill in the details in this tutorial\.

If you have Git installed, each project you create using cdk init is also initialized as a Git repository\. We'll ignore that for now, but it's there when you need it\.

## Build the app<a name="hello_world_tutorial_build"></a>

In most programming environments, after making changes to your code, you'd build \(compile\) it\. This isn't strictly necessary with the AWS CDK—the Toolkit does it for you so you can't forget\. But you can still build manually whenever you want to catch syntax and type errors\. For reference, here's how\.

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

## List the stacks in the app<a name="hello_world_tutorial_list_stacks"></a>

Just to verify everything is working correctly, list the stacks in your app\.

```
cdk ls
```

If you don't see `HelloCdkStack`, make sure you named your app's directory `hello-cdk`\. If you didn't, go back to [Create the app](#hello_world_tutorial_create_app) and try again\.

## Add an Amazon S3 bucket<a name="hello_world_tutorial_add_bucket"></a>

At this point, your app doesn't do anything because the stack it contains doesn't define any resources\. Let's add an Amazon S3 bucket\.

The CDK's Amazon S3 support is part of its main library, `aws-cdk-lib`, so we don't need to install another library\. We can just define an Amazon S3 bucket in the stack using the [Bucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) construct\.

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
	stack := awscdk.NewStack(scope, &id, sprops)

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

`Bucket` is the first construct we've seen, so let's take a closer look\. Like all constructs, the `Bucket` class takes three parameters\.
+ **scope:** Tells the bucket that the stack is its parent: it is defined within the scope of the stack\. You can define constructs inside of constructs, creating a hierarchy \(tree\)\. Here, and in most cases, the scope is `this` \(`self` in Python\), meaning the construct that contains the bucket: the stack\.
+ **Id:** The logical ID of the Bucket within your AWS CDK app\. This \(plus a hash based on the bucket's location within the stack\) uniquely identifies the bucket across deployments so the AWS CDK can update it if you change how it's defined in your app\. Here it is "MyFirstBucket\." Buckets can also have a name, which is separate from this ID \(it's the `bucketName` property\)\.
+ **props:** A bundle of values that define properties of the bucket\. Here we've defined only one property: `versioned`, which enables versioning for the files in the bucket\.

All constructs take these same three arguments, so it's easy to stay oriented as you learn about new ones\. And as you might expect, you can subclass any construct to extend it to suit your needs, or just to change its defaults\.

**Tip**  
If a construct's props are all optional, you can omit the `props` parameter entirely\.

Props are represented differently in the languages supported by the AWS CDK\.
+ In TypeScript and JavaScript, `props` is a single argument and you pass in an object containing the desired properties\.
+ In Python, props are passed as keyword arguments\.
+ In Java, a Builder is provided to pass the props\. Two, actually; one for `BucketProps`, and a second for `Bucket` to let you build the construct and its props object in one step\. This code uses the latter\.
+ In C\#, you instantiate a `BucketProps` object using an object initializer and pass it as the third parameter\.

## Synthesize an AWS CloudFormation template<a name="hello_world_tutorial_synth"></a>

Synthesize an AWS CloudFormation template for the app, as follows\. 

```
cdk synth
```

If your app contained more than one stack, you'd need to specify which stack\(s\) to synthesize\. But since it only contains one, the CDK Toolkit knows you must mean that one\.

**Tip**  
If you received an error like `--app is required...`, it's probably because you are running the command from a subdirectory\. Navigate to the main app directory and try again\.

The `cdk synth` command executes your app, which causes the resources defined in it to be translated into an AWS CloudFormation template\. The displayed output of `cdk synth` is a YAML\-format template; the beginning of our app's output is shown below\. The template is also saved in the `cdk.out` directory in JSON format\.

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

Even if you aren't very familiar with AWS CloudFormation, you should be able to find the definition for the bucket and see how the `versioned` property was translated\. 

**Note**  
Every generated template contains a `AWS::CDK::Metadata` resource by default\. \(We haven't shown it here\.\) The AWS CDK team uses this metadata to gain insight into how the AWS CDK is used, so we can continue to improve it\. For details, including how to opt out of version reporting, see [Version reporting](cli.md#version_reporting)\.

The `cdk synth` generates a perfectly valid AWS CloudFormation template\. You could take it and deploy it using the AWS CloudFormation console or another tool\. But the AWS CDK Toolkit can also do that\.

## Deploying the stack<a name="hello_world_tutorial_deploy"></a>

To deploy the stack using AWS CloudFormation, issue:

```
cdk deploy
```

As with `cdk synth`, you don't need to specify the name of the stack since there's only one in the app\.

It is optional \(though good practice\) to synthesize before deploying\. The AWS CDK synthesizes your stack before each deployment\.

If your code has security implications, you'll see a summary of these and need to confirm them before deployment proceeds\. This isn't the case in our stack\.

`cdk deploy` displays progress information as your stack is deployed\. When it's done, the command prompt reappears\. You can go to the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home) and see that it now lists `HelloCdkStack`\. You'll also find `MyFirstBucket` in the Amazon S3 console\.

You've deployed your first stack using the AWS CDK—congratulations\! But that's not all there is to the AWS CDK\.

## Modifying the app<a name="hello_world_tutorial_modify"></a>

The AWS CDK can update your deployed resources after you modify your app\. Let's change our bucket so it can be automatically deleted when we delete the stack, which involves changing its `RemovalPolicy`\. Also, because AWS CloudFormation won't delete Amazon S3 buckets that contain any objects, we'll ask the AWS CDK to delete the objects from our bucket before destroying the bucket, via the `autoDeleteObjects` property\.

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

Here, we haven't written any code that, in itself, changes our Amazon S3 bucket\. Instead, our code defines the desired state of the bucket\. The AWS CDK synthesizes that state to a new AWS CloudFormation template and deploys a changeset that makes only the changes necessary to reach that state\.

To see these changes, we'll use the `cdk diff` command \.

```
cdk diff
```

The AWS CDK Toolkit queries your AWS account for the last\-deployed AWS CloudFormation template for the `HelloCdkStack` and compares it with the template it just synthesized from your app\. The output should look like the following\.

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

This diff has four sections\.
+ **IAM Statement Changes** and **IAM Policy Changes** \- These permission changes are there because we set the `AutoDeleteObjects` property on our Amazon S3 bucket\. The auto\-delete feature uses a custom resource to delete the objects in the bucket before the bucket itself is deleted\. The IAM objects grant the custom resource's code access to the bucket\.
+ **Parameters** \- The AWS CDK uses these entries to locate the Lambda function asset for the custom resource\.
+ **Resources** \- The new and changed resources in this stack\. We can see the aforementioned IAM objects, the custom resource, and its associated Lambda function being added\. We can also see that the bucket's `DeletionPolicy` and `UpdateReplacePolicy` attributes are being updated\. These allow the bucket to be deleted along with the stack, and to be replaced with a new one\.

You may be curious about why we specified `RemovalPolicy` in our AWS CDK app but got a `DeletionPolicy` property in the resulting AWS CloudFormation template\. The AWS CDK uses a different name for the property because the AWS CDK default is to retain the bucket when the stack is deleted, while AWS CloudFormation's default is to delete it\. See [Removal policies](resources.md#resources_removal) for further details\.

It's informative to compare the output of cdk synth here with the previous output and see the many additional lines of AWS CloudFormation template that the AWS CDK generated for us based on these relatively small changes\.

**Important**  
All AWS CDK v2 deployments use dedicated AWS resources to hold data during deployment, so your AWS account and region must be [bootstrapped](bootstrapping.md) to create these resources before you can deploy\. If you haven't already bootstrapped, issue:  

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

Now let's deploy\.

```
cdk deploy
```

The AWS CDK warns you about the security policy changes we've already seen in the diff\. Enter y to approve the changes and deploy the updated stack\. The CDK Toolkit updates the bucket configuration as you requested\.

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

## Destroying the app's resources<a name="hello_world_tutorial_destroy"></a>

Now that you're done with the quick tour, destroy your app's resources to avoid incurring any costs from the bucket you created, as follows\.

```
cdk destroy
```

Enter y to approve the changes and delete any stack resources\.

**Note**  
If we hadn't changed the bucket's `RemovalPolicy`, the stack deletion would complete successfully, but the bucket would become orphaned \(no longer associated with the stack\)\.

## Next steps<a name="hello_world_next_steps"></a>

Where do you go now that you've dipped your toes in the AWS CDK?
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ Dig deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Abstractions and escape hatches](cfn_layer.md)\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) to begin exploring the CDK constructs available for your favorite AWS services\.
+ Visit [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0) to discover constructs created by AWS and others\. 
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples) of using the AWS CDK\.

The AWS CDK is an open\-source project\. Want to [contribute](https://github.com/aws/aws-cdk)?