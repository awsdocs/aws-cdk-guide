--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Stacks<a name="cdk_stacks"></a>

A [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) is an AWS CDK construct that can be deployed into an AWS environment\. The combination of region and account becomes the stack's *environment*, as described in [Environments and Authentication](cdk_environments.md)\. Most production apps consist of multiple stacks of resources that are deployed as a single transaction using a resource provisioning service like AWS CloudFormation\. Any resources added directly or indirectly as children of a stack are included in the stack's template as it is synthesized by your AWS CDK program\.

Define an application stack by extending the [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) class, as shown in the following example\.

```
import { Stack, StackProps } from '@aws-cdk/cdk'

interface MyStackProps extends StackProps {
    encryptedStorage: boolean;
}

class MyStack extends Stack {
    constructor(parent: Construct, name: string, props?: MyStackProps) {
        super(parent, name, props);

        new MyStorageLayer(this, 'Storage', { encryptedStorage: props.encryptedStorage });
        new MyControlPlane(this, 'CPlane');
        new MyDataPlane(this, 'DPlane');
    }
}
```

And then, add instances of this class to your app:

```
const app = new App();

new MyStack(app, 'NorthAmerica', { env: { region: 'us-east-1' } });
new MyStack(app, 'Europe', { env: { region: 'us-west-2' } });
```