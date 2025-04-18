# Use CloudFormation parameters to get a CloudFormation value<a name="get-cfn-param"></a>

Use AWS CloudFormation parameters within AWS Cloud Development Kit (AWS CDK) applications to input custom values into your synthesized CloudFormation templates at deployment.

For an introduction, see [Parameters and the AWS CDK](parameters.md).

## Define parameters in your CDK app<a name="parameters-define"></a>

Use the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnParameter.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnParameter.html) class to define a parameter. You'll want to specify at least a type and a description for most parameters, though both are technically optional. The description appears when the user is prompted to enter the parameter's value in the AWS CloudFormation console. For more information on the available types, see [Types](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html#parameters-section-structure-properties-type).

**Note**  
You can define parameters in any scope. However, we recommend defining parameters at the stack level so that their logical ID doesn't change when you refactor your code.

------
#### [ TypeScript ]

```
const uploadBucketName = new CfnParameter(this, "uploadBucketName", {
  type: "String",
  description: "The name of the Amazon S3 bucket where uploaded files will be stored."});
```

------
#### [ JavaScript ]

```
const uploadBucketName = new CfnParameter(this, "uploadBucketName", {
  type: "String",
  description: "The name of the Amazon S3 bucket where uploaded files will be stored."});
```

------
#### [ Python ]

```
upload_bucket_name = CfnParameter(self, "uploadBucketName", type="String",
    description="The name of the Amazon S3 bucket where uploaded files will be stored.")
```

------
#### [ Java ]

```
CfnParameter uploadBucketName = CfnParameter.Builder.create(this, "uploadBucketName")
        .type("String")
        .description("The name of the Amazon S3 bucket where uploaded files will be stored")
        .build();
```

------
#### [ C\$1 ]

```
var uploadBucketName = new CfnParameter(this, "uploadBucketName", new CfnParameterProps
{
    Type = "String",
    Description = "The name of the Amazon S3 bucket where uploaded files will be stored"
});
```

------

## Use parameters<a name="parameters-use"></a>

A `CfnParameter` instance exposes its value to your CDK app via a [token](tokens.md). Like all tokens, the parameter's token is resolved at synthesis time. But it resolves to a reference to the parameter defined in the AWS CloudFormation template (which will be resolved at deploy time), rather than to a concrete value.

You can retrieve the token as an instance of the `Token` class, or in string, string list, or numeric encoding. Your choice depends on the kind of value required by the class or method that you want to use the parameter with.

------
#### [ TypeScript ]


| Property | kind of value | 
| --- |--- |
| value | Token class instance | 
| valueAsList | The token represented as a string list | 
| valueAsNumber | The token represented as a number | 
| valueAsString | The token represented as a string | 

------
#### [ JavaScript ]


| Property | kind of value | 
| --- |--- |
| value | Token class instance | 
| valueAsList | The token represented as a string list | 
| valueAsNumber | The token represented as a number | 
| valueAsString | The token represented as a string | 

------
#### [ Python ]


| Property | kind of value | 
| --- |--- |
| value | Token class instance | 
| value\$1as\$1list | The token represented as a string list | 
| value\$1as\$1number | The token represented as a number | 
| value\$1as\$1string | The token represented as a string | 

------
#### [ Java ]


| Property | kind of value | 
| --- |--- |
| getValue() | Token class instance | 
| getValueAsList() | The token represented as a string list | 
| getValueAsNumber() | The token represented as a number | 
| getValueAsString() | The token represented as a string | 

------
#### [ C\$1 ]


| Property | kind of value | 
| --- |--- |
| Value | Token class instance | 
| ValueAsList | The token represented as a string list | 
| ValueAsNumber | The token represented as a number | 
| ValueAsString | The token represented as a string | 

------

For example, to use a parameter in a `Bucket` definition:

------
#### [ TypeScript ]

```
const bucket = new Bucket(this, "amzn-s3-demo-bucket", 
  { bucketName: uploadBucketName.valueAsString});
```

------
#### [ JavaScript ]

```
const bucket = new Bucket(this, "amzn-s3-demo-bucket", 
  { bucketName: uploadBucketName.valueAsString});
```

------
#### [ Python ]

```
bucket = Bucket(self, "amzn-s3-demo-bucket", 
    bucket_name=upload_bucket_name.value_as_string)
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(this, "amzn-s3-demo-bucket")
        .bucketName(uploadBucketName.getValueAsString())
        .build();
```

------
#### [ C\$1 ]

```
var bucket = new Bucket(this, "amzn-s3-demo-bucket")
{
    BucketName = uploadBucketName.ValueAsString
};
```

------

## Deploy CDK apps containing parameters<a name="parameters-deploy"></a>

When you deploy a generated AWS CloudFormation template through the AWS CloudFormation console, you will be prompted to provide the values for each parameter.

You can also provide parameter values using the CDK CLI `cdk deploy` command, or by specifying parameter values in your CDK project’s stack file.

### Provide parameter values with cdk deploy<a name="parameters-deploy-cli"></a>

When you deploy using the CDK CLI `cdk deploy` command, you can provide parameter values at deployment with the `--parameters` option.

The following is an example of the `cdk deploy` command structure:

```
$ cdk deploy stack-logical-id --parameters stack-name:parameter-name=parameter-value
```

If your CDK app contains a single stack, you don’t have to provide the stack logical ID argument or the `stack-name` value in the `--parameters` option. The CDK CLI will automatically find and provide these values. The following is an example that specifies an `uploadbucket` value for the `uploadBucketName` parameter of the single stack in our CDK app:

```
$ cdk deploy --parameters uploadBucketName=uploadbucket
```

### Provide parameter values with cdk deploy for multi-stack applications<a name="parameters-deploy-cli-multi-stack"></a>

The following is an example CDK application in TypeScript that contains two CDK stacks. Each stack contains an Amazon S3 bucket instance and a parameter to set the Amazon S3 bucket name:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

// Define the CDK app
const app = new cdk.App();

// First stack
export class MyFirstStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Set a default parameter name
    const bucketNameParam = new cdk.CfnParameter(this, 'bucketNameParam', {
      type: 'String',
      default: 'myfirststackdefaultbucketname'
    });

    // Define an S3 bucket
    new s3.Bucket(this, 'MyFirstBucket', {
      bucketName: bucketNameParam.valueAsString
    });
  }
}

// Second stack 
export class MySecondStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Set a default parameter name
    const bucketNameParam = new cdk.CfnParameter(this, 'bucketNameParam', {
      type: 'String',
      default: 'mysecondstackdefaultbucketname'
    });

    // Define an S3 bucket
    new s3.Bucket(this, 'MySecondBucket', {
      bucketName: bucketNameParam.valueAsString
    });
  }
}

// Instantiate the stacks
new MyFirstStack(app, 'MyFirstStack', {
  stackName: 'MyFirstDeployedStack',
});

new MySecondStack(app, 'MySecondStack', {
  stackName: 'MySecondDeployedStack',
});
```

For CDK apps that contain multiple stacks, you can do the following:
+ **Deploy one stack with parameters** – To deploy a single stack from a multi-stack application, provide the stack logical ID as an argument.

  The following is an example that deploys `MySecondStack` with `mynewbucketname` as the parameter value for `bucketNameParam`:

  ```
  $ cdk deploy MySecondStack --parameters bucketNameParam='mynewbucketname'
  ```
+ **Deploy all stacks and specify parameter values for each stack** – Provide the `'*'` wildcard or the `--all` option to deploy all stacks. Provide the `--parameters` option multiple times in a single command to specify parameter values for each stack. The following is an example:

  ```
  $ cdk deploy '*' --parameters MyFirstDeployedStack:bucketNameParam='mynewfirststackbucketname' --parameters MySecondDeployedStack:bucketNameParam='mynewsecondstackbucketname'
  ```
+ **Deploy all stacks and specify parameter values for a single stack** – Provide the `'*'` wildcard or the `--all` option to deploy all stacks. Then, specify the stack to define the parameter for in the `--parameters` option. The following are examples that deploys all stacks in a CDK app and specifies a parameter value for the `MySecondDeployedStack` AWS CloudFormation stack. All other stacks will deploy and use the default parameter value:

  ```
  $ cdk deploy '*' --parameters MySecondDeployedStack:bucketNameParam='mynewbucketname'
  $ cdk deploy --all --parameters MySecondDeployedStack:bucketNameParam='mynewbucketname'
  ```

### Provide parameter values with cdk deploy for applications with nested stacks<a name="parameters-deploy-cli-nested-stack"></a>

The CDK CLI behavior when working with applications containing nested stacks is similar to multi-stack applications. The main difference is, if you want to deploy all nested stacks, use the `'**'` wildcard. The `'*'` wildcard deploys all stacks but will not deploy nested stacks. The `'**'` wildcard deploys all stacks, including nested stacks.

The following is an example that deploys nested stacks while specifying the parameter value for one nested stack:

```
$ cdk deploy '**' --parameters MultiStackCdkApp/SecondStack:bucketNameParam='mysecondstackbucketname'
```

For more information on `cdk deploy` command options, see [cdk deploy](ref-cli-cmd-deploy.md).