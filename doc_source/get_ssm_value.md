# Get a value from the Systems Manager Parameter Store<a name="get_ssm_value"></a>

The AWS CDK can retrieve the value of AWS Systems Manager Parameter Store attributes\. During synthesis, the AWS CDK produces a [token](tokens.md) that is resolved by AWS CloudFormation during deployment\.

The AWS CDK supports retrieving both plain and secure values\. You may request a specific version of either kind of value\. For plain values only, you may omit the version from your request to receive the latest version\. You must always specify the version when requesting the value of a secure attribute\.

**Note**  
 This topic shows how to read attributes from the AWS Systems Manager Parameter Store\. You can also read secrets from the AWS Secrets Manager \(see [Get a value from AWS Secrets Manager](get_secrets_manager_value.md)\)\.

## Reading Systems Manager values at deployment time<a name="ssm_read"></a>

To read values from the Systems Manager Parameter Store, use the [valueForStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-string-parameterscope-parametername-version) and [valueForSecureStringParameter](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-for-secure-string-parameterscope-parametername-version) methods, depending on whether the attribute you want is a plain string or a secure string value\. These methods return [tokens](tokens.md), not the actual value\. The value is resolved by AWS CloudFormation during deployment\.

------
#### [ TypeScript ]

```
import * as ssm from '@aws-cdk/aws-ssm';

// Get latest version or specified version of plain string attribute
const latestStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name');      // latest version
const versionOfStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name', 1);   // version 1

// Get specified version of secure string attribute
const secureStringToken = ssm.StringParameter.valueForSecureStringParameter(
    this, 'my-secure-parameter-name', 1);   // must specify version
```

------
#### [ JavaScript ]

```
const ssm = require('@aws-cdk/aws-ssm');

// Get latest version or specified version of plain string attribute
const latestStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name');      // latest version
const versionOfStringToken = ssm.StringParameter.valueForStringParameter(
    this, 'my-plain-parameter-name', 1);   // version 1

// Get specified version of secure string attribute
const secureStringToken = ssm.StringParameter.valueForSecureStringParameter(
    this, 'my-secure-parameter-name', 1);   // must specify version
```

------
#### [ Python ]

```
import aws_cdk.aws_ssm as ssm
          
# Get latest version or specified version of plain string attribute
latest_string_token = ssm.StringParameter.value_for_string_parameter(
    self, "my-plain-parameter-name")
latest_string_token = ssm.StringParameter.value_for_string_parameter(
    self, "my-plain-parameter-name", 1)

# Get specified version of secure string attribute
secure_string_token = ssm.StringParameter.value_for_secure_string_parameter(
    self, "my-secure-parameter-name", 1)   # must specify version
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.ssm.StringParameter;

//Get latest version or specified version of plain string attribute
String latestStringToken = StringParameter.valueForStringParameter(
            this, "my-plain-parameter-name");       // latest version
String versionOfStringToken = StringParameter.valueForStringParameter(
            this, "my-plain-parameter-name", 1);    // version 1

//Get specified version of secure string attribute
String secureStringToken = StringParameter.valueForSecureStringParameter(
            this, "my-secure-parameter-name", 1);   // must specify version
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.SSM;

// Get latest version or specified version of plain string attribute
var latestStringToken = StringParameter.ValueForStringParameter(
    this, 'my-plain-parameter-name');      // latest version
var versionOfStringToken = StringParameter.ValueForStringParameter(
    this, 'my-plain-parameter-name', 1);   // version 1

// Get specified version of secure string attribute
var secureStringToken = StringParameter.ValueForSecureStringParameter(
    this, 'my-secure-parameter-name', 1);   // must specify version
```

------

## Reading Systems Manager values at synthesis time<a name="ssm_read"></a>

It is sometimes useful to "bake in" a parameter at synthesis time, so that the resulting AWS CloudFormation template always uses the same value, rather than resolving the value during deployment\.

To read a value from the Systems Manager parameter store at synthesis time, use the [valueFromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-wbr-from-wbr-lookupscope-parametername) method \(Python: `value_from_lookup`\)\. This method returns the actual value of the parameter as a [Runtime context](context.md) value\. If the value is not already cached in `cdk.json` or passed on the command line, it will be retrieved from the current AWS account\. For this reason, the stack *must* be synthesized with explicit account and region information\.

------
#### [ TypeScript ]

```
import * as ssm from '@aws-cdk/aws-ssm';

const stringValue = ssm.StringParameter.valueFromLookup(this, 'my-plain-parameter-name');
```

------
#### [ JavaScript ]

```
const ssm = require('@aws-cdk/aws-ssm');

const stringValue = ssm.StringParameter.valueFromLookup(this, 'my-plain-parameter-name');
```

------
#### [ Python ]

```
import aws_cdk.aws_ssm as ssm

string_value = ssm.StringParameter.value_from_lookup(self, "my-plain-parameter-name")
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.ssm.StringParameter;

String stringValue = StringParameter.valueFromLookup(this, "my-plain-parameter-name");
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.SSM;

var stringValue = StringParameter.ValueFromLookup(this, "my-plain-parameter-name");
```

------

Only plain Systems Manager strings may be retrieved, not secure strings\. It is not possible to request a specific version; the latest version is always returned\.

## Writing values to Systems Manager<a name="ssm_write"></a>

You can use the AWS CLI, the AWS Management Console, or an AWS SDK to set Systems Manager parameter values\. The following examples use the [ssm put\-parameter](https://docs.aws.amazon.com/cli/latest/reference/ssm/put-parameter.html) CLI command\.

```
aws ssm put-parameter --name "parameter-name" --type "String" --value "parameter-value"
aws ssm put-parameter --name "secure-parameter-name" --type "SecureString" --value "secure-parameter-value"
```

When updating an SSM value that already exists, also include the `--overwrite` option\.

```
aws ssm put-parameter --overwrite --name "parameter-name" --type "String" --value "parameter-value"
aws ssm put-parameter --overwrite --name "secure-parameter-name" --type "SecureString" --value "secure-parameter-value"
```