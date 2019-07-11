# Stacks<a name="stacks"></a>

The unit of deployment in the AWS CDK is called a *stack*\. All AWS resources defined within the scope of a stack, either directly or indirectly, are provisioned as a single unit\.

Because AWS CDK stacks are implemented through AWS CloudFormation stacks, they have the same limitations as in [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)\.

You can define any number of stacks in your AWS CDK app\. Any instance of the `Stack` construct represents a stack, and can be either defined directly within the scope of the app, like the `MyFirstStack` example shown previously, or indirectly by any construct within the tree\.

For example, the following code defines an AWS CDK app with two stacks\.

```
const app = new App();

new MyFirstStack(app, 'stack1');
new MySecondStack(app, 'stack2');

app.run();
```

To list all the stacks in an AWS CDK app, run the cdk ls command, which for the previous AWS CDK app would have the following output\.

```
stack1
stack2
```

When you run the cdk synth command for an app with multiple stacks, the cloud assembly includes a separate template for each stack instance\. Even if the two stacks are instances of the same class, the AWS CDK emits them as two individual templates\.

You can synthesize each template by specifying the stack name in the cdk synth command\. The following example synthesizes the template for **stack1**\.

```
cdk synth stack1
```

This approach is conceptually different from how AWS CloudFormation templates are normally used, where a template can be deployed multiple times and parameterized through [AWS CloudFormation parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)\. Although AWS CloudFormation parameters can be defined in the AWS CDK, they are generally discouraged because AWS CloudFormation parameters are resolved only during deployment\. This means that you cannot determine their value in your code\. For example, to conditionally include a resource in your app based on the value of a parameter, you must set up an [AWS CloudFormation condition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html) and tag the resource with this condition\. Because the AWS CDK takes an approach where concrete templates are resolved at synthesis time, you can use an **if** statement to check the value to determine whether a resource should be defined or some behavior should be applied\.

**Note**  
The AWS CDK provides as much resolution as possible during synthesis time to enable idiomatic and natural usage of your programming language\.

Like any other construct, stacks can be composed together into groups\. The following pseudocode shows an example of a service that consists of three stacks: a control plane, a data plane, and monitoring stacks\. The service construct is defined twice: once for the beta environment and once for the production environment\.

```
class ControlPlane extends Stack { ... }
class DataPlane extends Stack { ... }
class Monitoring extends Stack { ... }

class MyService extends Construct {
  constructor(...) {
    new ControlPlane(this, ...);
    new DataPlane(this, ...);
    new Monitoring(this, ...);
  }
}

const app = new App();
new MyService(app, 'beta');
new MyService(app, 'prod', { prod: true });
app.run();
```

This AWS CDK app eventually consists of six stacks, three for each environment\.

The physical names of the AWS CloudFormation stacks are automatically determined by the AWS CDK based on the stack's construct path in the tree\. By default, a stack's name is derived from the construct ID of the `Stack` object, but you can specify an explicit name using the `stackName` prop, as follows\.

```
new MyStack(this, 'not:a:stack:name', { stackName: 'this-is-stack-name' });
```

## Stack API<a name="stack_api"></a>

The [Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/stack.html) object provides a rich API, including the following:
+ `Stack.of(construct)` – A static method that returns the **Stack** in which a construct is defined\. This is useful if you need to interact with a stack from within a reusable construct\. The call fails if a stack cannot be found in scope\.
+ `stack.stackName` – Returns the physical name of the stack\. As mentioned previously, all AWS CDK stacks have a physical name that the AWS CDK can resolve during synthesis\.
+ `stack.region` and `stack.account` – Return the AWS Region and account, respectively, into which this stack will be deployed\. These properties return either the account or Region explicitly specified when the stack was defined, or a string\-encoded token that resolves to the AWS CloudFormation pseudo\-parameters for account and Region to indicate that this stack is environment agnostic\. See [Environments](environments.md) for information about how environments are determined for stacks\.
+ `stack.account` – Returns the AWS account into which this stack will be deployed\. Similarly to Region, this property returns either an explicit account ID or a token that resolves to \{ Ref: AWS::AccountId \}\. Use `Token.isUnresolved` method to determine whether the value is a token before reading the value returned from this property\.
+ `stack.addDependency(stack)` – Can be used to explicitly define dependency order between two stacks\. This order is respected by the cdk deploy command when deploying multiple stacks at once\.
+ `stack.tags` – Returns a [TagManager](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/tagmanager.html#core_TagManager) that you can use to add or remove stack\-level tags\. This tag manager tags all resources within the stack, and also tags the stack itself when it’s created through AWS CloudFormation\.
+ `stack.partition`, `stack.urlSuffix`, `stack.stackId`, and `stack.notificationArn` – Return tokens that resolve to the respective AWS CloudFormation pseudo\-parameters, such as \{ "Ref": "AWS::Partition" \}\. These tokens are associated with this specific stack object, so the framework can identify cross\-stack references\.
+ `stack.availabilityZones` – Returns the set of Availability Zones available in the environment in which this stack is deployed\. For environment\-agnostic stacks, this always returns an array with two Availability Zones, but for environment\-specific stacks, the AWS CDK queries the environment and returns the exact set of Availability Zones that are available\.
+ `stack.parseArn(arn)` and `stack.formatArn(comps)` – Can be used to work with Amazon Resource Names \(ARNs\)\.
+ `stack.toJsonString(obj)` – Can be used to format an arbitrary object as a JSON string that can be embedded in an AWS CloudFormation template\. The object can include tokens, attributes, and references, which are only resolved during deployment\.
+ `stack.templateOptions` – Enables you to specify AWS CloudFormation template options, such as Transform, `Description`, and Metadata, for your stack\.