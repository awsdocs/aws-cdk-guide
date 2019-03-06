--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Get a Value from AWS Secrets Manager<a name="passing_secrets_manager"></a>

To use values from AWS Secrets Manager in your CDK app, create an instance of [SecretsManager](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-secretsmanager.html/_aws-cdk_aws-secretsmanager.html#aws-cdk-aws-secretsmanager)\. It represents a value that is retrieved from Secrets Manager and used at AWS CloudFormation deployment time\.

```
import secretsmanager = require('@amp;aws-cdk/aws-secretsmanager');

const loginSecret = new secretsmanager.SecretString(stack, 'Secret', {
  secretId: 'MyLogin'

  // By default, the latest version is retrieved. It's possible to
  // use a specific version instead.
  // versionStage: 'AWSCURRENT'
});

// Retrieve a value from the secret's JSON
const username = loginSecret.jsonFieldValue('username');
const password = loginSecret.jsonFieldValue('password');

// Retrieve the whole secret's string value
const fullValue = loginSecret.value;
```