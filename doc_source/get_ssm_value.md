# Get a Value from the Systems Manager Parameter Store<a name="get_ssm_value"></a>

The AWS CDK can retrieve the value of AWS Systems Manager Parameter Store attributes\. During synthesis, the AWS CDK produces a [token](tokens.md) that is resolved by AWS CloudFormation during deployment\.

The AWS CDK supports retrieving both plain and secure values\. You may request a specific version of either kind of value\. For plain values only, you may omit the version from your request to receive the latest version\. You must always specify the version when requesting the value of a secure attribute\.

**Note**  
 This topic shows how to read attributes from the AWS Systems Manager Parameter Store\. You can also read secrets from the AWS Secrets Manager \(see [Get a Value from AWS Secrets Manager](get_secrets_manager_value.md)\)\.

## Reading Systems Manager Values at Deployment Time<a name="ssm_read"></a>

To read values from the Systems Manager Parameter Store, use the [valueForStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-string-parameterscope-parametername-version) and [valueForSecureStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-secure-string-parameterscope-parametername-version) methods, depending on whether the attribute you want is a plain string or a secure string value\. These methods return [tokens](tokens.md), not the actual value\. The value is resolved by AWS CloudFormation during deployment\.

```
import ssm = require('@aws-cdk/aws-ssm');

// Get latest version or specified version of plain string attribute
const latestStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name');      // latest version
const versionOfStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name', 1);   // version 1

// Get specified version of secure string attribute
const secureStringToken = ssm.StringParameter.valueForSecureStringParameter(
    this, 'my-secure-parameter-name', 1);   // must specify version
```

## Reading Systems Manager Values at Synthesis Time<a name="ssm_read"></a>

It is sometimes useful to "bake in" a parameter at synthesis time, so that the resulting AWS CloudFormation template always uses the same value, rather than resolving the value during deployment\.

To read a value from the Systems Manager parameter store at synthesis time, use the [valueFromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-wbr-from-wbr-lookupscope-parametername) method\. This method returns the actual value of the parameter as a [Runtime Context](context.md) value\. If the value is not already cached in `cdk.json` or passed on the command line, it will be retrieved from the current AWS account\. For this reason, the stack *must* be synthesized with explicit account and region information\.

```
import ssm = require('@aws-cdk/aws-ssm');

// ... later ...
const stringValue = ssm.StringParameter.valueFromLookup(this, 'my-plain-parameter-name');
```

**Note**  
Only plain Systems Manager strings may be retrieved, not secure strings\. It is not possible to request a specific version; the latest version is always returned\.

## Writing Values to Systems Manager<a name="ssm_write"></a>

You can use the AWS CLI, the AWS Management Console, or an AWS SDK to set Systems Manager parameter values\. The following examples use the [ssm put\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html) CLI command\.

```
aws ssm put-parameter --name "parameter-name" --type "String" --value "parameter-value"
aws ssm put-parameter --name "secure-parameter-name" --type "SecureString" --value "secure-parameter-value"
```

**Note**  
When updating an SSM value that already exists, also include the `--overwrite` option\.