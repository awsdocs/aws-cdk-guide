# Create an App with Multiple Stacks<a name="stack_how_to_create_multiple_stacks"></a>

Most of the other code examples in the *AWS CDK Developer Guide* involve only a single stack\. However, you can create apps containing any number of stacks\. Each stack results in its own AWS CloudFormation template\. Stacks are the *unit of deployment:* each stack in an app can be synthesized and deployed individually using the `cdk deploy` command\.

This topic shows how to create a simple stack that contains an Amazon S3 bucket\. The stack uses a Boolean property, named `encryptBucket`, to indicate whether the bucket should be encrypted\. If so, the stack enables encryption, using a key managed by AWS Key Management Service \(AWS KMS\)\. The app creates two instances of this stack, one with encryption and one without\.

## Prerequisites<a name="cdk-how-to-create-multipls-stacks-prereqs"></a>

To complete this example, first do the following:
+  Install Node\.js and the AWS CDK command line tools, if you haven't already\. See [Getting Started With the AWS CDK](getting_started.md) for details\.
+  Create a TypeScript AWS CDK project by entering the following commands at the command line\.

  ```
  mkdir multistack && cd multistack
  cdk init --language=typescript
  ```
+ Install the `core` and `s3` AWS Construct Library modules\. We use these modules in our app\.

  ```
  npm install @aws-cdk/core
  npm install @aws-cdk/aws-s3
  ```

## Extend `StackProps`<a name="cdk-how-to-create-multipls-stacks-extend-stackprops"></a>

The `props` argument of the `Stack` constructor fulfills the interface `StackProps`\. Because we want our stack to accept an additional property to tell us whether to encrypt the Amazon S3 bucket, we should create an interface that includes that property\. This allows TypeScript to make sure the property has a Boolean value and enables autocompletion for it in your IDE\.

1. Open `lib/multistack-stack.ts` in your IDE or editor\.

1. Add the new interface definition\. The code in `multistack-stack.ts` should now look like this\. The lines we added are shown in boldface\.

   ```
   import cdk = require('@aws-cdk/core');
   import s3 = require('@aws-cdk/aws-s3');
   
   interface MyStackProps extends cdk.StackProps {
     encryptBucket?: boolean;
   }
   
   export class MultistackStack extends cdk.Stack {
     constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
       super(scope, id, props);
   
       // The code that defines your stack goes here
     }
   }
   ```

We've declared the new property as optional\. If `encryptBucket` is not present, its value is `undefined`, which is false in a Boolean context\. In other words, the bucket will be unencrypted by default\.

## Define the Stack Class<a name="cdk-how-to-create-multipls-stacks-define-stack"></a>

 Now let's define our stack class, using our new property\. Still working in `multistack-stack.ts`, make the code look like the following\. The code that you need to add or change is shown in boldface\. 

```
import cdk = require('@aws-cdk/core');
import s3 = require('@aws-cdk/aws-s3');

interface MultistackProps extends cdk.StackProps {
  encryptBucket?: boolean;
}

export class MultistackStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: MultistackProps) {
    super(scope, id, props);
    
   // Add a Boolean property "encryptBucket" to the StackProps constructor.
    // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
    // Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
    if (props && props.encryptBucket) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KMS_MANAGED, 
        removalPolicy: RemovalPolicy.DESTROY
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket", {
        removalPolicy: RemovalPolicy.DESTROY});
    }
  }
}
```

## Create Two Stack Instances<a name="tack_how_to_create_multiple_stacks-create-stacks"></a>

Open the file `bin/multistack.ts` and add the code to instantiate two stacks\. As before, the lines of code shown in boldface are the ones you need to add\. Delete the existing `new MultistackStack` definition\.

```
#!/usr/bin/env node
import 'source-map-support/register';
import cdk = require('@aws-cdk/core');
import { MultistackStack } from '../lib/multistack-stack';

const app = new cdk.App();

new MultistackStack(app, "MyWestCdkStack", {
    env: {region: "us-west-2"},
    encryptBucket: false
});
  
new MultistackStack(app, "MyEastCdkStack", {
    env: {region: "us-east-1"},
    encryptBucket: true
});
```

 This code uses the new `encryptBucket` property on the `MultistackStack` class to create the following: 
+  One stack with an encrypted Amazon S3 bucket in the `us-east-1` AWS Region\. 
+  One stack with an unencrypted Amazon S3 bucket in the `us-west-1` AWS Region\. 

## Define the Stack Class<a name="cdk-how-to-create-multipls-stacks-define-stack"></a>

Now you can build the app\. The TypeScript compiler converts your code to JavaScript\.

```
npm run build
```

Next, synthesize an AWS CloudFormation template for `MyEastCdkStack`—, the stack in `us-east-1`\. This is the stack with the encrypted S3 bucket\.

```
$ cdk synth MyEastCdkStack
```

The output should look similar to the following AWS CloudFormation template \(there might be slight differences\)\.

```
Resources:
  MyGroovyBucketFD9882AC:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: MyEastCdkStack/MyGroovyBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.10.0,@aws-cdk/aws-events=1.10.0,@aws-cdk/aws-iam=1.10.0,@aws-cdk/aws-kms=1.10.0,@aws-cdk/aws-s3=1.10.0,@aws-cdk/core=1.10.0,@aws-cdk/cx-api=1.10.0,@aws-cdk/region-info=1.10.0,jsii-runtime=node.js/v10.16.2
```

To deploy this stack to your AWS account, issue the following command\. For *PROFILE\_NAME*, substitute the name of an AWS CLI profile that contains appropriate credentials for deploying to the `us-east-1` AWS Region\.

```
cdk deploy MyEastCdkStack --profile=PROFILE_NAME
```

## Clean Up<a name="cdk-how-to-create-multipls-stacks-destroy-stack"></a>

To avoid charges for resources that you deployed, destroy the stack using the following command\.

```
cdk destroy MyEastCdkStack
```

The destroy operation fails if there is anything stored in the bucket\. There shouldn't be anything stored if you've only followed the instructions in this topic\. But if you put something in the bucket yourself, you must delete it using the AWS Management Console or the AWS CLI before destroying the stack\.