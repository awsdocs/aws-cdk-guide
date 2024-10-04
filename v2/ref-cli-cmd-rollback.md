# cdk rollback<a name="ref-cli-cmd-rollback"></a>

Use the AWS Cloud Development Kit \(AWS CDK\) Command Line Interface \(CLI\) `cdk rollback` command to rollback a failed or paused stack from an AWS CloudFormation deployment to its last stable state\.

**Note**  
To use this command, you must have v23 of the bootstrap template deployed to your environment\. For more information, see [Bootstrap template version history](bootstrapping-env.md#bootstrap-template-history)\.

When you deploy using `cdk deploy`, the CDK CLI will rollback a failed deployment by default\. If you specify `--no-rollback` with `cdk deploy`, you can then use the `cdk rollback` command to manually rollback a failed deployment\. This will initiate a rollback to the last stable state of your stack\.

## Usage<a name="ref-cli-cmd-rollback-usage"></a>

```
$ cdk rollback <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-rollback-args"></a>

**CDK stack ID**  <a name="ref-cli-cmd-rollback-args-stack-name"></a>
The construct ID of the CDK stack from your app to rollback\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-rollback-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--all BOOLEAN`  <a name="ref-cli-cmd-rollback-options-all"></a>
Rollback all stacks in your CDK app\.  
*Default value*: `false`

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-rollback-options-force"></a>
When you use `cdk rollback`, some resources may fail to rollback\. Provide this option to force the rollback of all resources\. This is the same behavior as providing the `--orphan` option for each resource in your stack\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-rollback-options-help"></a>
Show command reference information for the `cdk rollback` command\.

`--orphan LogicalId`  <a name="ref-cli-cmd-rollback-options-orphan"></a>
When you use `cdk rollback`, some resources may fail to rollback\. When this happens, you can try to force the rollback of a resource by using this option and providing the logical ID of the resource that failed to rollback\.  
This option can be provided multiple times in a single command The following is an example:  

```
$ cdk rollback MyStack --orphan MyLambdaFunction --orphan MyLambdaFunction2
```
To force the rollback of all resources, use the `--force` option instead\.

`--toolkit-stack-name STRING`  <a name="ref-cli-cmd-rollback-options-toolkit-stack-name"></a>
The name of the existing CDK Toolkit stack that the environment is bootstrapped with\.  
By default, `cdk bootstrap` deploys a stack named `CDKToolkit` into the specified AWS environment\. Use this option to provide a different name for your bootstrap stack\.  
The CDK CLI uses this value to verify your bootstrap stack version\.

`--validate-bootstrap-version BOOLEAN`  <a name="ref-cli-cmd-rollback-options-validate-bootstrap-version"></a>
Specify whether to validate the bootstrap stack version\. Provide `--validate-bootstrap-version=false` or `--no-validate-bootsrap-version` to turn off this behavior\.  
*Default value*: `true`