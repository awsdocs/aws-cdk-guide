# Migrate existing resources and AWS CloudFormation templates to the AWS CDK<a name="migrate"></a>


|  | 
| --- |
| The CDK Migrate feature is in preview release for AWS CDK and is subject to change\. | 

Use the AWS Cloud Development Kit \(AWS CDK\) Command Line Interface \(AWS CDK CLI\) to migrate deployed AWS resources, deployed AWS CloudFormation stacks, and local AWS CloudFormation templates to AWS CDK\.

**Topics**
+ [How migration works](#migrate-intro)
+ [Benefits of CDK Migrate](#migrate-benefits)
+ [Considerations](#migrate-considerations)
+ [Prerequisites](#migrate-prerequisites)
+ [Get started with CDK Migrate](#migrate-gs)
+ [Migrate from an AWS CloudFormation stack](#migrate-stack)
+ [Migrate from an AWS CloudFormation template](#migrate-template)
+ [Migrate from deployed resources](#migrate-resources)
+ [Manage and deploy your CDK app](#migrate-manage)

## How migration works<a name="migrate-intro"></a>

Use the AWS CDK CLI `cdk migrate` command to migrate from the following sources:
+ Deployed AWS resources\.
+ Deployed AWS CloudFormation stacks\.
+ Local AWS CloudFormation templates\.

**Deployed AWS resources**  
You can migrate deployed AWS resources from a specific environment \(AWS account and AWS Region\) that are not associated with an AWS CloudFormation stack\.  
The AWS CDK CLI utilizes the *IaC generator* service to scan for resources in your AWS environment to gather resource details\. To learn more about IaC generator, see [Generating templates for existing resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC.html) in the *AWS CloudFormation User Guide*\.  
After gathering resource details, the AWS CDK CLI creates a new CDK app that includes a single stack containing your migrated resources\.

**Deployed AWS CloudFormation stacks**  
You can migrate a single AWS CloudFormation stack into a new AWS CDK app\. The AWS CDK CLI will retrieve the AWS CloudFormation template of your stack and create a new CDK app\. The CDK app will consist of a single stack that contains your migrated AWS CloudFormation stack\.

**Local AWS CloudFormation templates**  
You can migrate from a local AWS CloudFormation template\. Local templates may or may not contain deployed resources\. The AWS CDK CLI will create a new CDK app that contains a single stack with your resources\.

After migrating, you can manage, modify, and deploy your CDK app to AWS CloudFormation to provision or update your resources\.

## Benefits of CDK Migrate<a name="migrate-benefits"></a>

Migrating resources into AWS CDK has historically been a manual process that requires time and expertise with AWS CloudFormation and AWS CDK to even begin\. With CDK Migrate, the AWS CDK CLI facilitates a majority of the migration effort for you in a fraction of the time\. CDK Migrate will quickly get you started with using the AWS CDK to develop and manage new and existing applications on AWS\.

## Considerations<a name="migrate-considerations"></a>

### General considerations<a name="migrate-considerations-general"></a>

**CDK Migrate vs\. CDK Import**  
The `cdk import` command can import deployed resources into a new or existing CDK app\. When importing, each resource will have to manually be defined as an L1 construct in your app\. We recommend using `cdk import` to import one or more resources at a time into a new or existing CDK app\. To learn more, see [Importing existing resources into a stack](cli.md#cli-import)\.  
The `cdk migrate` command migrates from deployed resources, deployed AWS CloudFormation stacks, or local AWS CloudFormation templates into a new CDK app\. During migration, the AWS CDK CLI uses `cdk import` to import your resources into the new CDK app\. The AWS CDK CLI also generates L1 constructs for each resource for you\. We recommend using `cdk migrate` when importing from a supported migration source into a new AWS CDK app\.

**CDK Migrate creates L1 constructs only**  
The newly created CDK app will include L1 constructs only\. You can add higher\-level constructs to your app after migration\.

**CDK Migrate creates CDK apps that contain a single stack**  
The newly created CDK app will contain a single stack\.  
When migrating deployed resources, all migrated resources will be contained within a single stack in the new CDK app\.  
When migrating AWS CloudFormation stacks, you can only migrate a single AWS CloudFormation stack into a single stack in the new CDK app\.

**Migrating assets**  
Project assets, such as AWS Lambda code, will not directly migrate into the new CDK app\. After migration, you can specify asset values to include them in the CDK app\.

**Migrating stateful resources**  
When migrating stateful resources, such as databases and Amazon Simple Storage Service \(Amazon S3\) buckets, you’d most often want to migrate the existing resource instead of creating a new resource\.  
To migrate and preserve stateful resources, do the following:  
+ Verify that your stateful resource supports import\. For more information, see [Resource type support](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import-supported-resources.html) in the *AWS CloudFormation User Guide*\.
+ After migration, verify that the migrated resource’s logical ID in the new CDK app matches the logical ID of the deployed resource\.
+ If migrating from an AWS CloudFormation stack, verify that the stack name in the new CDK app matches the AWS CloudFormation stack\.
+ Deploy the CDK app using the same AWS account and AWS Region of the migrated resource\.

### Considerations when migrating from an AWS CloudFormation template<a name="migrate-considerations-template"></a>

**CDK Migrate supports single template migration**  
When migrating AWS CloudFormation templates, you can select a single template to migrate\. Nested templates are not supported\.

**Migrating templates with intrinsic functions**  
When migrating from an AWS CloudFormation template that uses intrinsic functions, the AWS CDK CLI will attempt to migrate your logic into the CDK app with the `Fn` class\. To learn more, see [class Fn](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Fn.html) in the *AWS Cloud Development Kit \(AWS CDK\) API Reference*\.

### Considerations when migrating from deployed resources<a name="migrate-considerations-resources"></a>

**Scan limitations**  
When scanning your environment for resources, IaC generator has specific limitations on the data it can retrieve and quota limitations when scanning\. To learn more, see [Considerations](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC.html#generate-template-considerations) in the *AWS CloudFormation User Guide*\.

## Prerequisites<a name="migrate-prerequisites"></a>

Before using the `cdk migrate` command, complete all set up steps in [Getting started with the AWS CDK](getting_started.md)\.

## Get started with CDK Migrate<a name="migrate-gs"></a>

To begin, run the AWS CDK CLI `cdk migrate` command from a directory of your choice\. Provide required and optional options, depending on the type of migration you are performing\.

For a full list and description of options that you can use with `cdk migrate`, see [cdk migrate](ref-cli-cdk-migrate.md)\.

The following are some important options that you may want to provide\.

**Stack name**  
The only required option is `--stack-name`\. Use this option to specify a name for the stack that will be created within the AWS CDK app after migration\. The stack name will also be used as the name of your AWS CloudFormation stack at deployment\.

**Language**  
Use `--language` to specify the programming language of the new CDK app\.

**AWS account and AWS Region**  
The AWS CDK CLI retrieves AWS account and AWS Region information from default sources\. For more information, see [Environments](environments.md)\. You can use `--account` and `--region` options with `cdk migrate` to provide other values\.

**Output directory of your new CDK project**  
By default, the AWS CDK CLI will create a new CDK project in your working directory and use the value you provide with `--stack-name` to name the project folder\. If a folder with the same name currently exists, the AWS CDK CLI will overwrite that folder\.  
You can specify a different output path for the new CDK project folder with the `--output-path` option\.

**Migration source**  
Provide an option to specify the source you are migrating from\.  
+ `--from-path` – Migrate from a local AWS CloudFormation template\.
+ `--from-scan` – Migrate from deployed resources in an AWS account and AWS Region\.
+ `--from-stack` – Migrate from an AWS CloudFormation stack\.
Depending on your migration source, you can provide additional options to customize the `cdk migrate` command\.

## Migrate from an AWS CloudFormation stack<a name="migrate-stack"></a>

To migrate from a deployed AWS CloudFormation stack, provide the `--from-stack` option\. Provide the name of your deployed AWS CloudFormation stack with `--stack-name`\. The following is an example:

```
$ cdk migrate --from-stack --stack-name "myCloudFormationStack"
```

The AWS CDK CLI will do the following:

1. Retrieve the AWS CloudFormation template of your deployed stack\.

1. Run `cdk init` to initialize a new CDK app\.

1. Create a stack within the CDK app that contains your migrated AWS CloudFormation stack\.

When you migrate from a deployed AWS CloudFormation stack, the AWS CDK CLI attempts to match deployed resource logical IDs and the deployed AWS CloudFormation stack name to the migrated resources and stack in the new CDK app\.

After migration, you can manage and modify your CDK app normally\. When you deploy, AWS CloudFormation will identify the deployment as an AWS CloudFormation stack update due to the matching AWS CloudFormation stack name\. Resources with matching logical IDs will be updated\. For more information on deploying, see [Manage and deploy your CDK app](#migrate-manage)\.

## Migrate from an AWS CloudFormation template<a name="migrate-template"></a>

CDK Migrate supports migrating from AWS CloudFormation templates formatted in `JSON` or `YAML`\.

To migrate from a local AWS CloudFormation template, use the `--from-path` option and provide a path to the local template\. You must also provide the required `--stack-name` option\. The following is an example:

```
$ cdk migrate --from-path "./template.json" --stack-name "myCloudFormationStack"
```

The AWS CDK CLI will do the following:

1. Retrieve your local AWS CloudFormation template\.

1. Run `cdk init` to initialize a new CDK app\.

1. Create a stack within the CDK app that contains your migrated AWS CloudFormation template\.

After migration, you can manage and modify your CDK app normally\. At deployment, you have the following options:
+ **Update an AWS CloudFormation stack** – If the local AWS CloudFormation template was previously deployed, you can update the deployed AWS CloudFormation stack\.
+ **Deploy a new AWS CloudFormation stack** – If the local AWS CloudFormation template was never deployed, or if you want to create a new stack from a previously deployed template, you can deploy a new AWS CloudFormation stack\.

### Migrate from an AWS SAM template<a name="migrate-template-sam"></a>

To migrate from an AWS Serverless Application Model \(AWS SAM\) template, you must first convert it to an AWS CloudFormation template or deploy to create an AWS CloudFormation stack\.

To convert an AWS SAM template to AWS CloudFormation, you can use the AWS SAM CLI `sam validate --debug` command\. You may have to set `lint` to `false` in your `samconfig.toml` file before running this command\.

To convert to an AWS CloudFormation stack, deploy the AWS SAM template using the AWS SAM CLI\. Then migrate from the deployed stack\.

## Migrate from deployed resources<a name="migrate-resources"></a>

To migrate from deployed AWS resources, provide the `--from-scan` option\. You must also provide the required `--stack-name` option\. The following is an example:

```
$ cdk migrate --from-scan --stack-name "myCloudFormationStack"
```

The AWS CDK CLI will do the following:

1. **Scan your account for resource and property details** – The AWS CDK CLI utilizes IaC generator to scan your account and gather details\.

1. **Generate an AWS CloudFormation template** – After scanning, the AWS CDK CLI utilizes IaC generator to create an AWS CloudFormation template\.

1. **Initialize a new CDK app and migrate your template** – The AWS CDK CLI runs `cdk init` to initialize a new AWS CDK app and migrates your AWS CloudFormation template into the CDK app as a single stack\.

### Use filters<a name="migrate-resources-filters"></a>

By default, the AWS CDK CLI will scan the entire AWS environment and migrate resources up to the maximum quota limit of IaC generator\. You can provide filters with the AWS CDK CLI to specify a criteria for which resources get migrated from your account to the new CDK app\. To learn more, see `\-\-filter`\.

### Scanning for resources with IaC generator<a name="migrate-resources-scan"></a>

Depending on the number of resources in your account, scanning may take a few minutes\. A progress bar will display during the scanning process\.

**Supported resource types**  <a name="migrate-resources-supported"></a>
The AWS CDK CLI will migrate resources supported by the IaC generator\. For a full list, see [Resource type support](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import-supported-resources.html) in the *AWS CloudFormation User Guide*\.

### Resolve write\-only properties<a name="migrate-resources-writeonly"></a>

Some supported resources contain write\-only properties\. These properties can be written to, to configure the property, but can’t be read by IaC generator or AWS CloudFormation to obtain the value\. For example, a property used to specify a database password may be write\-only for security reasons\.

When scanning resources during migration, IaC generator will detect resources that may contain write\-only properties and will categorize them into any of the following types:
+ `MUTUALLY_EXCLUSIVE_PROPERTIES` – These are write\-only properties for a specific resource that are interchangeable and serve a similar purpose\. One of the mutually exclusive properties are required to configure your resource\. For example, the `S3Bucket`, `ImageUri`, and `ZipFile` properties for an `AWS::Lambda::Function` resource are mutually exclusive write\-only properties\. Any one of them can be used to specify your function assets, but you must use one\.
+ `MUTUALLY_EXCLUSIVE_TYPES` – These are required write\-only properties that accept multiple configuration types\. For example, the `Body` property of an `AWS::ApiGateway::RestApi` resource accepts an object or string type\.
+ `UNSUPPORTED_PROPERTIES` – These are write\-only properties that don't fall under the other two categories\. They are either optional properties or required properties that accept an array of objects\.

For more information on write\-only properties and how IaC generator manages them when scanning for deployed resources and creating AWS CloudFormation templates, see [ IaC generator and write\-only properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC-write-only-properties.html) in the *AWS CloudFormation User Guide*\.

After migration, you must specify write\-only property values in the new CDK app\. The AWS CDK CLI will append a **Warnings** section to the CDK project’s `ReadMe` file to document any write\-only properties that were identified by IaC generator\. The following is an example:

```
# Welcome to your CDK TypeScript project
...
## Warnings
### Write-only properties
Write-only properties are resource property values that can be written to but can't be read by AWS CloudFormation or CDK Migrate. For more information, see [IaC generator and write-only properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC-write-only-properties.html).

Write-only properties discovered during migration are organized here by resource ID and categorized by write-only property type. Resolve write-only properties by providing property values in your CDK app. For guidance, see [Resolve write-only properties](https://docs.aws.amazon.com/cdk/v2/guide/migrate.html#migrate-resources-writeonly).
### MyLambdaFunction
- **UNSUPPORTED_PROPERTIES**: 
  - SnapStart/ApplyOn: Applying SnapStart setting on function resource type.Possible values: [PublishedVersions, None]
This property can be replaced with other types
  - Code/S3ObjectVersion: For versioned objects, the version of the deployment package object to use.
This property can be replaced with other exclusive properties
- **MUTUALLY_EXCLUSIVE_PROPERTIES**: 
  - Code/S3Bucket: An Amazon S3 bucket in the same AWS Region as your function. The bucket can be in a different AWS account.
This property can be replaced with other exclusive properties
  - Code/S3Key: The Amazon S3 key of the deployment package.
This property can be replaced with other exclusive properties
```
+ Warnings are organized under headings that identify the resource’s logical ID that they are associated with\.
+ Warnings are categorized by type\. These types come directly from IaC generator\.

**To resolve write\-only properties**

1. Identify write\-only properties to resolve from the **Warnings** section of your CDK project’s `ReadMe` file\. Here, you can take note of the resources in your CDK app that may contain write\-only properties and identify the write\-only property types that were discovered\.

   1. For `MUTUALLY_EXCLUSIVE_PROPERTIES`, determine which mutually exclusive property to configure in your AWS CDK app\.

   1. For `MUTUALLY_EXCLUSIVE_TYPES`, determine which accepted type that you will use to configure the property\.

   1. For `UNSUPPORTED_PROPERTIES`, determine if the property is optional or required\. Then, configure as necessary\.

1. Use guidance from [IaC generator and write\-only properties](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/generate-IaC-write-only-properties.html) to reference what the warning types mean\.

1. In your CDK app, write\-only property values to resolve will also be specified in the `Props` section of your app\. Provide the correct values here\. For property descriptions and guidance, you can reference the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html)\.

   The following is an example of the `Props` section within a migrated CDK app with two write\-only properties to resolve:

   ```
   export interface MyTestAppStackProps extends cdk.StackProps {
     /**
      * The Amazon S3 key of the deployment package.
      */
     readonly lambdaFunction00asdfasdfsadf008grk1CodeS3Keym8P82: string;
     /**
      * An Amazon S3 bucket in the same AWS Region as your function. The bucket can be in a different AWS account.
      */
     readonly lambdaFunction00asdfasdfsadf008grk1CodeS3Bucketzidw8: string;
   }
   ```

Once you resolve all write\-only property values, you’re ready to prepare for deployment\.

### The migrate\.json file<a name="migrate-resources-file"></a>

The AWS CDK CLI creates a `migrate.json` file in your AWS CDK project during migration\. This file contains reference information on your deployed resources\. When you deploy your CDK app for the first time, the AWS CDK CLI uses this file to reference your deployed resources, associates your resources with the new AWS CloudFormation stack, and deletes the file\.

## Manage and deploy your CDK app<a name="migrate-manage"></a>

When migrating to AWS CDK, the new CDK app may not be deployment\-ready immediately\. This topic describes action items to consider while managing and deploying your new CDK app\.

### Prepare for deployment<a name="migrate-manage-prepare"></a>

Before deploying, you must prepare your CDK app\.

**Synthesize your app**  
Use the `cdk synth` command to synthesize the stack in your CDK app into an AWS CloudFormation template\.  
If you migrated from a deployed AWS CloudFormation stack or template, you can compare the synthesized template to the migrated template to verify resource and property values\.  
To learn more about `cdk synth`, see [Synthesizing stacks](cli.md#cli-synth)\.

**Perform a diff**  
If you migrated from a deployed AWS CloudFormation stack, you can use the cdk diff command to compare with the stack in your new CDK app\.  
To learn more about cdk diff, see [Comparing stacks](cli.md#cli-diff)\.

**Bootstrap your environment**  
If you are deploying from an AWS environment for the first time, use `cdk bootstrap` to prepare your environment\. To learn more, see [Bootstrapping](bootstrapping.md)\.

### Deploy your CDK app<a name="migrate-manage-deploy"></a>

When you deploy a CDK app, the AWS CDK CLI utilizes the AWS CloudFormation service to provision your resources\. Resources are bundled into a single stack in the CDK app and are deployed as a single AWS CloudFormation stack\.

Depending on where you migrated from, you can deploy to create a new AWS CloudFormation stack or update an existing AWS CloudFormation stack\.

**Deploy to create a new AWS CloudFormation stack**  
If you migrated from deployed resources, the AWS CDK CLI will automatically create a new AWS CloudFormation stack at deployment\. Your deployed resources will be included in the new AWS CloudFormation stack\.  
If you migrated from a local AWS CloudFormation template that was never deployed, the AWS CDK CLI will automatically create a new AWS CloudFormation stack at deployment\.  
If you migrated from a deployed AWS CloudFormation stack or local AWS CloudFormation template that was previously deployed, you can deploy to create a new AWS CloudFormation stack\. To create a new stack, do the following:  
+ Deploy to a new AWS environment\. This consists of using a different AWS account or deploying to a different AWS Region\.
+ If you want to deploy a new stack to the same AWS environment of the migrated stack or template, you must modify the stack name in your CDK app to a new value\. You must also modify all logical IDs of resources in your CDK app\. Then, you can deploy to the same environment to create a new stack and new resources\.

**Deploy to update an existing AWS CloudFormation stack**  
If you migrated from a deployed AWS CloudFormation stack or local AWS CloudFormation template that was previously deployed, you can deploy to update the existing AWS CloudFormation stack\.  
Verify that the stack name in your CDK app matches the stack name of the deployed AWS CloudFormation stack and deploy to the same AWS environment\.