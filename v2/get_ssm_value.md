# Get a value from the Systems Manager Parameter Store<a name="get_ssm_value"></a>

The AWS Cloud Development Kit \(AWS CDK\) can retrieve the value of AWS Systems Manager Parameter Store attributes\. During synthesis, the AWS CDK produces a [token](tokens.md) that is resolved by AWS CloudFormation during deployment\.

The AWS CDK supports retrieving both plain and secure values\. You may request a specific version of either kind of value\. For plain values, you may omit the version from your request to retrieve the latest version\. For secure values, you must specify the version when requesting the value of the secure attribute\.

**Note**  
This topic shows how to read attributes from the AWS Systems Manager Parameter Store\. You can also read secrets from the AWS Secrets Manager \(see [Get a value from AWS Secrets Manager](get_secrets_manager_value.md)\)\.

**Topics**
+ [Read Systems Manager values at deployment time](#ssm_read_at_deploy)
+ [Read Systems Manager values at synthesis time](#ssm_read_at_synth)
+ [Write values to Systems Manager](#ssm_write)

## Read Systems Manager values at deployment time<a name="ssm_read_at_deploy"></a>

To read values from the Systems Manager Parameter Store, use the [valueForStringParameter](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ssm.StringParameter.html#static-valuewbrforwbrstringwbrparameterscope-parametername-version) and [valueForSecureStringParameter](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ssm.StringParameter.html#static-valuewbrforwbrsecurewbrstringwbrparameterscope-parametername-version) methods\. Choose a method based on whether the attribute you want is a plain string or a secure string value\. These methods return [tokens](tokens.md), not the actual value\. The value is resolved by AWS CloudFormation during deployment\. The following is an example:

------
#### [ TypeScript ]

```
import * as ssm from 'aws-cdk-lib/aws-ssm';

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
const ssm = require('aws-cdk-lib/aws-ssm');

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
    this, "my-plain-parameter-name");      // latest version
var versionOfStringToken = StringParameter.ValueForStringParameter(
    this, "my-plain-parameter-name", 1);   // version 1

// Get specified version of secure string attribute
var secureStringToken = StringParameter.ValueForSecureStringParameter(
    this, "my-secure-parameter-name", 1);   // must specify version
```

------

A [limited number of AWS services](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references.html#template-parameters-dynamic-patterns-resources) currently support this feature\.

## Read Systems Manager values at synthesis time<a name="ssm_read_at_synth"></a>

At times, it's useful to provide a parameter at synthesis time\. By doing this, the AWS CloudFormation template will always use the same value instead of resolving the value during deployment\.

To read a value from the Systems Manager Parameter Store at synthesis time, use the [valueFromLookup](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ssm.StringParameter.html#static-valuewbrfromwbrlookupscope-parametername) method \(Python: `value_from_lookup`\)\. This method returns the actual value of the parameter as a [Runtime context](context.md) value\. If the value is not already cached in `cdk.json` or passed on the command line, it is retrieved from the current AWS account\. For this reason, the stack *must* be synthesized with explicit AWS environment information\.

The following is an example:

------
#### [ TypeScript ]

```
import * as ssm from 'aws-cdk-lib/aws-ssm';

const stringValue = ssm.StringParameter.valueFromLookup(this, 'my-plain-parameter-name');
```

------
#### [ JavaScript ]

```
const ssm = require('aws-cdk-lib/aws-ssm');

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

Only plain Systems Manager strings may be retrieved\. Secure strings cannot be retrieved\. The latest version will always be returned\. Specific versions cannot be requested\.

**Important**  
The retrieved value will end up in your synthesized AWS CloudFormation template\. This might be a security risk, depending on who has access to your AWS CloudFormation templates and what kind of value it is\. Generally, don't use this feature for passwords, keys, or other values you want to keep private\.

## Write values to Systems Manager<a name="ssm_write"></a>

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