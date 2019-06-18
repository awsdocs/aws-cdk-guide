--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Apps, Stacks, and Environments and Authentication<a name="apps_and_stacks"></a>

The main artifact of a AWS CDK program known as a *CDK app*\. This is an executable program that you can use to synthesize deployment artifacts that supporting tools, such as the AWS CDK Toolkit, can deploy, as described in [AWS CDK Command Line Interface \(cdk\)](tools.md#cli)\.

Stacks are AWS CDK constructs that you can deploy into an AWS environment\. The combination of AWS Region and account becomes the stack's *environment*\. Most production apps consist of multiple stacks of resources that are deployed as a single transaction using a resource provisioning service such as AWS CloudFormation\. Any resources added directly or indirectly as children of a stack are included in the stack's template when it is synthesized by your AWS CDK program\.

Let's look at apps and stacks from the bottom up\.

## Stacks<a name="stacks"></a>

Define an application stack by extending the [Stack]() class, as shown in the following example\.

```
import cdk = require("@aws-cdk/cdk");
import s3 = require("@aws-cdk/aws-s3");

interface MyStackProps extends cdk.StackProps {
  enc: boolean;
}

export class MyStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyStackProps) {
    super(scope, id, props);

    if (props.enc) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KmsManaged
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket");
    }
  }
}
```

Next we'll show you how to use one or more stacks to create a AWS CDK app\.

## Apps<a name="apps"></a>

An [app]() is a collection of [stack]() objects, as shown in the following example\.

```
import cdk = require("@aws-cdk/cdk");
import { MyStack } from "../lib/MyApp-stack";

const app = new cdk.App();

new MyStack(app, "MyWestCdkStack", {
  env: {
    region: "us-west-2"
  },
  enc: false
});

new MyStack(app, "MyEastCdkStack", {
  env: {
    region: "us-east-1"
  },
  enc: true
});
```

Use the cdk ls command to list the stacks in this executable, as shown in the following example\.

```
MyEastCdkStack
MyWestCdkStack
```

Use the cdk deploy command to deploy an individual stack\.

```
cdk deploy MyWestCdkStack
```

## Environments and Authentication<a name="environments"></a>

When you create a [Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/stack.html) instance, you can supply the target deployment environment for the stack using the `env` property, as described in [Specifying Your Credentials and Region](getting_started.md#getting_started_credentials)\.

We recommend that you use the default environment for development stacks, and explicitly specify accounts and regions using the `env` property for production stacks\.

You can always find the Region within which your stack is deployed by using the `region` property of the stack, as follows\.

```
this.region
```