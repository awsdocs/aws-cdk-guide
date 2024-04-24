# AWS CDK CLI command reference<a name="ref-cli-cmd"></a>

This section contains command reference information for the AWS Cloud Development Kit \(AWS CDK\) Command Line Interface \(CLI\)\. The CDK CLI is also referred to as the CDK Toolkit\.

## Usage<a name="ref-cli-cmd-usage"></a>

```
$ cdk <command> <arguments> <options>
```

## Commands<a name="ref-cli-cmd-commands"></a>

`acknowledge, ack`  <a name="ref-cli-cmd-commands-acknowledge"></a>
Acknowledge a notice by issue number and hide it from displaying again\.

`bootstrap`  <a name="ref-cli-cmd-commands-bootstrap"></a>
Prepare an AWS environment for CDK deployments by deploying the CDK bootstrap stack, named `CDKToolkit`, into the AWS environment\.

`context`  <a name="ref-cli-cmd-commands-context"></a>
Manage cached context values for your CDK application\.

`deploy`  <a name="ref-cli-cmd-commands-deploy"></a>
Deploy one or more CDK stacks into your AWS environment\.

`destroy`  <a name="ref-cli-cmd-commands-destroy"></a>
Delete one or more CDK stacks from your AWS environment\.

`diff`  <a name="ref-cli-cmd-commands-diff"></a>
Perform a diff to see infrastructure changes between CDK stacks\.

`docs, doc`  <a name="ref-cli-cmd-commands-docs"></a>
Open CDK documentation in your browser\.

`doctor`  <a name="ref-cli-cmd-commands-doctor"></a>
Inspect and display useful information about your local CDK project and development environment\.

`import`  <a name="ref-cli-cmd-commands-import"></a>
Use AWS CloudFormation resource imports to import existing AWS resources into a CDK stack\.

`init`  <a name="ref-cli-cmd-commands-init"></a>
Create a new CDK project from a template\.

`list, ls`  <a name="ref-cli-cmd-commands-list"></a>
List all CDK stacks and their dependencies from a CDK app\.

`metadata`  <a name="ref-cli-cmd-commands-metadata"></a>
Display metadata associated with a CDK stack\.

`migrate`  <a name="ref-cli-cmd-commands-migrate"></a>
Migrate AWS resources, AWS CloudFormation stacks, and AWS CloudFormation templates into a new CDK project\.

`notices`  <a name="ref-cli-cmd-commands-notices"></a>
Display notices for your CDK application\.

`synthesize, synth`  <a name="ref-cli-cmd-commands-synthesize"></a>
Synthesize a CDK app to produce a cloud assembly, including an AWS CloudFormation template for each stack\.

`watch`  <a name="ref-cli-cmd-commands-watch"></a>
Continuously watch a local CDK project for changes to perform deployments and hotswaps\.

## Global options<a name="ref-cli-cmd-options"></a>

The following options are compatible with all CDK CLI commands\.

`--app, -a STRING`  <a name="ref-cli-cmd-options-app"></a>
Provide the command for running your app or cloud assembly directory\.  
*Required*: Yes

`--asset-metadata BOOLEAN`  <a name="ref-cli-cmd-options-asset-metadata"></a>
Include `aws:asset:*` AWS CloudFormation metadata for resources that use assets\.  
*Required*: No  
*Default value*: `true`

`--build STRING`  <a name="ref-cli-cmd-options-build"></a>
Command for running a pre\-synthesis build\.  
*Required*: No

`--ca-bundle-path STRING`  <a name="ref-cli-cmd-options-ca-bundle-path"></a>
Path to a CA certificate to use when validating HTTPS requests\.  
If this option is not provided, the CDK CLI will read from the `AWS_CA_BUNDLE` environment variable\.  
*Required*: Yes

`--ci BOOLEAN`  <a name="ref-cli-cmd-options-ci"></a>
Indicate that CDK CLI commands are being run in a continuous integration \(CI\) environment\.  
This option modifies the behavior of the CDK CLI to better suit automated operations that are typical in CI pipelines\.  
When you provide this option, logs are sent to `stdout` instead of `stderr`\.  
*Required*: No  
*Default value*: `false`

`--context, -c ARRAY`  <a name="ref-cli-cmd-options-context"></a>
Add contextual string parameters as key\-value pairs\.

`--debug BOOLEAN`  <a name="ref-cli-cmd-options-debug"></a>
Enable detailed debugging information\. This option produces a verbose output that includes a lot more detail about what the CDK CLI is doing behind the scenes\.  
*Required*: No  
*Default value*: `false`

`--ec2creds, -i BOOLEAN`  <a name="ref-cli-cmd-options-ec2creds"></a>
Force the CDK CLI to try and fetch Amazon EC2 instance credentials\.  
By default, the CDK CLI guesses the Amazon EC2 instance status\.  
*Required*: No  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-options-help"></a>
Show command reference information for the CDK CLI\.  
*Required*: No  
*Default value*: `false`

`--ignore-errors BOOLEAN`  <a name="ref-cli-cmd-options-ignore-errors"></a>
Ignore synthesis errors, which will likely produce an output that is not valid\.  
*Required*: No  
*Default value*: `false`

`--json, -j BOOLEAN`  <a name="ref-cli-cmd-options-json"></a>
Use JSON instead of YAML for AWS CloudFormation templates that are printed to standard output \(`stdout`\)\.  
*Required*: No  
*Default value*: `false`

`--lookups BOOLEAN`  <a name="ref-cli-cmd-options-lookups"></a>
Perform context lookups\.  
Synthesis will fail if this value is `false` and context lookups need to be performed\.  
*Required*: No  
*Default value*: `true`

`--no-color BOOLEAN`  <a name="ref-cli-cmd-options-no-color"></a>
Remove color and other styling from the console output\.  
*Required*: No  
*Default value*: `false`

`--notices BOOLEAN`  <a name="ref-cli-cmd-options-notices"></a>
Show relevant notices\.  
*Required*: No  
*Default value*: `false`

`--output, -o STRING`  <a name="ref-cli-cmd-options-output"></a>
Specify the directory to output the synthesized cloud assembly to\.  
*Required*: Yes  
*Default value*: `cdk.out`

`--path-metadata BOOLEAN`  <a name="ref-cli-cmd-options-path-metadata"></a>
Include `aws::cdk::path` AWS CloudFormation metadata for each resource\.  
*Required*: No  
*Default value*: `true`

`--plugin, -p ARRAY`  <a name="ref-cli-cmd-options-plugin"></a>
Name or path of a node package that extends CDK features\. This option can be provided multiple times in a single command\.  
You can configure this option in the project’s `cdk.json` file or at `~/.cdk.json` on your local development machine:  

```
{
   // ...
   "plugin": [
      "module_1",
      "module_2"
   ],
   // ...
}
```
*Required*: No

`--profile STRING`  <a name="ref-cli-cmd-options-profile"></a>
Specify the name of the AWS profile, containing your AWS environment information, to use with the CDK CLI\.  
*Required*: Yes

`--proxy STRING`  <a name="ref-cli-cmd-options-proxy"></a>
Use the indicated proxy\.  
If this option is not provided, the CDK CLI will read from the `HTTPS_PROXY` environment variable\.  
*Required*: Yes  
*Default value*: Read from `HTTPS_PROXY` environment variable\.

`--role-arn, -r STRING`  <a name="ref-cli-cmd-options-role-arn"></a>
The ARN of the IAM role that the CDK CLI will assume when interacting with AWS CloudFormation\.  
*Required*: No

`--staging BOOLEAN`  <a name="ref-cli-cmd-options-staging"></a>
Copy assets to the output directory\.  
Specify `false` to prevent the copying of assets to the output directory\. This allows the AWS SAM CLI to reference the original source files when performing local debugging\.  
*Required*: No  
*Default value*: `true`

`--strict BOOLEAN`  <a name="ref-cli-cmd-options-strict"></a>
Do not construct stacks that contain warnings\.  
*Required*: No  
*Default value*: `false`

`--trace BOOLEAN`  <a name="ref-cli-cmd-options-trace"></a>
Print trace for stack warnings\.  
*Required*: No  
*Default value*: `false`

`--verbose, -v COUNT`  <a name="ref-cli-cmd-options-verbose"></a>
Show debug logs\. You can specify this option multiple times to increase verbosity\.  
*Required*: No

`--version BOOLEAN`  <a name="ref-cli-cmd-options-version"></a>
Show the CDK CLI version number\.  
*Required*: No  
*Default value*: `false`

`--version-reporting BOOLEAN`  <a name="ref-cli-cmd-options-version-reporting"></a>
Include the `AWS::CDK::Metadata` resource in synthesized AWS CloudFormation templates\.  
*Required*: No  
*Default value*: `true`

## Providing and configuring options<a name="ref-cli-cmd-configure"></a>

You can pass options through command\-line arguments\. For most options, you can configure them in a `cdk.json` configuration file\. When you use multiple configuration sources, the CDK CLI adheres to the following precedence:

1. **Command\-line values** – Any option provided at the command\-line overrides options configured in `cdk.json` files\.

1. **Project configuration file** – The `cdk.json` file in your CDK project’s directory\.

1. **User configuration file** – The `cdk.json` file located at `~/.cdk.json` on your local machine\.

## Passing options at the command line<a name="ref-cli-cmd-pass"></a>

### Passing boolean values<a name="ref-cli-cmd-pass-bool"></a>

For options that accept a boolean value, you can specify them in the following ways:
+ Use `true` and `false` values – Provide the boolean value with the command\. The following is an example:

  ```
  $ cdk deploy --watch=true
  $ cdk deploy --watch=false
  ```
+ Provide the option’s counterpart – Modify the option name by adding `no` to specify a `false` value\. The following is an example:

  ```
  $ cdk deploy --watch
  $ cdk deploy --no-watch
  ```
+ For options that default to `true` or `false`, you don’t have to provide the option unless you want to change from the default\.