# cdk bootstrap<a name="ref-cli-cmd-bootstrap"></a>

Prepare an AWS environment for CDK deployments by deploying the CDK bootstrap stack, named `CDKToolkit`, into the AWS environment\.

The bootstrap stack is a CloudFormation stack that provisions an Amazon S3 bucket and Amazon ECR repository in the AWS environment\. The AWS CDK CLI uses these resources to store synthesized templates and related assets during deployment\.

## Usage<a name="ref-cli-cmd-bootstrap-usage"></a>

```
$ cdk bootstrap <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-bootstrap-args"></a>

**AWS environment**  <a name="ref-cli-cmd-bootstrap-args-env"></a>
The target AWS environment to deploy the bootstrap stack to in the following format: `aws://<account-id>/<region>`\.  
Example: `aws://123456789012/us-east-1`  
This argument can be provided multiple times in a single command to deploy the bootstrap stack to multiple environments\.  
By default, the CDK CLI will bootstrap all environments referenced in the CDK app or will determine an environment from default sources\. This could be an environment specified using the `--profile` option, from environment variables, or default AWS CLI sources\.

## Options<a name="ref-cli-cmd-bootstrap-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--bootstrap-bucket-name, --toolkit-bucket-name, -b STRING`  <a name="ref-cli-cmd-bootstrap-options-bootstrap-bucket-name"></a>
The name of the Amazon S3 bucket that will be used by the CDK CLI\. This bucket will be created and must not currently exist\.  
Provide this option to override the default name of the Amazon S3 bucket\.  
When you use this option, you may have to customize synthesis\. To learn more, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.  
*Default value*: Undefined

`--bootstrap-customer-key BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-bootstrap-customer-key"></a>
Create a Customer Master Key \(CMK\) for the bootstrap bucket \(you will be charged but can customize permissions, modern bootstrapping only\)\.  
This option is not compatible with `--bootstrap-kms-key-id`\.  
*Default value*: Undefined

`--bootstrap-kms-key-id STRING`  <a name="ref-cli-cmd-bootstrap-options-bootstrap-kms-key-id"></a>
The AWS KMS master key ID to use for the SSE\-KMS encryption\.  
Provide this option to override the default AWS KMS key used to encrypt the Amazon S3 bucket\.  
This option is not compatible with `--bootstrap-customer-key`\.  
*Default value*: Undefined

`--cloudformation-execution-policies ARRAY`  <a name="ref-cli-cmd-bootstrap-options-cloudformation-execution-policies"></a>
The managed IAM policy ARNs that should be attached to the deployment role assumed by AWS CloudFormation during deployment of your stacks\.  
By default, stacks are deployed with full administrator permissions using the `AdministratorAccess` policy\.  
You can provide this option multiple times in a single command\. You can also provide multiple ARNs as a single string, with the individual ARNs separated by commas\. The following is an example:  

```
$ cdk bootstrap --cloudformation-execution-policies "arn:aws:iam::aws:policy/AWSLambda_FullAccess,arn:aws:iam::aws:policy/AWSCodeDeployFullAccess"
```
To avoid deployment failures, be sure that the policies you specify are sufficient for any deployments that you will perform into the environment being bootstrapped\.  
This option applies to modern bootstrapping only\.  
The modern bootstrap template effectively grants the permissions implied by the `--cloudformation-execution-policies` to any AWS account in the `--trust` list\. By default, this extends permissions to read and write to any resource in the bootstrapped account\. Make sure to [configure the bootstrapping stack](bootstrapping-customizing.md) with policies and trusted accounts that you are comfortable with\.
*Default value*: `[]`

`--custom-permissions-boundary, -cpb STRING`  <a name="ref-cli-cmd-bootstrap-options-custom-permissions-boundary"></a>
Specify the name of a permissions boundary to use\.  
This option is not compatible with `--example-permissions-boundary`\.  
*Default value*: Undefined

`--example-permissions-boundary, -epb BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-example-permissions-boundary"></a>
Use the example permissions boundary, supplied by the AWS CDK\.  
This option is not compatible with `--custom-permissions-boundary`\.  
The CDK supplied permissions boundary policy should be regarded as an example\. Edit the content and reference the example policy if you are testing out the feature\. Convert it into a new policy for actual deployments, if one does not already exist\. The concern is to avoid drift\. Most likely, a permissions boundary is maintained and has dedicated conventions, naming included\.  
For more information on configuring permissions, including using permissions boundaries, see the [Security and Safety Dev Guide](https://github.com/aws/aws-cdk/wiki/Security-And-Safety-Dev-Guide)\.  
*Default value*: Undefined

`--execute BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-execute"></a>
Configure whether to execute the change set\.  
*Default value*: `true`

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-force"></a>
Always bootstrap, even if it would downgrade the bootstrap template version\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-help"></a>
Show command reference information for the `cdk bootstrap` command\.

`--previous-parameters BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-previous-parameters"></a>
Use previous values for existing parameters\.  
Once a bootstrap template is deployed with a set of parameters, you must set this option to `false` to change any parameters on future deployments\. When `false`, you must re\-supply all previously supplied parameters\.  
*Default value*: `true`

`--public-access-block-configuration BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-public-access-block-configuration"></a>
Block public access configuration on the Amazon S3 bucket that is created and used by the CDK CLI\.  
*Default value*: `true`

`--qualifier STRING`  <a name="ref-cli-cmd-bootstrap-options-qualifier"></a>
String value that is unique for each bootstrap stack\. This value is added to the physical ID of resources in the bootstrap stack\.  
By providing a qualifier, you avoid resource name clashes when provisioning multiple bootstrap stacks in the same environment\.  
When you change the qualifier, your CDK app must pass the changed value to the stack synthesizer\. For more information, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.  
*Default value*: `hnb659fds`\. This value has no significance\.

`--show-template BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-show-template"></a>
Instead of bootstrapping, print the current bootstrap template to the standard output \(`stdout`\)\. You can then copy and customize the template as necessary\.  
*Default value*: `false`

`--tags, -t ARRAY`  <a name="ref-cli-cmd-bootstrap-options-tags"></a>
Tags to add to the bootstrap stack in the format of `KEY=VALUE`\.  
*Default value*: `[]`

`--template STRING`  <a name="ref-cli-cmd-bootstrap-options-template"></a>
Use the template from the given file instead of the built\-in one\.

`--termination-protection BOOLEAN`  <a name="ref-cli-cmd-bootstrap-options-termination-protection"></a>
Toggle AWS CloudFormation termination protection on the bootstrap stack\.  
When `true`, termination protection is enabled\. This prevents the bootstrap stack from being accidentally deleted\.  
To learn more about termination protection, see [Protecting a stack from being deleted](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-protect-stacks.html) in the *AWS CloudFormation User Guide*\.  
*Default value*: Undefined

`--toolkit-stack-name STRING`  <a name="ref-cli-cmd-bootstrap-options-toolkit-stack-name"></a>
The name of the bootstrap stack to create\.  
By default, `cdk bootstrap` deploys a stack named `CDKToolkit` into the specified AWS environment\. Use this option to provide a different name for your bootstrap stack\.  
*Default value*: `CDKToolkit`  
*Required*: Yes

`--trust ARRAY`  <a name="ref-cli-cmd-bootstrap-options-trust"></a>
The AWS account IDs that should be trusted to perform deployments into this environment\.  
The account performing the bootstrapping will always be trusted\.  
This option requires that you also provide `--cloudformation-execution-policies`\.  
You can provide this option multiple times in a single command\.  
This option applies to modern bootstrapping only\.  
To add trusted accounts to an existing bootstrap stack, you must specify all of the accounts to trust, including those that you may have previously provided\. If you only provide new accounts to trust, the previously trusted accounts will be removed\.  
The following is an example that trusts two accounts:  

```
$ cdk bootstrap aws://123456789012/us-west-2 --trust 234567890123 --trust 987654321098 --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess
 ⏳  Bootstrapping environment aws://123456789012/us-west-2...
Trusted accounts for deployment: 234567890123, 987654321098
Trusted accounts for lookup: (none)
Execution policies: arn:aws:iam::aws:policy/AdministratorAccess
CDKToolkit: creating CloudFormation changeset...
 ✅  Environment aws://123456789012/us-west-2 bootstrapped.
```
The modern bootstrap template effectively grants the permissions implied by the `--cloudformation-execution-policies` to any AWS account in the `--trust` list\. By default, this extends permissions to read and write to any resource in the bootstrapped account\. Make sure to [configure the bootstrapping stack](bootstrapping-customizing.md) with policies and trusted accounts that you are comfortable with\.
*Default value*: `[]`

`--trust-for-lookup ARRAY`  <a name="ref-cli-cmd-bootstrap-options-trust-for-lookup"></a>
The AWS account IDs that should be trusted to look up values in this environment\.  
Use this option to give accounts permission to synthesize stacks that will be deployed into the environment, without actually giving them permission to deploy those stacks directly\.  
You can provide this option multiple times in a single command\.  
This option applies to modern bootstrapping only\.  
*Default value*: `[]`

## Examples<a name="ref-cli-cmd-bootstrap-examples"></a>

### Bootstrap the AWS environment specified in the prod profile<a name="ref-cli-cmd-bootstrap-examples-1"></a>

```
$ cdk bootstrap --profile prod
```

### Deploy the bootstrap stack to environments foo and bar<a name="ref-cli-cmd-bootstrap-examples-2"></a>

```
$ cdk bootstrap --app='node bin/main.js' foo bar
```

### Export the bootstrap template to customize it<a name="ref-cli-cmd-bootstrap-examples-3"></a>

If you have specific requirements that are not met by the bootstrap template, you can customize it to fit your needs\.

You can export the bootstrap template, modify it, and deploy it using AWS CloudFormation\. The following is an example of exporting the existing template:

```
$ cdk bootstrap --show-template > bootstrap-template.yaml
```

You can also tell the CDK CLI to use a custom template\. The following is an example:

```
$ cdk bootstrap --template my-bootstrap-template.yaml
```

### Bootstrap with a permissions boundary\. Then, remove that permissions boundary<a name="ref-cli-cmd-bootstrap-examples-4"></a>

To bootstrap with a custom permissions boundary, we run the following:

```
$ cdk bootstrap --custom-permissions-boundary my-permissions-boundary
```

To remove the permissions boundary, we run the following:

```
$ cdk bootstrap --no-previous-parameters
```

### Use a qualifier to distinguish resources that are created for a development environment<a name="ref-cli-cmd-bootstrap-examples-5"></a>

```
$ cdk bootstrap --qualifier dev2024
```