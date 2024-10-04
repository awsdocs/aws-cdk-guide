# cdk watch<a name="ref-cli-cmd-watch"></a>

Continuously watch a local AWS CDK project for changes to perform deployments and hotswaps\.

This command is similar to `cdk deploy`, except that it can perform continuous deployments and hotswaps through a single command\.

This command is a shortcut for `cdk deploy --watch`\.

To end a `cdk watch` session, interrupt the process by pressing `Ctrl+C`\.

The files that are observed is determined by the `"watch"` setting in your `cdk.json` file\. It has two sub\-keys, `"include"` and `"exclude"`, that accepts a single string or an array of strings\. Each entry is interpreted as a path relative to the location of the `cdk.json` file\. Both `*` and `**` are accepted\.

If you create a project using the `cdk init` command, the following default behavior is configured for `cdk watch` in your project’s `cdk.json` file:
+ `"include"` is set to `"**/*"`, which includes all files and directories in the root of the project\.
+ `"exclude"` is optional, except for files and folders already ignored by default\. This consists of files and directories starting with `.`, the CDK output directory, and the `node_modules` directory\.

The minimal setting to configure `watch` is `"watch": {}`\.

If either your CDK code or application code requires a build step before deployment, `cdk watch` works with the `"build"` key in the `cdk.json` file\.

**Note**  
This command is considered experimental and may have breaking changes in the future\. 

The same limitations of `cdk deploy --hotswap` applies to `cdk watch`\. For more information, see `cdk deploy \-\-hotswap`\.

## Usage<a name="ref-cli-cmd-watch-usage"></a>

```
$ cdk watch <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-watch-args"></a>

**CDK stack ID**  <a name="ref-cli-cmd-watch-args-stack-name"></a>
The construct ID of the CDK stack from your app to watch\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-watch-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--build-exclude, -E ARRAY`  <a name="ref-cli-cmd-watch-options-build-exclude"></a>
Do not rebuild asset with the given ID\.  
This option can be specified multiple times in a single command\.  
*Default value*: `[]`

`--change-set-name STRING`  <a name="ref-cli-cmd-watch-options-change-set-name"></a>
The name of the CloudFormation change set to create\.

`--concurrency NUMBER`  <a name="ref-cli-cmd-watch-options-concurrency"></a>
Deploy and hotswap multiple stacks in parallel while accounting for inter\-stack dependencies\. Use this option to speed up deployments\. You must still factor in CloudFormation and other AWS account rate limiting\.  
Provide a number to specify the maximum number of simultaneous deployments \(dependency permitting\) to perform\.  
*Default value*: `1`

`--exclusively, -e BOOLEAN`  <a name="ref-cli-cmd-watch-options-exclusively"></a>
Only deploy requested stacks and don't include dependencies\.

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-watch-options-force"></a>
Always deploy stacks, even if templates are identical\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-watch-options-help"></a>
Show command reference information for the `cdk watch` command\.

`--hotswap BOOLEAN`  <a name="ref-cli-cmd-watch-options-hotswap"></a>
By default, `cdk watch` uses hotswap deployments when possible to update your resources\. The CDK CLI will attempt to perform a hotswap deployment and will not fall back to a full CloudFormation deployment if unsuccessful\. Any changes detected that cannot be updated through a hotswap are ignored\.  
*Default value*: `true`

`--hotswap-fallback BOOLEAN`  <a name="ref-cli-cmd-watch-options-hotswap-fallback"></a>
By default, `cdk watch` attempts to perform hotswap deployments and ignores changes that require CloudFormation deployments\. Provide `--hotswap-fallback` to fall back and perform a full CloudFormation deployment if the hotswap deployment is unsuccessful\.

`--logs BOOLEAN`  <a name="ref-cli-cmd-watch-options-logs"></a>
By default, `cdk watch` monitors all CloudWatch log groups in your application and streams the log events locally to `stdout`\.  
*Default value*: `true`

`--progress STRING`  <a name="ref-cli-cmd-watch-options-progress"></a>
Configure how the CDK CLI displays deployment progress\.  
+ `bar` – Display stack deployment events as a progress bar, with the events for the resource currently being deployed\.
+ `events` – Provide a complete history, including all CloudFormation events\.
You can also configure this option in the project’s `cdk.json` file or at `~/.cdk.json` on your local development machine:  

```
{
   "progress": "events"
}
```
*Valid values*: `bar`, `events`  
*Default value*: `bar`

`--rollback BOOLEAN`  <a name="ref-cli-cmd-watch-options-rollback"></a>
During deployment, if a resource fails to be created or updated, the deployment will roll back to the latest stable state before the CDK CLI returns\. All changes made up to that point will be undone\. Resources that were created will be deleted and updates that were made will be rolled back\.  
Use `--no-rollback` or `-R` to deactivate this behavior\. If a resource fails to be created or updated, the CDK CLI will leave changes made up to that point in place and return\. This may be helpful in development environments where you are iterating quickly\.  
When `false`, deployments that cause resource replacements will always fail\. You can only use this value for deployments that update or create new resources\.
*Default value*: `true`

`--toolkit-stack-name STRING`  <a name="ref-cli-cmd-watch-options-toolkit-stack-name"></a>
The name of the existing CDK Toolkit stack\.  
By default, `cdk bootstrap` deploys a stack named `CDKToolkit` into the specified AWS environment\. Use this option to provide a different name for your bootstrap stack\.  
The CDK CLI uses this value to verify your bootstrap stack version\.

## Examples<a name="ref-cli-cmd-watch-examples"></a>

### Watch a CDK stack with logical ID DevelopmentStack for changes<a name="ref-cli-cmd-watch-examples-1"></a>

```
$ cdk watch DevelopmentStack
Detected change to 'lambda-code/index.js' (type: change). Triggering 'cdk deploy'
DevelopmentStack: deploying...

 ✅  DevelopmentStack
```

### Configure a cdk\.json file for what to include and exclude from being watched for changes<a name="ref-cli-cmd-watch-examples-2"></a>

```
{
   "app": "mvn -e -q compile exec:java",
   "watch": {
    "include": "src/main/**",
    "exclude": "target/*"
   }
}
```

### Build a CDK project using Java before deployment by configuring the cdk\.json file<a name="ref-cli-cmd-watch-examples-3"></a>

```
{
  "app": "mvn -e -q exec:java",
  "build": "mvn package",
  "watch": {
    "include": "src/main/**",
    "exclude": "target/*"
  }
}
```