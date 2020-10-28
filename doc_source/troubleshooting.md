# Troubleshooting common AWS CDK issues<a name="troubleshooting"></a><a name="troubleshooting_top"></a>

This topic describes how to troubleshoot the following issues with the AWS CDK\.
+ [After updating the AWS CDK, code that used to work fine now results in errors](#troubleshooting_modules)
+ [After updating the AWS CDK, the AWS CDK Toolkit \(CLI\) reports a mismatch with the AWS Construct Library](#troubleshooting_toolkit)
+ [When deploying my AWS CDK stack, I receive a `NoSuchBucket` error](#troubleshooting_nobucket)
+ [When deploying my AWS CDK stack, I receive a `forbidden: null` message](#troubleshooting_forbidden_null)
+ [When synthesizing an AWS CDK stack, I get the message `--app is required either in command-line, in cdk.json or in ~/.cdk.json`](#troubleshooting_app_required)
+ [When deploying an AWS CDK stack, I receive an error because the AWS CloudFormation template contains too many resources](#troubleshooting_resource_count)
+ [I specified three \(or more\) Availability Zones for my EC2 Auto\-Scaling Group or Virtual Private Cloud, but it was only deployed in two](#troubleshooting_availability_zones)
+ [My S3 bucket, DynamoDB table, or other resource is not deleted when I issue `cdk destroy`](#troubleshooting_resource_not_deleted)<a name="troubleshooting_modules"></a>

**After updating the AWS CDK, code that used to work fine now results in errors**  
Errors in code that used to work is typically a symptom of having mismatched versions of AWS Construct Library modules\. Make sure all library modules are the same version and up\-to\-date\.

An error message commonly seen in this situation is `unable to determine cloud assembly output directory. Assets must be defined indirectly within a "Stage" or an "App" scope`\.

The modules that make up the AWS Construct Library are a matched set\. They are released together and are intended to be used together\. Interfaces between modules are considered private; we may change them when necessary to implement new features in the library\.

We also update the libraries that are used by the AWS Construct Library from time to time, and different versions of the library modules may have incompatible dependencies\. Synchronizing the versions of the library modules will also address this issue\.

[JSII](https://github.com/aws/jsii) is an important AWS CDK dependency, especially if you are using the AWS CDK in a language other than TypeScript or JavaScript\. You do not ordinarily have to concern yourself with the JSII versions, since it is a declared dependency of all AWS CDK modules\. If a compatible version is not installed, however, you can see unexpected type\-relatd errors, such as `'undefined' is not a valid TargetType`\. Making sure all AWS CDK modules are the same version will resolve JSII compatibility issues, since they will all depend on the same JSII version\.

Below, you'll find details on managing the versions of your installed AWS Construct Library modules in TypeScript, JavaScript, Python, Java, and C\#\.

------
#### [ TypeScript/JavaScript ]

Install your project's AWS Construct Library modules locally \(the default\)\. Use `npm` to install the modules and keep them up to date\.

To see what needs to be updated:

```
npm outdated
```

To actually update the modules to the latest version:

```
npm update
```

If you are working with a specific older version of the AWS Construct Library, rather than the latest, first uninstall all of your project's `@aws-cdk` modules, then reinstall the specific version you want to use\. For example, to install version 1\.9\.0 of the Amazon S3 module, use:

```
npm uninstall @aws-cdk/aws-s3
npm install @aws-cdk/aws-s3@1.9.0
```

Repeat these commands for each module your project uses\.

You can edit your `package.json` file to lock the AWS Construct Library modules to a specific version, so `npm update` won't update them\. You can also specify a version using `~` or `^` to allow modules to be updated to versions that are API\-compatible with the current version, such as `^1.0.0` to accept any update API\-compatible with version 1\.x\. Use the same version specification for all AWS Construct Library modules within a project, including these special characters\. Otherwise, Node\.js may not resolve all these specifications to the same concrete version, resulting in a mismatch\.

------
#### [ Python ]

Use a virtual environment to manage your project's AWS Construct Library modules\. For your convenience, `cdk init` creates a virtual environment for new Python projects in the project's `.env` directory\.

Add the AWS Construct Library modules your project uses to its `requirements.txt` file\. Use the `=` syntax to specify an exact version, or the `~=` syntax to constrain updates to versions without breaking API changes\. For example, the following specifies the latest version of the listed modules that are API\-compatible with version 1\.x: 

```
aws-cdk.core~=1.0
aws-cdk.aws-s3~=1.0
```

If you wanted to accept only bug\-fix updates to, for example, version 1\.9\.0, you could instead specify `~=1.9.0`\. Use the same version specification for all AWS Construct Library modules within a single project\.

Use `pip` to install and update the modules\.

To see what needs to be updated:

```
pip list --local --outdated
```

To actually update the modules to the latest compatible version:

```
pip install --upgrade -r requirements.txt
```

If your project requires a specific older version of the AWS Construct Library, rather than the latest, first uninstall all of your project's `aws-cdk` modules\. Edit `requirements.txt` to specify the exact versions of the modules you want to use using `=`, then install from `requirements.txt`\.

```
pip install -r requirements.txt
```

------
#### [ Java ]

Add your project's AWS Construct Library modules as dependencies in your project's `pom.xml`\. You may specify an exact version, or use Maven's [range syntax](https://docs.oracle.com/middleware/1212/core/MAVEN/maven_version.htm#MAVEN402) to specify a range of allowable versions\.

For example, to specify an exact version of a dependency:

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>s3</artifactId>
    <version>1.23.0</version>
</dependency>
```

To specify that any 1\.x\.x version is acceptable \(note use of right parenthesis to indicate that the end of the range excludes version 2\.0\.0\):

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>s3</artifactId>
    <version>[1.0.0,2.0.0)</version>
</dependency>
```

Maven automatically downloads and installs the latest versions that allow all requirements to be fulfilled when you build your application\.

If you prefer to pin dependencies to a specific version, you can issue `mvn versions:use-latest-versions` to rewrite the version specifications in `pom.xml` to the latest available versions when you decide to upgrade\.

------
#### [ C\# ]

Use the Visual Studio NuGet GUI \(**Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**\) to install the desired version of your application's AWS Construct Library modules\. 
+ The **Installed** panel shows you what modules are currently installed; you can install any available version of any module from this page\.
+ The **Updates** panel shows you modules for which updates are available, and lets you update some or all of them\.

------

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_toolkit"></a>

**After updating the AWS CDK, the AWS CDK Toolkit \(CLI\) reports a mismatch with the AWS Construct Library**  
The version of the AWS CDK Toolkit \(which provides the `cdk` command\) must be at least equal to the version of the AWS Construct Library\. The Toolkit is intended to be backward compatible within the same major version; the latest 1\.x version of the toolkit can be used with any 1\.x release of the library\. For this reason, we recommend you install this component globally and keep it up\-to\-date\. 

```
npm update -g aws-cdk
```

If, for some reason, you need to work with multiple versions of the AWS CDK Toolkit, you can install a specific version of the toolkit locally in your project folder\.

If you are using a language other than TypeScript or JavaScript, first create a `node_modules` folder in your project directory\. Then, regardless of language, use `npm` to install the AWS CDK Toolkit, omitting the `-g` flag and specifying the desired version\. For example:

```
npm install aws-cdk@1.50.0
```

To run a locally\-installed AWS CDK Toolkit, use the command `npx cdk` rather than just `cdk`\. For example:

```
npx cdk deploy MyStack
```

`npx cdk` runs the local version of the AWS CDK Toolkit if one exists, and falls back to the global version when a project doesn't have a local installation\. You may find it convenient to set up a shell alias or batch file to make sure `cdk` is always invoked this way\. For example, Linux users might add the following statement to their `.bash_profile` file\.

```
alias cdk=npx cdk
```

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_nobucket"></a>

**When deploying my AWS CDK stack, I receive a `NoSuchBucket` error**  
Your AWS environment does not have a staging bucket, which the AWS CDK uses to hold resources during deployment\. Stacks require this bucket if they contain [Assets](assets.md) or synthesize to AWS CloudFormation templates larger than 50 kilobytes\. You can create the staging bucket with the following command:

```
cdk bootstrap
```

To avoid generating unexpected AWS charges, the AWS CDK does not automatically create a staging bucket\. You must bootstrap your environment explicitly\.

By default, the staging bucket is created in the region specified by the default AWS profile \(set by `aws configure`\), using that profile's account\. You can specify a different account and region on the command line as follows\.

```
cdk bootstrap aws://123456789/us-east-1
```

You must bootstrap in every region where you will deploy stacks that require a staging bucket\.

For more information, see [Bootstrapping](bootstrapping.md)

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_forbidden_null"></a>

**When deploying my AWS CDK stack, I receive a `forbidden: null` message**  
You are deploying a stack that requires the use of a staging bucket, but are using an IAM role or account that lacks permission to write to it\. \(The staging bucket is used when deploying stacks that contain assets or that synthesize an AWS CloudFormation template larger than 50K\.\) Use an account or role that has permission to perform the action `s3:*` against the resource `arn:aws:s3:::cdktoolkit-stagingbucket-*`\.

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_app_required"></a>

**When synthesizing an AWS CDK stack, I get the message `--app is required either in command-line, in cdk.json or in ~/.cdk.json`**  
 This message usually means that you aren't in the main directory of your AWS CDK project when you issue `cdk synth`\. The file `cdk.json` in this directory, created by the `cdk init` command, contains the command line needed to run \(and thereby synthesize\) your AWS CDK app\. For a TypeScript app, for example, the default `cdk.json` looks something like this: 

```
{
  "app": "npx ts-node bin/my-cdk-app.ts"
}
```

 We recommend issuing `cdk` commands only in your project's main directory, so the AWS CDK toolkit can find `cdk.json` there and successfully run your app\. 

 If this isn't practical for some reason, the AWS CDK Toolkit looks for the app's command line in two other locations: 
+ in `cdk.json` in your home directory
+ on the `cdk synth` command itself using the `-a` option

For example, you might synthesize a stack from a TypeScript app as follows\.

```
cdk synth --app "npx ts-node my-cdk-app.ts" MyStack
```

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_resource_count"></a>

**When deploying an AWS CDK stack, I receive an error because the AWS CloudFormation template contains too many resources**  
The AWS CDK generates and deploys AWS CloudFormation templates\. AWS CloudFormation has a hard limit of 500 resources per stack\. With the AWS CDK, you can run up against this limit more quickly than you might expect, especially if you haven't already worked with AWS CloudFormation enough to know what resources are being generated by the AWS Construct Library constructs you're using\.

The AWS Construct Library's higher\-level, intent\-based constructs automatically provision any auxiliary resources that are needed for logging, key management, authorization, and other purposes\. For example, granting one resource access to another generates any IAM objects needed for the relevant services to communicate\. 

In our experience, real\-world use of intent\-based constructs results in 1–5 AWS CloudFormation resources per construct, though this can vary\. For serverless applications, 5–8 AWS resources per API endpoint is typical\.

Patterns, which represent a higher level of abstraction, let you define even more AWS resources with even less code\. The AWS CDK code in [Creating an AWS Fargate service using the AWS CDK](ecs_example.md), for example, generates more than fifty AWS CloudFormation resources while defining only three constructs\!

Synthesize regularly and keep an eye on how many resources your stack contains\. You'll quickly get a feel for how many resources will be generated by the constructs you use most frequently\.

**Tip**  
You can count the resources in your synthesized output using the following short script\. \(Since every CDK user has Node\.js installed, it is written in JavaScript\.\)  

```
// rescount.js - count the resources defined in a stack
// invoke with: node rescount.js <path-to-stack-json>
// e.g. node rescount.js cdk.out/MyStack.template.json
  
import * as fs from 'fs';
const path = process.argv[2];
  
if (path) fs.readFile(path, 'utf8', function(err, contents) {
  console.log(err ? `${err}` : 
  `${Object.keys(JSON.parse(contents).Resources).length} resources defined in ${path}`);
}); else console.log("Please specify the path to the stack's output .json file");
```

As your stack's resource count approaches 500, consider re\-architecting to reduce the number of resources your stack contains, for example by combining some Lambda functions, or to break it up into multiple stacks\. The CDK supports [references between stacks](resources.md#resource_stack), so it is straightforward to separate your app's functionality into different stacks in whatever way makes the most sense to you\.

**Note**  
AWS CloudFormation experts often suggest the use of nested stacks as a solution to the resource limit\. The AWS CDK supports this approach via the [`NestedStack`](stacks.md#stack_nesting) construct\.

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_availability_zones"></a>

**I specified three \(or more\) Availability Zones for my EC2 Auto\-Scaling Group or Virtual Private Cloud, but it was only deployed in two**  
To get the number of Availability Zones you requested, specify the account and region in the stack's `env` property\. If you do not specify both, the AWS CDK, by default, synthesizes the stack as environment\-agnostic, such that it can be deployed to any region\. You can then deploy the stack to a specific region using AWS CloudFormation\. Because some regions have only two availability zones, an environment\-agnostic template never uses more than two\. 

**Note**  
At this writing, there is one AWS region that has only one availability zone: ap\-northeast\-3 \(Osaka, Japan\)\. Environment\-agnostic AWS CDK stacks cannot be deployed to this region\.

You can change this behavior by overriding your stack's [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html#availabilityzones](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html#availabilityzones) \(Python: `availability_zones`\) property to explicitly specify the zones you want to use\.

For more information about specifying a stack's account and region at synthesis time, while retaining the flexibility to deploy to any region, see [Environments](environments.md)\.

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_resource_not_deleted"></a>

**My S3 bucket, DynamoDB table, or other resource is not deleted when I issue `cdk destroy`**  
By default, resources that can contain user data have a `removalPolicy` \(Python: `removal_policy`\) property of `RETAIN`, and the resource is not deleted when the stack is destroyed\. Instead, the resource is orphaned from the stack\. You must then delete the resource manually after the stack is destroyed\. Until you do, redeploying the stack fails, because the name of the new resource being created during deployment conflicts with the name of the orphaned resource\.

If you set a resource's removal policy to `DESTROY`, that resource will be deleted when the stack is destroyed\.

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

**Note**  
AWS CloudFormation cannot delete a non\-empty Amazon S3 bucket\. If you set an Amazon S3 bucket's removal policy to `DESTROY`, and it contains data, attempting to destroy the stack will fail because the bucket cannot be deleted\.  
It is possible to handle the destruction of an Amazon S3 bucket using an AWS CloudFormation [custom resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html) that deletes the bucket's contents before attempting to delete the bucket itself\. The third\-party construct [https://github.com/mobileposse/auto-delete-bucket/tree/master/src/lambda](https://github.com/mobileposse/auto-delete-bucket/tree/master/src/lambda), for example, uses such a custom resource\.

\([back to list](#troubleshooting_top)\)