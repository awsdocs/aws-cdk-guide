--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Environments and Run\-Time Context<a name="environments_and_context"></a>

The CDK refers to the combination of an account ID and an AWS Region as an *environment*\. The simplest environment is the one you get by default\. This is the one you get when you have set up your credentials and a default Region as described in [Specifying Your Credentials](install_config.md#credentials)\.

When you synthesize a stack to create an AWS CloudFormation template, the CDK might need information based on the run time context, such as the Availability Zones or Amazon Machine Images \(AMIs\) that are available in the Region\. To enable this feature, the CDK Toolkit uses *context providers*, and saves the context information into the `cdk.json` file the first time you call `cdk synth`\.

## Environments and Authentication<a name="environments"></a>

When you create a [stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) instance, you can supply the target deployment environment for the stack using the `env` property\. This is shown in the following example, where *REGION* is the AWS Region in which you want to create the stack and *ACCOUNT* is your account ID\.

```
new MyStack(app, { env: { region: 'REGION', account: 'ACCOUNT' } });
```

For each of the two arguments, `region` and `account`, the CDK uses the following lookup procedure:
+ If `region` or `account`is provided directly as a property to the stack, use that\.
+ If either property is not specified when you create a stack, the CDK determines them as follows:
  + `account` – Use the account from the default SDK credentials\. Environment variables are tried first \(`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`\), followed by credentials in `$HOME/.aws/credentials` on Linux or macOS, or `%USERPROFILE%\.aws\credentials` on Windows\.
  + `region` – Use the default Region configured in `$HOME/.aws/config` on Linux or macOS, or `%USERPROFILE%\.aws\config` on Windows\.

  You can set these defaults manually, but we recommend you use `aws configure`, as described in [Installing and Configuring the AWS CDK](install_config.md)\.

We recommend that you use the default environment for development stacks, and explicitly specify accounts and Regions for production stacks\.

**Note**  
Although the Region and account might explicitly be set on your stack, if you run `cdk deploy`, the CDK still uses the currently configured SDK credentials that are provided through environment variables or `aws configure`\. This means that if you want to deploy stacks to multiple accounts, you have to set the correct credentials for each invocation to `cdk deploy STACK`\.

You can always find the Region within which your stack is deployed by using the `region` property of the stack, as follows\.

```
this.region
```

## Run\-Time Context<a name="context"></a>

The CDK uses context to retrieve information such as the Availability Zones in your account or Amazon Machine Image \(AMI\) IDs used to start your instances\. To avoid unexpected changes to your deployments, such as when a new Amazon Linux AMI is released, thus changing your Auto Scaling group, the CDK stores context values in `cdk.context.json`\. This ensures that the CDK retrieves the same value the next time it synthesises your app\. Don't forget to put this file under version control\.

### Viewing and Managing Context<a name="context_viewing"></a>

Use the cdk context to see context values stored for your application\. The output should be something like the following\.

```
Context found in cdk.json:

┌───┬────────────────────────────────────────────────────┬────────────────────────────────────────────────────┐
│ # │ Key                                                │ Value                                              │
├───┼────────────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│ 1 │ availability-zones:account=123456789012:region=us- │ [ "us-east-1a", "us-east-1b", "us-east-1c",        │
│   │ east-1                                             │ "us-east-1d", "us-east-1e", "us-east-1f" ]         │
├───┼────────────────────────────────────────────────────┼────────────────────────────────────────────────────┤
│ 2 │ ssm:account=123456789012:parameterName=/aws/       │ "ami-013be31976ca2c322"                            │
│   │ service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_ │                                                    │
│   │ 64-gp2:region=us-east-1                            │                                                    │
└───┴────────────────────────────────────────────────────┴────────────────────────────────────────────────────┘

Run cdk context --reset KEY_OR_NUMBER to remove a context key. It will be refreshed on the next CDK synthesis run.
```

If at some point you want to update to the latest version of the Amazon Linux AMI, do a controlled update of the context value, reset it, and synthesize again\.

```
$ cdk context --reset 2
```

```
Context value
ssm:account=123456789012:parameterName=/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2:region=us-east-1
reset. It will be refreshed on the next SDK synthesis run.
```

```
cdk synth
```

```
...
```

To clear all context values, run cdk context \-\-clear\.

```
cdk context --clear
```

### Context Providers<a name="context_providers"></a>

The AWS CDK currently supports the following context providers\.

[AvailabilityZoneProvider](@cdk-class-url;#@aws-cdk/cdk.AvailabilityZoneProvider)  
Use this provider to get the list of all supported Availability Zones in this context, as shown in the following example\.  

```
// "this" refers to a Construct scope
const zones: string[] = new AvailabilityZoneProvider(this).availabilityZones;

for (let zone of zones) {
  // Do something for each zone!
}
```

[HostedZoneProvider](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-route53.html#@aws-cdk/aws-route53.HostedZoneProvider)  
Use this provider to discover existing hosted zones in your account\. For example, the following code imports an existing hosted zone into your CDK app so you can add records to it\.  

```
const zone: HostedZoneRef = new HostedZoneProvider(this, {
  domainName: 'test.com'
}).findAndImport(this, 'HostedZone');
```

[SSMParameterProvider](@cdk-class-url;#@aws-cdk/cdk.SSMParameterProvider)  
Use this provider to read values from the current Region's AWS Systems Manager parameter store\. For example, the following code returns the value of the *my\-awesome\-parameter* key\.  

```
const ami: string = new SSMParameterProvider(this, {
  parameterName: 'my-awesome-parameter'
).parameterValue();
```
This is only for reading plain strings, and not recommended for secrets\. For reading secure strings from the Systems Manager Parameter Store, see [Get a Value from a Systems Manager Parameter Store Variable](get_ssm_value.md)\.

[VpcNetworkProvider](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#@aws-cdk/aws-ec2.VpcNetworkProvider)  
Use this provider to look up and reference existing VPCs in your accounts\. For example, the following code imports a VPC by tag name\.  

```
const provider = new VpcNetworkProvider(this, {
  tags: {
    "tag:Purpose": 'WebServices'
  }
});

const vpc = VpcNetwork.import(this, 'VPC', provider.vpcProps);
```