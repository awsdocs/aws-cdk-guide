--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Get a Value from a Systems Manager Parameter Store Variable<a name="get_ssm_value"></a>

You can get the value of an AWS Systems Manager Parameter Store variable, depending on whether you want the latest version of a plain string, a particular version of a plain string, or a particular version of a secret string\. It isn't possible to retrieve the latest version of a secure string\. To read the latest version of a secret, you have to read the secret from AWS Secrets Manager \(see [Get a Value from AWS Secrets Manager](passing_secrets_manager.md)\)\.

To retrieve the latest value one time \(as a context value, see [Run\-Time Context](environments_and_context.md#context)\), and keep on using the same value until the context value is manually refreshed, use an [SSMParameterProvider](@cdk-class-url;#@aws-cdk/cdk.SSMParameterProvider)\.

```
import cdk = require('@aws-cdk/cdk');

const myvalue = new cdk.SSMParameterProvider(this, {parameterName: "my-parameter-name"}).parameterValue();
```

To read a particular version of a Systems Manager Parameter Store plain string value at AWS CloudFormation deployment time, use [ssm\.ParameterStoreString](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ssm.html/#parameterstorestring)\.

```
import ssm = require('@aws-cdk/aws-ssm');

const parameterString = new ssm.ParameterStoreString(this, 'MyParameter', {
    parameterName: 'my-parameter-name',
    version: 1,
});

const myvalue = parameterString.stringValue;
```

To read a particular version of a Systems Manager Parameter Store `SecureString` value at AWS CloudFormation deployment time, use [ssm\.ParameterStoreSecureString](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ssm.html/#parameterstoresecurestring)

```
import ssm = require('@aws-cdk/aws-ssm');

const secureString = new ssm.ParameterStoreSecureString(this, 'MySecretParameter', {
    parameterName: 'my-secret-parameter-name',
    version: 1,
});

const myvalue = secureString.stringValue;
```
