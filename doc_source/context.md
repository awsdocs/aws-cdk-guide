--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Run\-Time Context<a name="context"></a>

The AWS CDK uses context to retrieve information such as the Availability Zones in your account or Amazon Machine Image \(AMI\) IDs used to start your instances\. To avoid unexpected changes to your deployments, such as when a new Amazon Linux AMI is released, thus changing your Auto Scaling group, the AWS CDK stores context values in `cdk.context.json`\. This ensures that the AWS CDK retrieves the same value the next time it synthesises your app\. Don't forget to put this file under version control\.

## Viewing and Managing Context<a name="context_viewing"></a>

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

## Context Providers<a name="context_providers"></a>

The AWS CDK currently supports the following context providers\.

[AvailabilityZoneProvider](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/availabilityzoneprovider.html)  
Use this provider to get the list of all supported Availability Zones in this context, as shown in the following example\.  

```
// "this" refers to a Construct scope
const zones: string[] = Context.getAvailabilityZones();

for (let zone of zones) {
  // Do something for each zone!
}
```

[HostedZoneProvider](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-route53/hostedzoneprovider.html)  
Use this provider to discover existing hosted zones in your account\. For example, the following code imports an existing hosted zone into your AWS CDK app so you can add records to it\.  

```
const zone: HostedZoneRef = new HostedZoneProvider(this, {
  domainName: 'test.com'
}).findAndImport(this, 'HostedZone');
```

[SSMParameterProvider](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/ssmparameterprovider.html)  
Use this provider to read values from the current Region's AWS Systems Manager parameter store\. For example, the following code returns the value of the *my\-awesome\-parameter* key\.  

```
const ami: string = Context.getSsmParameter(this, {
  parameterName: 'my-awesome-parameter'
).parameterValue();
```
This is only for reading plain strings, and not recommended for secrets\. For reading secure strings from the Systems Manager Parameter Store, see [Get a Value from a Systems Manager Parameter Store Variable](get_ssm_value.md)\.

[VpcNetworkProvider](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-ec2/vpcnetworkprovider.html)  
Use this provider to look up and reference existing VPCs in your accounts\. For example, the following code imports a VPC by tag name\.  

```
const provider = new VpcNetworkProvider(this, {
  tags: {
    "tag:Purpose": 'WebServices'
  }
});

const vpc = Vpc.import(this, 'VPC', provider.vpcProps);
```