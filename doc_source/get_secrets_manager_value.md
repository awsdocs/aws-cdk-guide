# Get a value from AWS Secrets Manager<a name="get_secrets_manager_value"></a>

To use values from AWS Secrets Manager in your CDK app, use the [fromSecretAttributes](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-secretsmanager/secret.html#aws_secretsmanager_Secret_fromSecretAttributes) method\. It represents a value that is retrieved from Secrets Manager and used at AWS CloudFormation deployment time\.

------
#### [ TypeScript ]

```
import * as sm from "@aws-cdk/aws-secretsmanager";

export class SecretsManagerStack extends core.Stack {
  constructor(scope: core.App, id: string, props?: core.StackProps) {
    super(scope, id, props);

    const secret = sm.Secret.fromSecretAttributes(this, "ImportedSecret", {
      secretArn:
        "arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>"
      // If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
      // encryptionKey: ...
    });
```

------
#### [ JavaScript ]

```
const sm = require("@aws-cdk/aws-secretsmanager");

class SecretsManagerStack extends core.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const secret = sm.Secret.fromSecretAttributes(this, "ImportedSecret", {
      secretArn:
        "arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>"
      // If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
      // encryptionKey: ...
    });
  }
}

module.exports = { SecretsManagerStack }
```

------
#### [ Python ]

```
import aws_cdk.aws_secretsmanager as sm

class SecretsManagerStack(core.Stack):
    def __init__(self, scope: core.App, id: str, **kwargs):
      super().__init__(scope, name, **kwargs)

      secret = sm.Secret.from_secret_attributes(self, "ImportedSecret",
          secret_arn="arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>",
          # If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
          # encryption_key=....
      )
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.secretsmanager.Secret;
import software.amazon.awscdk.services.secretsmanager.SecretAttributes;

public class SecretsManagerStack extends Stack {
    public SecretsManagerStack(App scope, String id) {
        this(scope, id, null);
    }
    
    public SecretsManagerStack(App scope, String id, StackProps props) {
        super(scope, id, props);
        
        Secret secret = (Secret)Secret.fromSecretAttributes(this, "ImportedSecret", SecretAttributes.builder()
        .secretArn("arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>")
        // If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
        // .encryptionKey(...)
        .build());
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.SecretsManager;

public class SecretsManagerStack : Stack
{
    public SecretsManagerStack(App scope, string id, StackProps props) : base(scope, id, props) {

        var secret = Secret.FromSecretAttributes(this, "ImportedSecret", new SecretAttributes {
            SecretArn = "arn:aws:secretsmanager:<region>:<account-id-number>:secret:<secret-name>-<random-6-characters>"
            // If the secret is encrypted using a KMS-hosted CMK, either import or reference that key:
            // encryptionKey = ...,
        });
    }
```

------

Use the [create\-secret](https://docs.aws.amazon.com/cli/latest/reference/secretsmanager/create-secret.html) CLI command to create a secret from the command\-line, such as when testing:

```
aws secretsmanager create-secret --name ImportedSecret --secret-string mygroovybucket
```

The command returns an ARN you can use for the example\.