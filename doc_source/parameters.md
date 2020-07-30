# Parameters<a name="parameters"></a>

AWS CloudFormation templates can contain [parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)—custom values that are supplied at deployment time and incorporated into the template\. Since the AWS CDK synthesizes AWS CloudFormation templates, it too offers support for deployment\-time parameters\. 

Using the AWS CDK, you can both define parameters, which can then be used in the properties of constructs you create, and you can also deploy stacks containing parameters\. 

When deploying the AWS CloudFormation template using the AWS CDK Toolkit, you provide the parameter values on the command line\. If you deploy the template through the AWS CloudFormation console, you are prompted for the parameter values\.

In general, we recommend against using AWS CloudFormation parameters with the AWS CDK\. Unlike [context values](context.md) or environment variables, the usual way to pass values into your AWS CDK apps without hard\-coding them, parameter values are not available at synthesis time, and thus cannot be easily used in other parts of your AWS CDK app, particularly for control flow\.

**Note**  
To do control flow with parameters, you can use [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnCondition.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnCondition.html) constructs, although this is awkward compared to native `if` statements\.

Using parameters requires you to be mindful of how the code you're writing behaves at deployment time, as well as at synthesis time\. This makes it harder to understand and reason about your AWS CDK application, in many cases for little benefit\.

It is better, again in general, to have your CDK app accept any necessary information from the user and use it directly to declare constructs in your CDK app\. An ideal AWS CDK\-generated AWS CloudFormation template is concrete, with no values remaining to be specified at deployment time\. 

There are, however, use cases to which AWS CloudFormation parameters are uniquely suited\. If you have separate teams defining and deploying infrastructure, for example, you can use parameters to make the generated templates more widely useful\. Additionally, the AWS CDK's support for AWS CloudFormation parameters lets you use the AWS CDK with AWS services that use AWS CloudFormation templates \(such as AWS Service Catalog\), which use parameters to configure the template being deployed\.

## Defining parameters<a name="parameters_define"></a>

Use the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnParameter.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnParameter.html) class to define a parameter\. You'll want to specify at least a type and a description for most parameters, though both are technically optional\. The description appears when the user is prompted to enter the parameter's value in the AWS CloudFormation console\.

**Note**  
We recommend defining parameters at the stack level to ensure that their logical ID does not change when you refactor your code\.

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
#### [ C\# ]

```
var uploadBucketName = new CfnParameter(this, "uploadBucketName", new CfnParameterProps
{
    Type = "String",
    Description = "The name of the Amazon S3 bucket where uploaded files will be stored"
});
```

------

## Using parameters<a name="parameters_use"></a>

A `CfnParameter` instance exposes its value to your AWS CDK app via a [token](tokens.md)\. Like all tokens, the parameter's token is resolved at synthesis time, but it resolves to a reference to the parameter defined in the AWS CloudFormation template, which will be resolved at deploy time, rather than to a concrete value\.

You can retrieve the token as an instance of the `Token` class, or in string, string list, or numeric encoding, depending on the type of value required by the class or method you want to use the parameter with\.

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
| value\_as\_list | The token represented as a string list | 
| value\_as\_number | The token represented as a number | 
| value\_as\_string | The token represented as a string | 

------
#### [ Java ]


| Property | kind of value | 
| --- |--- |
| getValue\(\) | Token class instance | 
| getValueAsList\(\) | The token represented as a string list | 
| getValueAsNumber\(\) | The token represented as a number | 
| getValueAsString\(\) | The token represented as a string | 

------
#### [ C\# ]


| Property | kind of value | 
| --- |--- |
| Value | Token class instance | 
| ValueAsList | The token represented as a string list | 
| ValueAsNumber | The token represented as a number | 
| ValueAsString | The token represented as a string | 

------

For example, to use a parameter in a Bucket definition:

------
#### [ TypeScript ]

```
const bucket = new Bucket(this, "myBucket", 
  { bucketName: uploadBucketName.valueAsString});
```

------
#### [ JavaScript ]

```
const bucket = new Bucket(this, "myBucket", 
  { bucketName: uploadBucketName.valueAsString});
```

------
#### [ Python ]

```
bucket = Bucket(self, "myBucket", 
    bucket_name=upload_bucket_name.value_as_string)
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(this, "myBucket")
        .bucketName(uploadBucketName.getValueAsString())
        .build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "myBucket")
{
    BucketName = uploadBucketName.ValueAsString
};
```

------

## Deploying with parameters<a name="parameters_deploy"></a>

A generated template containing parameters can be deployed in the usual way through the AWS CloudFormation console; you are prompted for the values of each parameter\.

The AWS CDK Toolkit \(`cdk` command\-line tool\) also supports specifying parameters at deployment\. You may provide these on the command line following the `--parameters` flag\. You might deploy a stack that uses the `uploadBucketName` parameter like this\.

```
cdk deploy MyStack --parameters uploadBucketName=UploadBucket
```

To define multiple parameters, use multiple `--parameters` flags\.

```
cdk deploy MyStack --parameters uploadBucketName=UpBucket --parameters downloadBucketName=DownBucket
```

If you are deploying multiple stacks, you can specify a different value of each parameter for each stack by prefixing the name of the parameter with the stack name and a colon\.

```
cdk deploy MyStack YourStack --parameters MyStack:uploadBucketName=UploadBucket --parameters YourStack:uploadBucketName=UpBucket
```

By default, the AWS CDK retains values of parameters from previous deployments and uses them in subsequent deployments if they are not specified explicitly\. Use the `--no-previous-parameters flag` to require all parameters to be specified\.