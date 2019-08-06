# Get a Value from the Systems Manager Parameter Store<a name="get_ssm_value"></a>

The AWS CDK can retrieve the value of AWS Systems Manager Parameter Store attribute at synthesis time \(as the AWS CloudFormation template is being generated\) or at deployment time \(when the synthesized template is being deployed by AWS CloudFormation\)\. In the latter case, the AWS CDK provides a [token](tokens.md) that is later resolved by AWS CloudFormation\.

The AWS CDK supports retrieving both plain and secure values\. You may request a specific version of either kind of value\. For plain values only, you may omit the version from your request to receive the latest version\. You must always specify the version when requesting the value of a secure attribute\.

**Note**  
 This topic shows how to read attributes from the AWS Systems Manager Parameter Store\. You can also read secrets from the AWS Secrets Manager \(see [Get a Value from AWS Secrets Manager](get_secrets_manager_value.md)\)\.

## Reading Values at Synthesis Time<a name="ssm_read"></a>

To read a particular version of a Systems Manager Parameter Store plain string value at synthesis time, use the [fromStringParameterAttributes](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-from-string-parameter-attributesscope-id-attrs) method\. If you don't supply a `version` value, you get the latest version\.

```
import ssm = require('@aws-cdk/aws-ssm');

const parameterString = new ssm.StringParameter.fromStringParameterAttributes(this, 'MyParameter', {
    parameterName: 'my-plain-parameter-name',
    version: 1,   // omit to get latest version
});

const myValue = parameterString.stringValue;
```

To read a particular version of a Systems Manager Parameter Store secure string value at synthesis time, use [fromSecureStringParameterAttributes](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-from-secure-string-parameter-attributesscope-id-attrs)\. You must supply a `version` value for secure strings\.

```
import ssm = require('@aws-cdk/aws-ssm');

const secureString = new ssm.StringParameter.fromSecureStringParameterAttributes(this, 'MySecretParameter', {
    parameterName: 'my-secure-parameter-name',
    version: 1,
});

const myValue = secureString.stringValue;
```

## Reading Values at Deployment Time<a name="ssm_read_token"></a>

To read values from the Systems Manager Parameter Store at deployment time, use the [valueForStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-string-parameterscope-parametername-version) and [valueForSecureStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-secure-string-parameterscope-parametername-version) methods, depending on whether the attribute you want is a plain string or a secure string value\. These methods return [tokens](tokens.md) that are later resolved by AWS CloudFormation during deployment\.

```
import ssm = require('@aws-cdk/aws-ssm');

// Get latest version of specified version of plain string attribute
const latestStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name');      // latest version
const versionOfStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name', 1);   // version 1

// Get specified version of secure string attribute
const secureStringToken = new ssm.StringParameter.valueForSecureStringParameter(
    this, 'my-secure-parameter-name', 1);   // must specify version
```

## Writing Values to Systems Manager<a name="ssm_write"></a>

Use the [ssm put\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html) CLI command to add a string parameter to the Systems Manager, such as when testing:

```
aws ssm put-parameter --name "parameter-name" --type "String" --value "parameter-value"
  aws ssm put-parameter --name "secure-parameter-name" --type "SecureString" --value "secure-parameter-value"
```

This command returns an ARN that you can use to retrieve the value in your AWS CDK code\.