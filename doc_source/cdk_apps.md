--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Apps<a name="cdk_apps"></a>

The main artifact of an AWS CDK program is called a *CDK App*\. This is an executable program that can be used to synthesize deployment artifacts that can be deployed by supporting tools like the AWS CDK Toolkit, which are described in [Command\-line Toolkit \(cdk\)](cdk_tools.md)\.

Tools interact with apps through the program's **argv**/**stdout** interface, which can be easily implemented using the **App** class, as shown in the following example\.

```
import { App } from '@aws-cdk/cdk'

const app = new App(); // input: ARGV

// <add stacks here>

app.run();
```

An [App](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#app) is a collection of [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) objects, as shown in the following example\.

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

Use the cdk deploy command to deploy an individual stack:

```
cdk deploy Dev
```