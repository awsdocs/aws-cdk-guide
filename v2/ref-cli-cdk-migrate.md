# cdk migrate<a name="ref-cli-cdk-migrate"></a>

Migrate deployed AWS resources, AWS CloudFormation stacks, and CloudFormation templates into a new AWS CDK project\.

This command creates a new CDK app that includes a single stack that is named with the value you provide using `--stack-name`\. You can configure the migration source using `--from-scan`, `--from-stack`, or `--from-path`\.

For more information on using `cdk migrate`, see [Migrate existing resources and AWS CloudFormation templates to the AWS CDK](migrate.md)\.

**Note**  
The `cdk migrate` command is experimental and may have breaking changes in the future\.

## Usage<a name="ref-cli-cdk-migrate-usage"></a>

```
$ cdk migrate <options>
```

## Options<a name="ref-cli-cdk-migrate-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

### Required options<a name="ref-cli-cdk-migrate-options-required"></a>

`--stack-name STRING`  <a name="ref-cli-cdk-migrate-options-stack-name"></a>
The name of the AWS CloudFormation stack that will be created within the CDK app after migrating\.  
*Required*: Yes

### Conditional options<a name="ref-cli-cdk-migrate-options-conditional"></a>

`--from-path PATH`  <a name="ref-cli-cdk-migrate-options-from-path"></a>
The path to the AWS CloudFormation template to migrate\. Provide this option to specify a local template\.  
*Required*: Conditional\. Required if migrating from a local AWS CloudFormation template\.

`--from-scan STRING`  <a name="ref-cli-cdk-migrate-options-from-scan"></a>
When migrating deployed resources from an AWS environment, use this option to specify whether a new scan should be started or if the AWS CDK CLI should use the last successful scan\.  
*Required*: Conditional\. Required when migrating from deployed AWS resources\.  
*Accepted values*: `most-recent`, `new`

`--from-stack BOOLEAN`  <a name="ref-cli-cdk-migrate-options-from-stack"></a>
Provide this option to migrate from a deployed AWS CloudFormation stack\. Use `--stack-name` to specify the name of the deployed AWS CloudFormation stack\.  
*Required*: Conditional\. Required if migrating from a deployed AWS CloudFormation stack\.

### Optional options<a name="ref-cli-cdk-migrate-options-optional"></a>

`--account STRING`  <a name="ref-cli-cdk-migrate-options-account"></a>
The account to retrieve the AWS CloudFormation stack template from\.  
*Required*: No  
*Default*: The AWS CDK CLI obtains account information from default sources\.

`--compress BOOLEAN`  <a name="ref-cli-cdk-migrate-options-compress"></a>
Provide this option to compress the generated CDK project into a ZIP file\.  
*Required*: No

`--filter ARRAY`  <a name="ref-cli-cdk-migrate-options-filter"></a>
Use when migrating deployed resources from an AWS account and AWS Region\. This option specifies a filter to determine which deployed resources to migrate\.  
This option accepts an array of key\-value pairs, where **key** represents the filter type and **value** represents the value to filter\.  
The following are accepted keys:  
+ `resource-identifier` – An identifier for the resource\. Value can be the resource logical or physical ID\. For example, `resource-identifier="ClusterName"`\.
+ `resource-type-prefix` – The AWS CloudFormation resource type prefix\. For example, specify `resource-type-prefix="AWS::DynamoDB::"` to filter all Amazon DynamoDB resources\.
+ `tag-key` – The key of a resource tag\. For example, `tag-key="myTagKey"`\.
+ `tag-value` – The value of a resource tag\. For example, `tag-value="myTagValue"`\.
Provide multiple key\-value pairs for `AND` conditional logic\. The following example filters for any DynamoDB resource that is tagged with `myTagKey` as the tag key: `--filter resource-type-prefix="AWS::DynamoDB::", tag-key="myTagKey"`\.  
Provide the `--filter` option multiple times in a single command for `OR` conditional logic\. The following example filters for any resource that is a DynamoDB resource or is tagged with `myTagKey` as the tag key: `--filter resource-type-prefix="AWS::DynamoDB::" --filter tag-key="myTagKey"`\.  
*Required*: No

`--help, -h BOOLEAN`  <a name="ref-cli-cdk-migrate-options-help"></a>
Show command reference information for the `cdk migrate` command\.

`--language STRING`  <a name="ref-cli-cdk-migrate-options-language"></a>
The programming language to use for the CDK project created during migration\.  
*Required*: No  
*Valid values*: `typescript`, `python`, `java`, `csharp`, `go`\.  
*Default*: `typescript`

`--output-path PATH`  <a name="ref-cli-cdk-migrate-options-output-path"></a>
The output path for the migrated CDK project\.  
*Required*: No  
*Default*: By default, the AWS CDK CLI will use your current working directory\.

`--region STRING`  <a name="ref-cli-cdk-migrate-options-region"></a>
The AWS Region to retrieve the AWS CloudFormation stack template from\.  
*Required*: No  
*Default*: The AWS CDK CLI obtains AWS Region information from default sources\.

## Examples<a name="ref-cli-cdk-migrate-examples"></a>

### Simple example of migrating from a CloudFormation stack<a name="ref-cli-cdk-migrate-examples-1"></a>

Migrate from a deployed CloudFormation stack in a specific AWS environment using `--from-stack`\. Provide `--stack-name` to name your new CDK stack\. The following is an example that migrates `myCloudFormationStack` to a new CDK app that is using TypeScript:

```
$ cdk migrate --language typescript --from-stack --stack-name 'myCloudFormationStack'
```

### Simple example of migrating from a local CloudFormation template<a name="ref-cli-cdk-migrate-examples-1"></a>

Migrate from a local JSON or YAML CloudFormation template using `--from-path`\. Provide `--stack-name` to name your new CDK stack\. The following is an example that creates a new CDK app in TypeScript that includes a `myCloudFormationStack` stack from a local `template.json` file:

```
$ cdk migrate --stack-name "myCloudFormationStack" --language typescript --from-path "./template.json"
```

### Simple example of migrating from deployed AWS resources<a name="ref-cli-cdk-migrate-examples-1"></a>

Migrate deployed AWS resources from a specific AWS environment that are not associated with a CloudFormation stack using `--from-scan`\. The CDK CLI utilizes the IaC generator service to scan for resources and generate a template\. Then, the CDK CLI references the template to create the new CDK app\. The following is an example that creates a new CDK app in TypeScript with a new `myCloudFormationStack` stack containing migrated AWS resources:

```
$ cdk migrate --language typescript --from-scan --stack-name "myCloudFormationStack"
```