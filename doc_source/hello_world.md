# Your first AWS CDK app<a name="hello_world"></a>

You've read [Getting started with the AWS CDK](getting_started.md)? Great\! Now let's see how it feels to work with the AWS CDK by building the simplest possible AWS CDK app\. In this tutorial you'll learn about the structure of a AWS CDK project, how to work with the AWS Construct Library, and how to use the AWS CDK Toolkit command\-line tool\.

The standard AWS CDK development workflow is similar to the workflow you're already familiar with as a developer, just with a few extra steps to synthesize your stack to an AWS CloudFormation template and deploy it\.

1. Create the app from a template provided by the AWS CDK

1. Add code to the app to create resources within stacks

1. Build the app \(optional; the AWS CDK Toolkit will do it for you if you forget\)

1. Synthesize one or more stacks in the app to create an AWS CloudFormation template

1. Deploy one or more stacks to your AWS account

The build step catches syntax and type errors\. The synthesis step catches logical errors in defining your AWS resources\. The deployment may find permission issues\. As always, you go back to the code, find the problem, fix it, then build, synthesize and deploy again\.

**Note**  
Don't forget to keep your AWS CDK code under version control\!

This tutorial walks you through creating and deploying a simple AWS CDK app, from initializing the project to deploying the resulting AWS CloudFormation template\. The app contains one stack, which contains one resource: an Amazon S3 bucket\. 

We'll also show what happens when you make a change and re\-deploy, and how to clean up when you're done\.

## Create the app<a name="hello_world_tutorial_create_app"></a>

Each AWS CDK app should be in its own directory, with its own local module dependencies\. Create a new directory for your app\. Starting in your home directory, or another directory if you prefer, issue the following commands\.

```
mkdir hello-cdk
cd hello-cdk
```

**Important**  
Be sure to name your project directory `hello-cdk`, *exactly as shown here\.* The AWS CDK project template uses the directory name to name things in the generated code, so if you use a different name, some of the code in this tutorial won't work\.

Now initialize the app using the cdk init command, specifying the desired template \("app"\) and programming language\.

```
cdk init TEMPLATE --language LANGUAGE
```

That is:

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

After the app has been created, also enter the following two commands to activate the app's Python virtual environment and install its dependencies\.

```
source .env/bin/activate
python -m pip install -r requirements.txt
```

------
#### [ Java ]

```
cdk init app --language java
```

If you are using an IDE, you can now open or import the project\. In Eclipse, for example, choose **File** > **Import** > **Maven** > **Existing Maven Projects**\. Make sure that the project settings are set ta use Java 8 \(1\.8\)\.

------
#### [ C\# ]

```
cdk init app --language csharp
```

If you are using Visual Studio, open the solution file in the `src` directory\.

------

**Tip**  
If you don't specify a template, the default is "app," which is the one we wanted anyway, so technically you can leave it out and save four keystrokes\.

The cdk init command creates a number of files and folders inside the `hello-cdk` directory to help you organize the source code for your AWS CDK app\. Take a moment to explore\. The structure of a basic app is all there; you'll fill in the details as you progress in this tutorial\.

If you have Git installed, each project you create using cdk init is also initialized as a Git repository\. We'll ignore that for now, but it's there when you need it\.

## Build the app<a name="hello_world_tutorial_build"></a>

Normally, after making any changes to your code, you'd build \(compile\) it\. This isn't strictly necessary with the AWS CDK—the Toolkit does it for you so you can't forget\. But you can still build manually to catch syntax and type errors\. For reference, here's how\.

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

**Note**  
Or press Control\-B in Eclipse \(other Java IDEs may vary\)

------
#### [ C\# ]

```
dotnet build src
```

**Note**  
Or press F6 in Visual Studio

------

## List the stacks in the app<a name="hello_world_tutorial_list_stacks"></a>

Just to verify everything is working correctly, list the stacks in your app\.

```
cdk ls
```

If you don't see `HelloCdkStack`, make sure you named your app's directory `hello-cdk`\. If you didn't, go back to [Create the app](#hello_world_tutorial_create_app) and try again\.

## Add an Amazon S3 bucket<a name="hello_world_tutorial_add_bucket"></a>

At this point, your app doesn't do anything useful because the stack doesn't define any resources\. Let's define an Amazon S3 bucket\.

Install the Amazon S3 package from the AWS Construct Library\.

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

------
#### [ Java ]

Add the following to the `<dependencies>` container of `pom.xml`\.

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>s3</artifactId>
    <version>${cdk.version}</version>
</dependency>
```

If you are using a Java IDE, it probably has a simpler way to add this dependency to your project\. Resist temptation and edit `pom.xml` by hand\.

------
#### [ C\# ]

Run the following command in the `src/HelloCdk` directory\.

```
dotnet add package Amazon.CDK.AWS.S3
```

Or **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio, then locate and install the `Amazon.CDK.AWS.S3` package

------

Next, define an Amazon S3 bucket in the stack using an L2 construct, the [Bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) class\.

------
#### [ TypeScript ]

In `lib/hello-cdk-stack.ts`:

```
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

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
const cdk = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

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

Replace the first import statement in `hello_cdk_stack.py` in the `hello_cdk` directory with the following code\.

```
from aws_cdk import (
    aws_s3 as s3,
    core as cdk
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

In `src/main/java/com/myorg/HelloCdkStack.java`:

```
package com.myorg;

import software.amazon.awscdk.core.*;
import software.amazon.awscdk.services.s3.Bucket;

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

Update `HelloCdkStack.cs` to look like this\.

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace HelloCdk
{
    public class HelloCdkStack : Stack
    {
        public HelloCdkStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
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

`Bucket` is the first construct we've seen, so let's take a closer look\. Like all constructs, the `Bucket` class takes three parameters\.
+ **scope:** Tells the bucket that the stack is its parent: it is defined within the scope of the stack\. You can define constructs inside of constructs, creating a hierarchy \(tree\)\.
+ **Id:** The logical ID of the Bucket within your AWS CDK app\. This \(plus a hash based on the bucket's location within the stack\) uniquely identifies the bucket across deployments so the AWS CDK can update it if you change how it's defined in your app\. Buckets can also have a name, which is separate from this ID \(it's the `bucketName` property\)\.
+ **props:** A bundle of values that define properties of the bucket\. Here we've defined only one property: `versioned`, which enables versioning for the files in the bucket\.

All constructs take these same three arguments, so it's easy to stay oriented as you learn about new ones\. And as you might expect, you can subclass any construct to extend it to suit your needs, or just to change its defaults\.

**Tip**  
If all a construct's props are optional, you can omit the third parameter entirely\.

It's interesting to take note of how props are represented in the different supported languages\.
+ In TypeScript and JavaScript, `props` is a single argument and you pass in an object containing the desired properties\.
+ In Python, props are represented as keyword arguments\.
+ In Java, a Builder is provided to pass the props\. \(Two, actually; one for `BucketProps`, and a second for `Bucket` to let you build the construct and its props object in one step\. This code uses the latter\.\)
+ In C\#, you instantiate a `BucketProps` object using an object initializer and pass it as the third parameter\.

## Synthesize an AWS CloudFormation template<a name="hello_world_tutorial_synth"></a>

Synthesize an AWS CloudFormation template for the app, as follows\. 

```
cdk synth
```

If your app contained more than one stack, you'd need to specify which stack\(s\) to synthesize\. But since it only contains one, the Toolkit knows you must mean that one\.

**Tip**  
If you received an error like `--app is required...`, it's probably because you are running the command from a subdirectory\. Navigate to the main app directory and try again\.

The `cdk synth` command executes your app, which causes the resources defined in it to be translated to an AWS CloudFormation template\. The output of `cdk synth` is a YAML\-format AWS CloudFormation template, which looks something like this\.

```
Resources:
  MyFirstBucketB8884501:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: HelloCdkStack/MyFirstBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.XX.X,@aws-cdk/aws-events=1.XX.X,@aws-cdk/aws-iam=1.XX.X,@aws-cdk/aws-kms=1.XX.X,@aws-cdk/aws-s3=1.XX.X,@aws-cdk/cdk-assets-schema=1.XX.X,@aws-cdk/cloud-assembly-schema=1.XX.X,@aws-cdk/core=1.XX.X,@aws-cdk/cx-api=1.XX.X,@aws-cdk/region-info=1.XX.X,jsii-runtime=node.js/vXX.XX.X
```

Even if you aren't very familiar with AWS CloudFormation, you should be able to find the definition for an `AWS::S3::Bucket` and see how the versioning configuration was translated\. 

**Note**  
Every generated template contains a `AWS::CDK::Metadata` resource by default\. The AWS CDK team uses this metadata to gain insight into how the AWS CDK is used, so we can continue to improve it\. For details, including how to opt out of version reporting, see [Version reporting](cli.md#version_reporting)\.

The `cdk synth` generates a perfectly valid AWS CloudFormation template\. You could take it and deploy it using the AWS CloudFormation console\. But the AWS CDK Toolkit also has that feature built\-in\.

## Deploying the stack<a name="hello_world_tutorial_deploy"></a>

To deploy the stack using AWS CloudFormation, issue:

```
cdk deploy
```

As with `cdk synth`, you don't need to specify the name of the stack since there's only one in the app\.

It is optional \(though good practice\) to synthesize before deploying\. The AWS CDK synthesizes your stack before each deployment\.

If your code changes have security implications, you'll see a summary of these, and be asked to confirm them before deployment proceeds\.

`cdk deploy` displays progress information as your stack is deployed\. When it's done, the command prompt reappears\. You can go to the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home) and see that it now lists `HelloCdkStack`\. You'll also find `MyFirstBucket` in the Amazon S3 console\.

You've deployed your first stack using the AWS CDK—congratulations\! But that's not all there is to the AWS CDK\.

## Modifying the app<a name="hello_world_tutorial_modify"></a>

The AWS CDK can update your deployed resources after you modify your app\. Let's make a little change to our bucket\. We want to be able to delete the bucket automatically when we delete the stack, so we'll change the `RemovalPolicy`\. 

------
#### [ TypeScript ]

Update `lib/hello-cdk-stack.ts`

```
new s3.Bucket(this, 'MyFirstBucket', {
  versioned: true,
  removalPolicy: cdk.RemovalPolicy.DESTROY
});
```

------
#### [ JavaScript ]

Update `lib/hello-cdk-stack.js`\.

```
new s3.Bucket(this, 'MyFirstBucket', {
    versioned: true,
    removalPolicy: cdk.RemovalPolicy.DESTROY
});
```

------
#### [ Python ]

Update `hello_cdk/hello_cdk_stack.py`

```
bucket = s3.Bucket(self, 
    "MyFirstBucket", 
    versioned=True,
    removal_policy=cdk.RemovalPolicy.DESTROY)
```

------
#### [ Java ]

Update `src/main/java/com/myorg/HelloCdkStack.java`\.

```
import software.amazon.awscdk.services.s3.BucketEncryption;
```

```
Bucket.Builder.create(this, "MyFirstBucket")
        .versioned(true)
        .removalPolicy(RemovalPolicy.DESTROY)
        .build();
```

------
#### [ C\# ]

Update `HelloCdkStack.cs`\.

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
    Versioned = true,
    RemovalPolicy = RemovalPolicy.DESTROY
});
```

------

Now we'll use the `cdk diff` command to see the differences between what's already been deployed, and the code we just changed\.

```
cdk diff
```

The AWS CDK Toolkit queries your AWS account for the current AWS CloudFormation template for the `hello-cdk` stack, and compares it with the template it synthesized from your app\. The Resources section of the output should look like the following\.

```
[~] AWS::S3::Bucket MyFirstBucket MyFirstBucketB8884501
 ├─ [~] DeletionPolicy
 │   ├─ [-] Retain
 │   └─ [+] Delete
 └─ [~] UpdateReplacePolicy
     ├─ [-] Retain
     └─ [+] Delete
```

As you can see, the diff indicates that the `DeletionPolicy` property of the bucket is now set to `Delete`, enabling the bucket to be deleted when its stack is deleted\. The `UpdateReplacePolicy `is also changed\.

Don't be confused by the difference in name\. The AWS CDK calls it `RemovalPolicy` because its meaning is slightly different from AWS CloudFormation's `DeletionPolicy`: the AWS CDK default is to retain the bucket when the stack is deleted, while AWS CloudFormation's default is to delete it\. See [Removal policies](resources.md#resources_removal) for further details\.

You can also see that the bucket isn't going to be replaced, but will be updated instead\.

Now let's deploy\.

```
cdk deploy
```

Enter y to approve the changes and deploy the updated stack\. The Toolkit updates the bucket configuration as you requested\.

```
HelloCdkStack: deploying...
HelloCdkStack: creating CloudFormation changeset...
 1/1 | 8:39:43 AM | UPDATE_COMPLETE      | AWS::S3::Bucket    | MyFirstBucket (MyFirstBucketB8884501)
 1/1 | 8:39:44 AM | UPDATE_COMPLETE_CLEA | AWS::CloudFormation::Stack | HelloCdkStack
 2/1 | 8:39:45 AM | UPDATE_COMPLETE      | AWS::CloudFormation::Stack | HelloCdkStack

 ✅  HelloCdkStack

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT:stack/HelloCdkStack/ID
```

## Destroying the app's resources<a name="hello_world_tutorial_destroy"></a>

Now that you're done with the quick tour, destroy your app's resources to avoid incurring any costs from the bucket you created, as follows\.

```
cdk destroy
```

Enter y to approve the changes and delete any stack resources\.

**Note**  
This wouldn't have worked if we hadn't changed the bucket's `RemovalPolicy` just a minute ago\!

If cdk destroy fails, it probably means you put something in your Amazon S3 bucket\. AWS CloudFormation won't delete buckets with files in them\. Delete the files and try again\.

## Next steps<a name="hello_world_next_steps"></a>

Where do you go now that you've dipped your toes in the AWS CDK?
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) to begin exploring the CDK constructs available for your favorite AWS services\.
+ Dig deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Escape hatches](cfn_layer.md)\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples) of using the AWS CDK\.

The AWS CDK is an open\-source project\. Want to [contribute](https://github.com/aws/aws-cdk)?