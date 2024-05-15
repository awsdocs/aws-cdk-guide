# cdk destroy<a name="ref-cli-cmd-destroy"></a>

Delete one or more AWS CDK stacks from your AWS environment\.

When you delete a stack, resources in the stack will be destroyed, unless they were configured with a `DeletionPolicy` of `Retain`\.

During stack deletion, this command will output progress information similar to `cdk deploy` behavior\.

## Usage<a name="ref-cli-cmd-destroy-usage"></a>

```
$ cdk destroy <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-destroy-args"></a>

**CDK stack logical ID**  <a name="ref-cli-cmd-destroy-args-stack-name"></a>
The logical ID of the CDK stack from your app to delete\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-destroy-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--all BOOLEAN`  <a name="ref-cli-cmd-destroy-options-all"></a>
Destroy all available stacks\.  
*Default value*: `false`

`--exclusively, -e BOOLEAN`  <a name="ref-cli-cmd-destroy-options-exclusively"></a>
Only destroy requested stacks and don’t include dependencies\.

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-destroy-options-force"></a>
Do not ask for confirmation before destroying the stacks\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-destroy-options-help"></a>
Show command reference information for the `cdk destroy` command\.

## Examples<a name="ref-cli-cmd-destroy-examples"></a>

### Delete a stack named MyStackName<a name="ref-cli-cmd-destroy-examples-1"></a>

```
$ cdk destroy --app='node bin/main.js' MyStackName
```