# Runtime context<a name="context"></a>

Context values are key\-value pairs that can be associated with a stack or construct\. The AWS CDK uses context to cache information from your AWS account, such as the Availability Zones in your account or the Amazon Machine Image \(AMI\) IDs used to start your instances\. [Feature flags](featureflags.md) are also context values\. You can create your own context values for use by your apps or constructs\.

Context keys and values are strings\. If you want to pass other types of value, such as numbers but also including structured data such as JSON, it must be passed as a string\. Code that consumes such a context value will need to convert or parse the data as appropriate\.

## Construct context<a name="context_construct"></a>

Context values are made available to your AWS CDK app in six different ways:
+ Automatically from the current AWS account\.
+ Through the \-\-context option to the cdk command\.
+ In the project's `cdk.context.json` file\.
+ In the project's `cdk.json` file\.
+ In the `context` key of your `~/.cdk.json` file\.
+ In your AWS CDK app using the `construct.node.setContext` method\.

The project file `cdk.context.json` is where the AWS CDK caches context values retrieved from your AWS account\. This practice avoids unexpected changes to your deployments when, for example, a new Amazon Linux AMI is released, changing your Auto Scaling group\. The AWS CDK does not write context data to any of the other files listed\. 

We recommend that your project's context files be placed under version control along with the rest of your application, as the information in them is part of your app's state and is critical to being able to synthesize and deploy consistently\.

Context values are scoped to the construct that created them; they are visible to child constructs, but not to siblings\. Context values set by the AWS CDK Toolkit \(the cdk command\), whether automatically, from a file, or from the \-\-context option, are implicitly set on the `App` construct, and so are visible to every construct in the app\.

You can get a context value using the `construct.node.tryGetContext` method\. If the requested entry is not found on the current construct or any of its parents, the result is `undefined` \(or your language's equivalent, such as `None` in Python\)\.

## Context methods<a name="context_methods"></a>

The AWS CDK supports several context methods that enable AWS CDK apps to get contextual information\. For example, you can get a list of Availability Zones that are available in a given AWS account and AWS Region, using the [stack\.availabilityZones](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html#availabilityzones) method\.

The following are the context methods:

[HostedZone\.fromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-route53.HostedZone.html#static-from-wbr-lookupscope-id-query)  
Gets the hosted zones in your account\.

[stack\.availabilityZones](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html#availabilityzones)  
Gets the supported Availability Zones\.

[StringParameter\.valueFromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ssm.StringParameter.html#static-value-wbr-from-wbr-lookupscope-parametername)  
Gets a value from the current Region's Amazon EC2 Systems Manager Parameter Store\.

[Vpc\.fromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ec2.Vpc.html#static-from-wbr-lookupscope-id-options)  
Gets the existing Amazon Virtual Private Clouds in your accounts\.

[LookupMachineImage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ec2.LookupMachineImage.html)  
Looks up a machine image for use with a NAT instance in an Amazon Virtual Private Cloud\.

If a given context information isn't available, the AWS CDK app notifies the AWS CDK CLI that the context information is missing\. The CLI then queries the current AWS account for the information, stores the resulting context information in the `cdk.context.json` file, and executes the AWS CDK app again with the context values\.

Don't forget to add the `cdk.context.json` file to your source control repository to ensure that subsequent synth commands will return the same result, and that your AWS account won't be needed when synthesizing from your build system\.

## Viewing and managing context<a name="context_viewing"></a>

Use the cdk context command to view and manage the information in your `cdk.context.json` file\. To see this information, use the cdk context command without any options\. The output should be something like the following\.

```
Context found in cdk.json:

┌───┬─────────────────────────────────────────────────────────────┬─────────────────────────────────────────────────────────┐
│ # │ Key                                                         │ Value                                                   │
├───┼─────────────────────────────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 1 │ availability-zones:account=123456789012:region=eu-central-1 │ [ "eu-central-1a", "eu-central-1b", "eu-central-1c" ]   │
├───┼─────────────────────────────────────────────────────────────┼─────────────────────────────────────────────────────────┤
│ 2 │ availability-zones:account=123456789012:region=eu-west-1    │ [ "eu-west-1a", "eu-west-1b", "eu-west-1c" ]            │
└───┴─────────────────────────────────────────────────────────────┴─────────────────────────────────────────────────────────┘

Run cdk context --reset KEY_OR_NUMBER to remove a context key. If it is a cached value, it will be refreshed on the next cdk synth.
```

To remove a context value, run cdk context \-\-reset, specifying the value's corresponding key or number\. The following example removes the value that corresponds to the second key in the preceding example, which is the list of availability zones in the Ireland region\.

```
cdk context --reset 2
```

```
Context value
availability-zones:account=123456789012:region=eu-west-1
reset. It will be refreshed on the next SDK synthesis run.
```

Therefore, if you want to update to the latest version of the Amazon Linux AMI, you can use the preceding example to do a controlled update of the context value and reset it, and then synthesize and deploy your app again\.

```
cdk synth
```

To clear all of the stored context values for your app, run cdk context \-\-clear, as follows\.

```
cdk context --clear
```

Only context values stored in `cdk.context.json` can be reset or cleared\. The AWS CDK does not touch other context files\. To protect a context value from being reset using these commands, then, you might copy the value to `cdk.json`\.

## AWS CDK Toolkit `--context` flag<a name="context_cli"></a>

Use the `--context` \(`-c` for short\) option to pass runtime context values to your CDK app during synthesis or deployment\.

```
cdk synth --context key=value MyStack
```

To specify multiple context values, repeat the \-\-context option any number of times, providing one key\-value pair each time\.

```
cdk synth --context key1=value1 --context key2=value2 MyStack
```

When deploying multiple stacks, the specified context values are normally passed to all of them\. If you wish, you may specify different values for each stack by prefixing the stack name to the context value\.

```
cdk synth --context Stack1:key=value --context Stack2:key=value Stack1 Stack2
```

## Example<a name="context_example"></a>

Below is an example of importing an existing Amazon VPC using AWS CDK context\.

------
#### [ TypeScript ]

```
import * as cdk from '@aws-cdk/core';
import * as ec2 from '@aws-cdk/aws-ec2';

export class ExistsVpcStack extends cdk.Stack {

  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
  
    super(scope, id, props);
    
    const vpcid = this.node.tryGetContext('vpcid');
    const vpc = ec2.Vpc.fromLookup(this, 'VPC', {
      vpcId: vpcid,
    });
    
    const pubsubnets = vpc.selectSubnets({subnetType: ec2.SubnetType.PUBLIC});
    
    new cdk.CfnOutput(this, 'publicsubnets', {
      value: pubsubnets.subnetIds.toString(),
    });
  }
}
```

------
#### [ JavaScript ]

```
const cdk = require('@aws-cdk/core');
const ec2 = require('@aws-cdk/aws-ec2');

class ExistsVpcStack extends cdk.Stack {

  constructor(scope, id, props) {
  
    super(scope, id, props);
    
    const vpcid = this.node.tryGetContext('vpcid');
    const vpc = ec2.Vpc.fromLookup(this, 'VPC', {
      vpcId: vpcid
    });
    
    const pubsubnets = vpc.selectSubnets({subnetType: ec2.SubnetType.PUBLIC});
    
    new cdk.CfnOutput(this, 'publicsubnets', {
      value: pubsubnets.subnetIds.toString()
    });
  }
}

module.exports = { ExistsVpcStack }
```

------
#### [ Python ]

```
import aws_cdk.core as cdk
import aws_cdk.aws_ec2 as ec2

class ExistsVpcStack(cdk.Stack):

    def __init__(scope: cdk.Construct, id: str, **kwargs):
  
        super().__init__(scope, id, **kwargs)
    
        vpcid = self.node.try_get_context("vpcid");
        vpc = ec2.Vpc.from_lookup(self, "VPC", vpc_id=vpcid)
    
        pubsubnets = vpc.select_subnets(subnetType=ec2.SubnetType.PUBLIC);
    
        cdk.CfnOutput(self, "publicsubnets",
            value=pubsubnets.subnet_ids.to_string())
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.CfnOutput;

import software.amazon.awscdk.services.ec2.Vpc;
import software.amazon.awscdk.services.ec2.VpcLookupOptions;
import software.amazon.awscdk.services.ec2.SelectedSubnets;
import software.amazon.awscdk.services.ec2.SubnetSelection;
import software.amazon.awscdk.services.ec2.SubnetType;

public class ExistsVpcStack extends Stack {
    public ExistsVpcStack(App context, String id) {
        this(context, id, null);
    }
    
    public ExistsVpcStack(App context, String id, StackProps props) {
        super(context, id, props);
        
        String vpcId = (String)this.getNode().tryGetContext("vpcid");
        Vpc vpc = (Vpc)Vpc.fromLookup(this, "VPC", VpcLookupOptions.builder()
                .vpcId(vpcId).build());

        SelectedSubnets pubSubNets = vpc.selectSubnets(SubnetSelection.builder()
                .subnetType(SubnetType.PUBLIC).build());
        
        CfnOutput.Builder.create(this, "publicsubnets")
                .value(pubSubNets.getSubnetIds().toString()).build();                
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.EC2;

class ExistsVpcStack : Stack
{
    public ExistsVpcStack(App scope, string id, StackProps props) : base(scope, id, props)
    {
        var vpcId = (string)this.Node.TryGetContext("vpcid");
        var vpc = Vpc.FromLookup(this, "VPC", new VpcLookupOptions
        {
            VpcId = vpcId
        });

        SelectedSubnets pubSubNets = vpc.SelectSubnets([new SubnetSelection
        {
            SubnetType = SubnetType.PUBLIC
        }]);

        new CfnOutput(this, "publicsubnets", new CfnOutputProps {
            Value = pubSubNets.SubnetIds.ToString()
        });
    }
}
```

------

You can use cdk diff to see the effects of passing in a context value on the command line:

```
cdk diff -c vpcid=vpc-0cb9c31031d0d3e22
```

```
Stack ExistsvpcStack
Outputs
[+] Output publicsubnets publicsubnets: {"Value":"subnet-06e0ea7dd302d3e8f,subnet-01fc0acfb58f3128f"}
```

The resulting context values can be viewed as shown here\.

```
cdk context -j
```

```
{
  "vpc-provider:account=123456789012:filter.vpc-id=vpc-0cb9c31031d0d3e22:region=us-east-1": {
    "vpcId": "vpc-0cb9c31031d0d3e22",
    "availabilityZones": [
      "us-east-1a",
      "us-east-1b"
    ],
    "privateSubnetIds": [
      "subnet-03ecfc033225be285",
      "subnet-0cded5da53180ebfa"
    ],
    "privateSubnetNames": [
      "Private"
    ],
    "privateSubnetRouteTableIds": [
      "rtb-0e955393ced0ada04",
      "rtb-05602e7b9f310e5b0"
    ],
    "publicSubnetIds": [
      "subnet-06e0ea7dd302d3e8f",
      "subnet-01fc0acfb58f3128f"
    ],
    "publicSubnetNames": [
      "Public"
    ],
    "publicSubnetRouteTableIds": [
      "rtb-00d1fdfd823c82289",
      "rtb-04bb1969b42969bcb"
    ]
  }
}
```