--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Apps and Stacks<a name="apps_and_stacks"></a>

The main artifact of a CDK program known as a *CDK app*\. This is an executable program that you can use to synthesize deployment artifacts that supporting tools, such as the CDK Toolkit, can deploy, as described in [AWS CDK Command Line Toolkit \(cdk\)](tools.md)\.

Stacks are CDK constructs that you can deploy into an AWS environment\. The combination of AWS Region and account becomes the stack's *environment*, as described in [Environments and Authentication](environments_and_context.md#environments)\. Most production apps consist of multiple stacks of resources that are deployed as a single transaction using a resource provisioning service such as AWS CloudFormation\. Any resources added directly or indirectly as children of a stack are included in the stack's template when it is synthesized by your CDK program\.

## Apps<a name="apps"></a>

An [app](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#app) is a collection of [stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) objects, as shown in the following example\.

```
import { App } from '@aws-cdk/cdk'
import { MyStack } from './my-stack'

const app = new App();

const dev = new MyStack(app, { name: 'Dev', region: 'us-west-2', dev: true })
const preProd = new MyStack(app, { name: 'PreProd', region: 'us-west-2', preProd: true })
const prod = [
    new MyStack(app, { name: 'NAEast', region: 'us-east-1' }),
    new MyStack(app, { name: 'NAWest', region: 'us-west-2' }),
    new MyStack(app, { name: 'EU', region: 'eu-west-1', encryptedStorage: true })
]

new DeploymentPipeline(app, {
    region: 'us-east-1',
    strategy: DeploymentStrategy.Waved,
    preProdStages: [ preProd ],
    prodStages: prod
});

app.run();
```

Use the cdk list command to list the stacks in this executable, as shown in the following example\.

```
[
   { name: "Dev", region: "us-west-2" }
   { name: "PreProd", region: "us-west-2" },
   { name: "NAEast", region: "us-east-1" },
   { name: "NAWest", region: "us-west-2" },
   { name: "EU", region: "eu-west-1" },
   { name: "DeploymentPipeline", region: 'us-east-1' }
]
```

Use the cdk deploy command to deploy an individual stack\.

```
cdk deploy Dev
```

## Stacks<a name="stacks"></a>

Define an application stack by extending the [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) class, as shown in the following example\.

```
import { Stack, StackProps } from '@aws-cdk/cdk'

interface MyStackProps extends StackProps {
    encryptedStorage: boolean;
}

class MyStack extends Stack {
    constructor(scope: Construct, id: string, props?: MyStackProps) {
        super(scope, id, props);

        new MyStorageLayer(this, 'Storage', { encryptedStorage: props.encryptedStorage });
        new MyControlPlane(this, 'CPlane');
        new MyDataPlane(this, 'DPlane');
    }
}
```

And then, add instances of this class to your app\.

```
const app = new App();

new MyStack(app, 'NorthAmerica', { env: { region: 'us-east-1' } });
new MyStack(app, 'Europe', { env: { region: 'us-west-2' } });
```

See [Stack How\-Tos](stack_how_to.md) for additional information about using stacks\.