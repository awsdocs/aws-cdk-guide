# cdk deploy<a name="ref-cli-cmd-deploy"></a>

Deploy one or more AWS CDK stacks into your AWS environment\.

During deployment, the CDK CLI will output progress indicators, similar to what can be observed from the AWS CloudFormation console\.

If the AWS environment is not bootstrapped, only stacks without assets and with synthesized templates under 51,200 bytes will successfully deploy\.

## Usage<a name="ref-cli-cmd-deploy-usage"></a>

```
$ cdk deploy <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-deploy-args"></a>

**CDK stack logical ID**  <a name="ref-cli-cmd-deploy-args-stack-name"></a>
The logical ID of the CDK stack from your app to deploy\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-deploy-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--all BOOLEAN`  <a name="ref-cli-cmd-deploy-options-all"></a>
Deploy all stacks in your CDK app\.  
*Default value*: `false`

`--asset-parallelism BOOLEAN`  <a name="ref-cli-cmd-deploy-options-asset-parallelism"></a>
Specify whether to build and publish assets in parallel\.

`--asset-prebuild BOOLEAN`  <a name="ref-cli-cmd-deploy-options-asset-prebuild"></a>
Specify whether to build all assets before deploying the first stack\. This option is useful for failing Docker builds\.  
*Default value*: `true`

`--build-exclude, -E ARRAY`  <a name="ref-cli-cmd-deploy-options-build-exclude"></a>
Do not rebuild asset with the given ID\.  
This option can be specified multiple times in a single command\.  
*Default value*: `[]`

`--change-set-name STRING`  <a name="ref-cli-cmd-deploy-options-change-set-name"></a>
The name of the AWS CloudFormation change set to create\.  
This option is not compatible with `--method='direct'`\.

`--concurrency NUMBER`  <a name="ref-cli-cmd-deploy-options-concurrency"></a>
Deploy multiple stacks in parallel while accounting for inter\-stack dependencies\. Use this option to speed up deployments\. You must still factor in AWS CloudFormation and other AWS account rate limiting\.  
Provide a number to specify the maximum number of simultaneous deployments \(dependency permitting\) to perform\.  
*Default value*: `1`

`--exclusively, -e BOOLEAN`  <a name="ref-cli-cmd-deploy-options-exclusively"></a>
Only deploy requested stacks and don't include dependencies\.

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-deploy-options-force"></a>
When you deploy to update an existing stack, the CDK CLI will compare the template and tags of the deployed stack to the stack about to be deployed\. If no changes are detected, the CDK CLI will skip deployment\.  
To override this behavior and always deploy stacks, even if no changes are detected, use this option\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-deploy-options-help"></a>
Show command reference information for the `cdk deploy` command\.

`--hotswap BOOLEAN`  <a name="ref-cli-cmd-deploy-options-hotswap"></a>
Hotswap deployments for faster development\. This option attempts to perform a faster, hotswap deployment if possible\. For example, if you modify the code of a Lambda function in your CDK app, the CDK CLI will update the resource directly through service APIs instead of performing a CloudFormation deployment\.  
If the CDK CLI detects changes that don’t support hotswapping, those changes will be ignored and a message will display\. If you prefer to perform a full CloudFormation deployment as a fall back, use `--hotswap-fallback` instead\.  
The CDK CLI uses your current AWS credentials to perform the API calls\. It does not assume the roles from your bootstrap stack, even if the `@aws-cdk/core:newStyleStackSynthesis` feature flag is set to `true`\. Those roles do not have the necessary permissions to update AWS resources directly, without using CloudFormation\. For that reason, make sure that your credentials are for the same AWS account of the stacks that you are performing hotswap deployments against and that they have the necessary IAM permissions to update the resources\.  
Hotswapping is currently supported for the following changes:  
+ Code assets \(including Docker images and inline code\), tag changes, and configuration changes \(only description and environment variables are supported\) of Lambda functions\.
+ Lambda versions and alias changes\.
+ Definition changes of AWS Step Functions state machines\.
+ Container asset changes of Amazon ECS services\.
+ Website asset changes of Amazon S3 bucket deployments\.
+ Source and environment changes of AWS CodeBuild projects\.
+ VTL mapping template changes for AWS AppSync resolvers and functions\.
+ Schema changes for AWS AppSync GraphQL APIs\.
Usage of certain CloudFormation intrinsic functions are supported as part of a hotswapped deployment\. These include:  
+ `Ref`
+ `Fn::GetAtt` – Only partially supported\. Refer to [this implementation](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk/lib/api/evaluate-cloudformation-template.ts#L477-L492) for supported resources and attributes\.
+ `Fn::ImportValue`
+ `Fn::Join`
+ `Fn::Select`
+ `Fn::Split`
+ `Fn::Sub`
This option is also compatible with nested stacks\.  
+ This option deliberately introduces drift in CloudFormation stacks in order to speed up deployments\. For this reason, only use it for development purposes\. Do not use this option for your production deployments\.
+ This option is considered experimental and may have breaking changes in the future\.
+ Defaults for certain parameters may be different with the hotswap parameter\. For example, an Amazon ECS service’s minimum healthy percentage will currently be set at `0`\. Review the source accordingly if this occurs\. 
*Default value*: `false`

`--hotswap-fallback BOOLEAN`  <a name="ref-cli-cmd-deploy-options-hotswap-fallback"></a>
This option is is similar to `--hotswap`\. The difference being that `--hotswap-fallback` will fall back to perform a full CloudFormation deployment if a change is detected that requires it\.  
For more information about this option, see `--hotswap`\.  
*Default value*: `false`

`--ignore-no-stacks BOOLEAN`  <a name="ref-cli-cmd-deploy-options-ignore-no-stacks"></a>
Perform a deployment even if your CDK app doesn’t contain any stacks\.  
This option is helpful in the following scenario: You may have an app with multiple environments, such as `dev` and `prod`\. When starting development, your prod app may not have any resources, or the resources may be commented out\. This will result in a deployment error with a message stating that the app has no stacks\. Use `--ignore-no-stacks` to bypass this error\.  
*Default value*: `false`

`--logs BOOLEAN`  <a name="ref-cli-cmd-deploy-options-logs"></a>
Show Amazon CloudWatch log in the standard output \(`stdout`\) for all events from all resources in the selected stacks\.  
This option is only compatible with `--watch`\.  
*Default value*: `true`

`--method, -m STRING`  <a name="ref-cli-cmd-deploy-options-method"></a>
Configure the method to perform a deployment\.  
+ `change-set` – Default method\. The CDK CLI creates a CloudFormation change set with the changes that will be deployed, then performs deployment\.
+ `direct` – Do not create a change set\. Instead, apply the change immediately\. This is typically faster than creating a change set, but you lose progress information\.
+ `prepare-change-set` – Create change set but don’t perform deployment\. This is useful if you have external tools that will inspect the change set or if you have an approval process for change sets\.
*Valid values*: `change-set`, `direct`, `prepare-change-set`  
*Default value*: `change-set`

`--notification-arns ARRAY`  <a name="ref-cli-cmd-deploy-options-notification-arns"></a>
The ARNs of Amazon SNS topics that CloudFormation will notify for stack related events\.

`--outputs-file, -O STRING`  <a name="ref-cli-cmd-deploy-options-outputs-file"></a>
The path to where stack outputs from deployments are written to\.  
After deployment, stack outputs will be written to the specified output file in JSON format\.  
You can configure this option in the project’s `cdk.json` file or at `~/.cdk.json` on your local development machine:  

```
{
   "app": "npx ts-node bin/myproject.ts",
   // ...
   "outputsFile": "outputs.json"
}
```
If multiple stacks are deployed, outputs are written to the same output file, organized by keys representing the stack name\.

`--parameters ARRAY`  <a name="ref-cli-cmd-deploy-options-parameters"></a>
Pass additional parameters to CloudFormation during deployment\.  
This option accepts an array in the following format: `STACK:KEY=VALUE`\.  
+ `STACK` – The name of the stack to associate the parameter with\.
+ `KEY` – The name of the parameter from your stack\.
+ `VALUE` – The value to pass at deployment\.
If a stack name is not provided, or if `*` is provided as the stack name, parameters will be applied to all stacks being deployed\. If a stack does not make use of the parameter, deployment will fail\.  
Parameters do not propagate to nested stacks\. To pass parameters to nested stacks, use the `[NestedStack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.NestedStack.html)` construct\.  
*Default value*: `{}`

`--previous-parameters BOOLEAN`  <a name="ref-cli-cmd-deploy-options-previous-parameters"></a>
Use previous values for existing parameters\.  
When this option is set to `false`, you must specify all parameters on every deployment\.  
*Default value*: `true`

`--progress STRING`  <a name="ref-cli-cmd-deploy-options-progress"></a>
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

`--require-approval STRING`  <a name="ref-cli-cmd-deploy-options-require-approval"></a>
Specify what security\-sensitive changes require manual approval\.  
+ `any-change ` – Manual approval required for any change to the stack\.
+ `broadening` – Manual approval required if changes involve a broadening of permissions or security group rules\.
+ `never` – Approval is not required\.
*Valid values*: `any-change`, `broadening`, `never`  
*Default value*: `broadening`

`--rollback BOOLEAN`  <a name="ref-cli-cmd-deploy-options-rollback"></a>
During deployment, if a resource fails to be created or updated, the deployment will roll back to the latest stable state before the CDK CLI returns\. All changes made up to that point will be undone\. Resources that were created will be deleted and updates that were made will be rolled back\.  
Specify `false` to deactivate this behavior\. If a resource fails to be created or updated, the CDK CLI will leave changes made up to that point in place and return\. This may be helpful in development environments where you are iterating quickly\.  
For `--rollback=false`, you can use `--no-rollback` or `-R`\.  
When `false`, deployments that cause resource replacements will always fail\. You can only use this option value for deployments that update or create new resources\.
*Default value*: `true`

`--toolkit-stack-name STRING`  <a name="ref-cli-cmd-deploy-options-toolkit-stack-name"></a>
The name of the existing CDK Toolkit stack\.  
This option is only used for CDK apps using legacy synthesis\.

`--watch BOOLEAN`  <a name="ref-cli-cmd-deploy-options-watch"></a>
Continuously observe CDK project files, and deploy the specified stacks automatically when changes are detected\.  
This option implies `--hotswap` by default\.  
This option has an equivalent CDK CLI command\. For more information, see [cdk watch](ref-cli-cmd-watch.md)\.

## Examples<a name="ref-cli-cmd-deploy-examples"></a>

### Deploy the stack named MyStackName<a name="ref-cli-cmd-deploy-examples-1"></a>

```
$ cdk deploy MyStackName --app='node bin/main.js'
```

### Deploy multiple stacks in an app<a name="ref-cli-cmd-deploy-examples-2"></a>

Use `cdk list` to list your stacks:

```
$ cdk list
CdkHelloWorldStack
CdkStack2
CdkStack3
```

To deploy all stacks, use the `--all` option:

```
$ cdk deploy --all
```

To choose which stacks to deploy, provide stack names as arguments:

```
$ cdk deploy CdkHelloWorldStack CdkStack3
```

### Deploy pipeline stacks<a name="ref-cli-cmd-deploy-examples-3"></a>

Use `cdk list` to show stack names as paths, showing where they are in the pipeline hierarchy:

```
$ cdk list
PipelineStack
PiplelineStack/Prod
PipelineStack/Prod/MyService
```

Use the `--all` option or the wildcard `*` to deploy all stacks\. If you have a hierarchy of stacks as described above, `--all` and `*` will only match stacks on the top level\. To match all stacks in the hierarchy, use `**`\.

You can combine these patterns\. The following deploys all stacks in the `Prod` stage:

```
$ cdk deploy PipelineStack/Prod/**
```

### Pass parameters at deployment<a name="ref-cli-cmd-deploy-examples-4"></a>

Define parameters in your CDK stack\. The following is an example that creates a parameter named `TopicNameParam` for an Amazon SNS topic:

```
new sns.Topic(this, 'TopicParameter', {
    topicName: new cdk.CfnParameter(this, 'TopicNameParam').value.toString()
});
```

To provide a parameter value of `parameterized`, run the following:

```
$ cdk deploy --parameters "MyStackName:TopicNameParam=parameterized"
```

You can override parameter values by using the `--force` option\. The following is an example of overriding the topic name from a previous deployment:

```
$ cdk deploy --parameters "MyStackName:TopicNameParam=parameterName" --force
```

### Write stack outputs to a file after deployment<a name="ref-cli-cmd-deploy-examples-5"></a>

Define outputs in your CDK stack file\. The following is an example that creates an output for a function ARN:

```
const fn = new lambda.Function(this, "fn", {
  handler: "index.handler",
  code: lambda.Code.fromInline(`exports.handler = \${handler.toString()}`),
  runtime: lambda.Runtime.NODEJS_LATEST
});

new cdk.CfnOutput(this, 'FunctionArn', {
  value: fn.functionArn,
});
```

Deploy the stack and write outputs to `outputs.json`:

```
$ cdk deploy --outputs-file outputs.json
```

The following is an example of `outputs.json` after deployment:

```
{
  "MyStack": {
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:MyStack-fn5FF616E3-G632ITHSP5HK"
  }
}
```

From this example, the key `FunctionArn` corresponds to the logical ID of the `CfnOutput` instance\.

The following is an example of `outputs.json` after deployment when multiple stacks are deployed:

```
{
  "MyStack": {
    "FunctionArn": "arn:aws:lambda:us-east-1:123456789012:function:MyStack-fn5FF616E3-G632ITHSP5HK"
  },
  "AnotherStack": {
    "VPCId": "vpc-z0mg270fee16693f"
  }
}
```

### Modify the deployment method<a name="ref-cli-cmd-deploy-examples-6"></a>

To deploy faster, without using change sets, use `--method='direct'`:

```
$ cdk deploy --method='direct'
```

To create a change set but don’t deploy, use `--method='prepare-change-set'`\. By default, a change set named `cdk-deploy-change-set` will be created\. If a previous change set with this name exists, it will be overwritten\. If no changes are detected, an empty change set is still created\.

You can also name your change set\. The following is an example:

```
$ cdk deploy --method='prepare-change-set' --change-set-name='MyChangeSetName'
```