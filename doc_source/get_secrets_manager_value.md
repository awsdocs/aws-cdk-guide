# Get a Value from AWS Secrets Manager<a name="get_secrets_manager_value"></a>

To use values from AWS Secrets Manager in your CDK app, use the [fromSecretAttributes](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-secretsmanager/secret.html#aws_secretsmanager_Secret_fromSecretAttributes) method\. It represents a value that is retrieved from Secrets Manager and used at AWS CloudFormation deployment time\.

```
import sm = require("@aws-cdk/aws-secretsmanager");

export class SecretsManagerStack extends core.Stack {
  constructor(scope: core.App, id: string, props?: core.StackProps) {
    super(scope, id, props);

    const secret = sm.Secret.fromSecretAttributes(this, "ImportedSecret", {
      secretArn:
        "arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>"
      // If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
      // Key,
    });
```

Use the [create\-secret](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/create-secret.html) CLI command to create a secret from the command\-line, such as when testing:

```
aws secretsmanager create-secret --name ImportedSecret --secret-string mygroovybucket
```

The command returns an ARN you can use for the example\.