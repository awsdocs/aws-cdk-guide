--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Environmental Context<a name="cdk_context"></a>

When you synthesize a stack to create a AWS CloudFormation template, the AWS CDK might need information based on the environment \(account and Region\), such as the availability zones or AMIs available in the Region\. To enable this feature, the AWS CDK Toolkit uses *context providers*, and saves the context information into `cdk.json` the first time you call `cdk synth`\.

The AWS CDK currently supports the following context providers\.

[AvailabilityZoneProvider](@cdk-class-url;#@aws-cdk/cdk.AvailabilityZoneProvider)   
Use this provider to get the list of all supported availability zones in this environment\. For example, the following code iterates over all of the AZs in the current environment\.  

```
// "this" refers to a parent Construct
const zones: string[] = new AvailabilityZoneProvider(this).availabilityZones;

for (let zone of zones) {
  // do somethning for each zone!
}
```

[SSMParameterProvider](@cdk-class-url;#@aws-cdk/cdk.SSMParameterProvider)  
Use this provider to read values from the current Region's SSM parameter store\. For example, the follow code returns the value of the *my\-awesome\-parameter* key:  

```
const ami: string = new SSMParameterProvider(this, {
  parameterName: 'my-awesome-parameter'
).parameterValue();
```
This is only for reading plain strings, and not recommended for secrets\. For reading secure strings from SSM Parmeter store, see [Getting a Value from an SSM Store Variable](cdk_passing_in_data.md#cdk_passing_ssm_value)\.\.

[HostedZoneProvider](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-route53.html#@aws-cdk/aws-route53.HostedZoneProvider)  
Use this provider to discover existing hosted zones in your account\. For example, the following code imports an existing hosted zone into your CDK app so you can add records to it:  

```
const zone: HostedZoneRef = new HostedZoneProvider(this, {
  domainName: 'test.com'
}).findAndImport(this, 'HostedZone');
```

[VpcNetworkProvider](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_aws-ec2.html#@aws-cdk/aws-ec2.VpcNetworkProvider)  
Use this provider to look up and reference existing VPC in your accounts\. For example, the follow code imports a VPC by tag name:  

```
const provider = new VpcNetworkProvider(this, {
  tags: {
    Purpose: 'WebServices'
  }
});

const vpc = VpcNetworkRef.import(this, 'VPC', provider.vpcProps);
```

## Viewing and managing context<a name="cdk_context_viewing"></a>

Context is used to retrieve information such as the Availability Zones in your account or AMI IDs used to start your instances\. To avoid unexpected changes to your deployments, such as when you add a `Queue` to your application, but a new Amazon Linux AMI was released, thus changing your AutoScalingGroup, the AWS CDK stores the context values in `cdk.json`\. This ensures that the AWS CDK retrieves the same value on the next synthesis\.

Use the cdk context to see context values stored for your application\. The output should be something like the following:

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

If at some point you want to update to the latest version of the Amazon Linux AMI, do a controlled update of the context value, reset it, and synthesize again:

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

To clear all context values, run cdk context \-\-clear:

```
cdk context --clear
```