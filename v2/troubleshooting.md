# Troubleshooting common AWS CDK issues<a name="troubleshooting"></a><a name="troubleshooting_top"></a>

This topic describes how to troubleshoot the following issues with the AWS CDK\.
+ [After updating the AWS CDK, the AWS CDK Toolkit \(CLI\) reports a mismatch with the AWS Construct Library](#troubleshooting_toolkit)
+ [When deploying my AWS CDK stack, I receive a `NoSuchBucket` error](#troubleshooting_nobucket)
+ [When deploying my AWS CDK stack, I receive a `forbidden: null` message](#troubleshooting_forbidden_null)
+ [When synthesizing an AWS CDK stack, I get the message `--app is required either in command-line, in cdk.json or in ~/.cdk.json`](#troubleshooting_app_required)
+ [When synthesizing an AWS CDK stack, I receive an error because the AWS CloudFormation template contains too many resources](#troubleshooting_resource_count)
+ [I specified three \(or more\) Availability Zones for my EC2 Auto\-Scaling Group or Virtual Private Cloud, but it was only deployed in two](#troubleshooting_availability_zones)
+ [My S3 bucket, DynamoDB table, or other resource is not deleted when I issue `cdk destroy`](#troubleshooting_resource_not_deleted)<a name="troubleshooting_toolkit"></a>

**After updating the AWS CDK, the AWS CDK Toolkit \(CLI\) reports a mismatch with the AWS Construct Library**  
The version of the AWS CDK Toolkit \(which provides the `cdk` command\) must be at least equal to the version of the main AWS Construct Library module, `aws-cdk-lib`\. The Toolkit is intended to be backward compatible; the latest 2\.x version of the toolkit can be used with any 1\.x or 2\.x release of the library\. For this reason, we recommend you install this component globally and keep it up\-to\-date\. 

```
npm update -g aws-cdk
```

If, for some reason, you need to work with multiple versions of the AWS CDK Toolkit, you can install a specific version of the toolkit locally in your project folder\.

If you are using TypeScript or JavaScript, your project directory already contains a versioned local copy of the CDK Toolkit\.

If you are using another language, use `npm` to install the AWS CDK Toolkit, omitting the `-g` flag and specifying the desired version\. For example:

```
npm install aws-cdk@2.0
```

To run a locally\-installed AWS CDK Toolkit, use the command `npx aws-cdk` rather than just `cdk`\. For example:

```
npx aws-cdk deploy MyStack
```

`npx aws-cdk` runs the local version of the AWS CDK Toolkit if one exists, and falls back to the global version when a project doesn't have a local installation\. You may find it convenient to set up a shell alias to make sure `cdk` is always invoked this way\.

------
#### [ macOS/Linux ]

```
alias cdk="npx aws-cdk"
```

------
#### [ Windows ]

```
doskey cdk=npx aws-cdk $*
```

------

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_nobucket"></a>

**When deploying my AWS CDK stack, I receive a `NoSuchBucket` error**  
Your AWS environment has not been bootstrapped, and so does not have an Amazon S3 bucket to hold resources during deployment\. Stacks require this bucket if they contain [Assets](assets.md) or synthesize to AWS CloudFormation templates larger than 50 kilobytes or if they contain assets such as Lambda function code or container images\. You can create the staging bucket with the following command:

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

To avoid generating unexpected AWS charges, the AWS CDK does not automatically bootstrap any environment\. You must bootstrap each environment into which you will deploy explicitly\.

By default, the bootstrap resources are created in the region\(s\) used by stacks in the current AWS CDK application, or the region specified in your local AWS profile \(set by `aws configure`\), using that profile's account\. You can specify a different account and region on the command line as follows\. \(You must specify the account and region if you are not in an app's directory\.\)

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

For more information, see [Bootstrapping](bootstrapping.md)

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_forbidden_null"></a>

**When deploying my AWS CDK stack, I receive a `forbidden: null` message**  
You are deploying a stack that requires bootstrap resources, but are using an IAM role or account that lacks permission to write to it\. \(The staging bucket is used when deploying stacks that contain assets or that synthesize an AWS CloudFormation template larger than 50K\.\) Use an account or role that has permission to perform the action `s3:*` against the bucket mentioned in the error message\.

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

**When synthesizing an AWS CDK stack, I receive an error because the AWS CloudFormation template contains too many resources**  
The AWS CDK generates and deploys AWS CloudFormation templates\. AWS CloudFormation has a hard limit on the number of resources a stack can contain\. With the AWS CDK, you can run up against this limit more quickly than you might expect\.

**Note**  
The AWS CloudFormation resource limit is 500 at this writing\. See [AWS CloudFormation quotas](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html) for the current resource limit\.

The AWS Construct Library's higher\-level, intent\-based constructs automatically provision any auxiliary resources that are needed for logging, key management, authorization, and other purposes\. For example, granting one resource access to another generates any IAM objects needed for the relevant services to communicate\. 

In our experience, real\-world use of intent\-based constructs results in 1–5 AWS CloudFormation resources per construct, though this can vary\. For serverless applications, 5–8 AWS resources per API endpoint is typical\.

Patterns, which represent a higher level of abstraction, let you define even more AWS resources with even less code\. The AWS CDK code in [Creating an AWS Fargate service using the AWS CDK](ecs_example.md), for example, generates more than fifty AWS CloudFormation resources while defining only three constructs\!

Exceeding the AWS CloudFormation resource limit is an error during AWS CloudFormation synthesis\. The AWS CDK issues a warning if your stack exceeds 80% of the limit\. You can use a different limit by setting the `maxResources` property on your stack, or disable validation by setting `maxResources` to 0\.

**Tip**  
You can get an exact count of the resources in your synthesized output using the following utility script\. \(Since every AWS CDK developer needs Node\.js, the script is written in JavaScript\.\)  

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

As your stack's resource count approaches the limit, consider re\-architecting to reduce the number of resources your stack contains: for example, by combining some Lambda functions, or by breaking your stack into multiple stacks\. The CDK supports [references between stacks](resources.md#resource_stack), so it is straightforward to separate your app's functionality into different stacks in whatever way makes the most sense to you\.

**Note**  
AWS CloudFormation experts often suggest the use of nested stacks as a solution to the resource limit\. The AWS CDK supports this approach via the [`NestedStack`](stacks.md#stack_nesting) construct\.

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_availability_zones"></a>

**I specified three \(or more\) Availability Zones for my EC2 Auto\-Scaling Group or Virtual Private Cloud, but it was only deployed in two**  
To get the number of Availability Zones you requested, specify the account and region in the stack's `env` property\. If you do not specify both, the AWS CDK, by default, synthesizes the stack as environment\-agnostic, such that it can be deployed to any region\. You can then deploy the stack to a specific region using AWS CloudFormation\. Because some regions have only two availability zones, an environment\-agnostic template never uses more than two\. 

**Note**  
In the past, regions have occasionally launched with only one availability zone\. Environment\-agnostic AWS CDK stacks cannot be deployed to such regions\. At this writing, however, all AWS regions have at least two AZs\.

You can change this behavior by overriding your stack's [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#availabilityzones](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#availabilityzones) \(Python: `availability_zones`\) property to explicitly specify the zones you want to use\.

For more information about specifying a stack's account and region at synthesis time, while retaining the flexibility to deploy to any region, see [Environments](environments.md)\.

\([back to list](#troubleshooting_top)\)<a name="troubleshooting_resource_not_deleted"></a>

**My S3 bucket, DynamoDB table, or other resource is not deleted when I issue `cdk destroy`**  
By default, resources that can contain user data have a `removalPolicy` \(Python: `removal_policy`\) property of `RETAIN`, and the resource is not deleted when the stack is destroyed\. Instead, the resource is orphaned from the stack\. You must then delete the resource manually after the stack is destroyed\. Until you do, redeploying the stack fails, because the name of the new resource being created during deployment conflicts with the name of the orphaned resource\.

If you set a resource's removal policy to `DESTROY`, that resource will be deleted when the stack is destroyed\.

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
  
export class CdkTestStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
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
const cdk = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');
  
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
import aws_cdk as cdk
from constructs import Constroct
import aws_cdk.aws_s3 as s3

class CdkTestStack(cdk.stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)
        
        bucket = s3.Bucket(self, "Bucket",
            removal_policy=cdk.RemovalPolicy.DESTROY)
```

------
#### [ Java ]

```
software.amazon.awscdk.*;
import software.amazon.awscdk.services.s3.*;
import software.constructs;

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
AWS CloudFormation cannot delete a non\-empty Amazon S3 bucket\. If you set an Amazon S3 bucket's removal policy to `DESTROY`, and it contains data, attempting to destroy the stack will fail because the bucket cannot be deleted\. You can have the AWS CDK delete the objects in the bucket before attempting to destroy it by setting the bucket's `autoDeleteObjects` prop to `true`\.

\([back to list](#troubleshooting_top)\)