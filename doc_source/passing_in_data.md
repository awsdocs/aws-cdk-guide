--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Passing in External Values to Your AWS CDK App<a name="passing_in_data"></a>

There may be cases where you want to parameterize one or more of your stack resources\. Therefore, you want to be able to pass values into your app from outside your app\. Currently, you can get values into your app from outside your app through any of the following\.
+ Using a context variable
+ Using an environment variable
+ Using an SSM Parameter Store variable
+ Using a Secrets manager value
+ Using a value from another stack
+ Using a AWS CloudFormation parameter
+ Using a resource in an existing AWS CloudFormation template

Each of these techniques is described in the following sections\.

## Getting a Value from a Context Variable<a name="passing_context_var"></a>

You can specify a context variable either as part of a AWS CDK Toolkit command, or in a **context** section of `cdk.json`\.

To create a command\-line context variable, use the **\-\-context** \(**\-c**\) option of a AWS CDK Toolkit command, as shown in the following example\.

```
cdk synth -c bucket_name=mygroovybucket
```

To specify the same context variable and value in *cdk\.json*:

```
{
  "context": {
    "bucket_name": "myotherbucket"
  }
}
```

To get the value of a context variable in your app, use code like the following, which gets the value of the context variable **bucket\_name**\.

```
const bucket_name string = this.getContext("bucket_name");
```

## Getting a Value from an Environment Variable<a name="passing_env_var"></a>

To get the value of an environment variable, use code like the following, which gets the value of the environment variable `MYBUCKET`\.

```
const bucket_name = process.env.MYBUCKET;
```

## Getting a Value from an SSM Store Variable<a name="passing_ssm_value"></a>

There are three ways to get the value of an SSM parameter store variable, depending on whether you want the latest version of a plain string, a particular version of a plain string, or a particular version of a secret string\. It is not possible to retrieve the latest version of a secure string\. To read the latest version of a secret, you have to read the secret from SecretsManager \(see [Getting a Value from AWS Secrets Manager](#passing_secrets_manager)\)\.

The first two are not recommended for values that are supposed to be secrets, such as passwords\.

To retrieve the latest value once \(as a context value, see [Environmental Context](context.md)\), and keep on using the same value until the context value manually refreshed, use a [SSMParameterProvider](@cdk-class-url;#@aws-cdk/cdk.SSMParameterProvider):

```
import cdk = require('@amp;aws-cdk/cdk');

const myvalue = new cdk.SSMParameterProvider(this).getString("my-parameter-name");
```

To read a particular version of an SSM Parameter Store plain string value at CloudFormation deployment time, use [ssm\.ParameterStoreString](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ssm.html/#parameterstorestring):

```
import ssm = require('@amp;aws-cdk/aws-ssm');

const parameterString = new ssm.ParameterStoreString(this, 'MyParameter', {
    parameterName: 'my-parameter-name',
    version: 1,
});

const myvalue = parameterString.value;
```

To read a particular version of an SSM Parameter Store SecureString value at CloudFormation deployment time, use [ssm\.ParameterStoreSecureString](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ssm.html/#parameterstoresecurestring):

```
import ssm = require('@amp;aws-cdk/aws-ssm');

const secureString = new ssm.ParameterStoreSecureString(this, 'MySecretParameter', {
    parameterName: 'my-secret-parameter-name',
    version: 1,
});

const myvalue = secureString.value;
```

## Getting a Value from AWS Secrets Manager<a name="passing_secrets_manager"></a>

To use values from AWS Secrets Manager in your CDK app, create an instance of [SecretsManager](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-secretsmanager.html/_aws-cdk_aws-secretsmanager.html#aws-cdk-aws-secretsmanager)\. It represents a value that is retrieved from Secrets Manager and used at AWS CloudFormation deployment time\.

```
import secretsmanager = require('@amp;aws-cdk/aws-secretsmanager');

const loginSecret = new secretsmanager.SecretString(stack, 'Secret', {
  secretId: 'MyLogin'

  // By default, the latest version is retrieved. It's possible to
  // use a specific version instead.
  // versionStage: 'AWSCURRENT'
});

// Retrieve a value from the secret's JSON
const username = loginSecret.jsonFieldValue('username');
const password = loginSecret.jsonFieldValue('password');

// Retrieve the whole secret's string value
const fullValue = loginSecret.value;
```

## Passing in a Value From Another Stack<a name="passing_stack_value"></a>

You can pass a value from one stack to another stack in the same app by using the `export` method in one stack and the `import` method in the other stack\.

The following example creates a bucket on one stack and passes a reference to that bucket to the other stack through an interface\.

First create a stack with a bucket\. The stack includes a property we use to pass the bucket's properties to the other stack\. Note how we use the `export` method on the bucket to get its properties and save them in the stack property\.

```
class HelloCdkStack extends cdk.Stack {
  // Property that defines the stack you are exporting from
  public readonly myBucketRefProps: s3.BucketRefProps;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
      super(scope, id, props);

      const mybucket = new s3.Bucket(this, "MyFirstBucket");

      // Save bucket's BucketRefProps
      this.myBucketRefProps = mybucket.export();
  }
}
```

Create an interface for the second stack's properties\. We use this interface to pass the bucket properties between the two stacks\.

```
// Interface we'll use to pass the bucket's properties to another stack
interface MyCdkStackProps {
    theBucketRefProps: s3.BucketRefProps;
}
```

Create the second stack that gets a reference to the other bucket from the properties passed in through the constructor\.

```
// The class for the other stack
class MyCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyCdkStackProps) {
    super(scope, name);

    const myOtherBucket = s3.Bucket.import(this, "MyOtherBucket", props.theBucketRefProps);

    // Do something with myOtherBucket
  }
}
```

Finally, connect the dots in your app\.

```
const app = new cdk.App();

const myStack = new HelloCdkStack(app, "HelloCdkStack");

new MyCdkStack(app, "MyCdkStack", {
  theBucketRefProps: myStack.myBucketRefProps
});

app.run();
```

## Using an AWS CloudFormation Parameter<a name="passing_cfn_param"></a>

See [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) for information about using the optional *Parameters* section to customize your AWS CloudFormation templates\.

You can also get a reference to a resource in an existing AWS CloudFormation template, as described in [AWS CDK Concepts](concepts.md)\.

## Using an Existing AWS CloudFormation Template<a name="passing_cfn_template"></a>

The AWS CDK provides a mechanism that you can use to incorporate resources from an existing AWS CloudFormation template into your AWS CDK app\. For example, suppose you have a template, `my-template.json`, with the following resource, where **S3Bucket** is the logical ID of the bucket in your template:

```
{
  "S3Bucket": {
    "Type": "AWS::S3::Bucket",
    "Properties": {
      "prop1": "value1"
    }
  }
}
```

You can include this bucket in your AWS CDK app, as shown in the following example \(note that you cannot use this method in an AWS Construct Library construct\):

```
import cdk = require("@amp;aws-cdk/cdk");
import fs = require("fs");

new cdk.Include(this, "ExistingInfrastructure", {
  template: JSON.parse(fs.readFileSync("my-template.json").toString())
});
```

Then to access an attribute of the resource, such as the bucket's ARN:

```
const bucketArn = new cdk.FnGetAtt("S3Bucket", "Arn");
```