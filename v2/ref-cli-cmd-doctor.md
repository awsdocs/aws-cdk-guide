# cdk doctor<a name="ref-cli-cmd-doctor"></a>

Inspect and display useful information about your local AWS CDK project and development environment\.

This information can help with troubleshooting CDK issues and should be provided when submitting bug reports\.

## Usage<a name="ref-cli-cmd-doctor-usage"></a>

```
$ cdk doctor <options>
```

## Options<a name="ref-cli-cmd-doctor-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-doctor-options-help"></a>
Show command reference information for the `cdk doctor` command\.

## Examples<a name="ref-cli-cmd-doctor-examples"></a>

### Simple example of the `cdk doctor` command<a name="ref-cli-cmd-doctor-examples-1"></a>

```
$ cdk doctor
ℹ️ CDK Version: 1.0.0 (build e64993a)
ℹ️ AWS environment variables:
  - AWS_EC2_METADATA_DISABLED = 1
  - AWS_SDK_LOAD_CONFIG = 1
```