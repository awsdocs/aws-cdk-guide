# Stacks<a name="stacks"></a>

The unit of deployment in the AWS CDK is called a *stack*\. All AWS resources defined within the scope of a stack, either directly or indirectly, are provisioned as a single unit\.

Because AWS CDK stacks are implemented through AWS CloudFormation stacks, they have the same limitations as in [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)\.

You can define any number of stacks in your AWS CDK app\. Any instance of the `Stack` construct represents a stack, and can be either defined directly within the scope of the app, like the `MyFirstStack` example shown previously, or indirectly by any construct within the tree\.

For example, the following code defines an AWS CDK app with two stacks\.

------
#### [ TypeScript ]

```
const app = new App();

new MyFirstStack(app, 'stack1');
new MySecondStack(app, 'stack2');

app.synth();
```

------
#### [ JavaScript ]

```
const app = new App();

new MyFirstStack(app, 'stack1');
new MySecondStack(app, 'stack2');

app.synth();
```

------
#### [ Python ]

```
app = App()

MyFirstStack(app, 'stack1')
MySecondStack(app, 'stack2')

app.synth()
```

------
#### [ Java ]

```
App app = new App();

new MyFirstStack(app, "stack1");
new MySecondStack(app, "stack2");

app.synth();
```

------
#### [ C\# ]

```
var app = new App();

new MyFirstStack(app, "stack1");
new MySecondStack(app, "stack2");

app.Synth();
```

------

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

Like any other construct, stacks can be composed together into groups\. The following code shows an example of a service that consists of three stacks: a control plane, a data plane, and monitoring stacks\. The service construct is defined twice: once for the beta environment and once for the production environment\.

------
#### [ TypeScript ]

```
import { App, Construct, Stack } from "@aws-cdk/core";

interface EnvProps {
  prod: boolean;
}

// imagine these stacks declare a bunch of related resources
class ControlPlane extends Stack {}
class DataPlane extends Stack {}
class Monitoring extends Stack {}

class MyService extends Construct {

  constructor(scope: Construct, id: string, props?: EnvProps) {
  
    super(scope, id);
  
    // we might use the prod argument to change how the service is configured
    new ControlPlane(this, "cp");
    new DataPlane(this, "data");
    new Monitoring(this, "mon");  }
}

const app = new App();
new MyService(app, "beta");
new MyService(app, "prod", { prod: true });

app.synth();
```

------
#### [ JavaScript ]

```
const { App , Construct , Stack } = require("@aws-cdk/core");

// imagine these stacks declare a bunch of related resources
class ControlPlane extends Stack {}
class DataPlane extends Stack {}
class Monitoring extends Stack {}

class MyService extends Construct {

  constructor(scope, id, props) {
  
    super(scope, id);
  
    // we might use the prod argument to change how the service is configured
    new ControlPlane(this, "cp");
    new DataPlane(this, "data");
    new Monitoring(this, "mon");
  }
}

const app = new App();
new MyService(app, "beta");
new MyService(app, "prod", { prod: true });

app.synth();
```

------
#### [ Python ]

```
from aws_cdk.core import App, Construct, Stack

# imagine these stacks declare a bunch of related resources
class ControlPlane(Stack): pass
class DataPlane(Stack): pass
class Monitoring(Stack): pass

class MyService(Construct):

  def __init__(self, scope: Construct, id: str, *, prod=False):
  
    super().__init__(scope, id)
  
    # we might use the prod argument to change how the service is configured
    ControlPlane(self, "cp")
    DataPlane(self, "data")
    Monitoring(self, "mon")
    
app = App();
MyService(app, "beta")
MyService(app, "prod", prod=True)

app.synth()
```

------
#### [ Java ]

```
package com.myorg;

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.Construct;

public class MyApp {

    // imagine these stacks declare a bunch of related resources
    static class ControlPlane extends Stack {
        ControlPlane(Construct scope, String id) {
            super(scope, id);
        }
    }

    static class DataPlane extends Stack {
        DataPlane(Construct scope, String id) {
            super(scope, id);
        }
    }

    static class Monitoring extends Stack {
        Monitoring(Construct scope, String id) {
            super(scope, id);
        }
    }

    static class MyService extends Construct {
        MyService(Construct scope, String id) {
            this(scope, id, false);
        }
        
        MyService(Construct scope, String id, boolean prod) {
            super(scope, id);
            
            // we might use the prod argument to change how the service is configured
            new ControlPlane(this, "cp");
            new DataPlane(this, "data");
            new Monitoring(this, "mon");         
        }
    }
    
    public static void main(final String argv[]) {
        App app = new App();

        new MyService(app, "beta");
        new MyService(app, "prod", true);
        
        app.synth();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;

// imagine these stacks declare a bunch of related resources
public class ControlPlane : Stack {
    public ControlPlane(Construct scope, string id=null) : base(scope, id) { }
}

public class DataPlane : Stack {
    public DataPlane(Construct scope, string id=null) : base(scope, id) { }
}

public class Monitoring : Stack
{
    public Monitoring(Construct scope, string id=null) : base(scope, id) { }
}

public class MyService : Construct
{
    public MyService(Construct scope, string id, Boolean prod=false) : base(scope, id)
    {
        // we might use the prod argument to change how the service is configured
        new ControlPlane(this, "cp");
        new DataPlane(this, "data");
        new Monitoring(this, "mon");
    }
}

class Program
{
    static void Main(string[] args)
    {

        var app = new App();
        new MyService(app, "beta");
        new MyService(app, "prod", prod: true);
        app.Synth();
    }
}
```

------

This AWS CDK app eventually consists of six stacks, three for each environment:

```
$ cdk ls
    
betacpDA8372D3
betadataE23DB2BA
betamon632BD457
prodcp187264CE
proddataF7378CE5
prodmon631A1083
```

The physical names of the AWS CloudFormation stacks are automatically determined by the AWS CDK based on the stack's construct path in the tree\. By default, a stack's name is derived from the construct ID of the `Stack` object, but you can specify an explicit name using the `stackName` prop \(in Python, `stack_name`\), as follows\.

------
#### [ TypeScript ]

```
new MyStack(this, 'not:a:stack:name', { stackName: 'this-is-stack-name' });
```

------
#### [ JavaScript ]

```
new MyStack(this, 'not:a:stack:name', { stackName: 'this-is-stack-name' });
```

------
#### [ Python ]

```
MyStack(self, "not:a:stack:name", stack_name="this-is-stack-name")
```

------
#### [ Java ]

```
new MyStack(this, "not:a:stack:name", StackProps.builder()
    .StackName("this-is-stack-name").build());
```

------
#### [ C\# ]

```
new MyStack(this, "not:a:stack:name", new StackProps
{
    StackName = "this-is-stack-name"
});
```

------

## Stack API<a name="stack_api"></a>

The [Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/core/stack.html) object provides a rich API, including the following:
+ `Stack.of(construct)` – A static method that returns the **Stack** in which a construct is defined\. This is useful if you need to interact with a stack from within a reusable construct\. The call fails if a stack cannot be found in scope\.
+ `stack.stackName` \(Python: `stack_name`\) – Returns the physical name of the stack\. As mentioned previously, all AWS CDK stacks have a physical name that the AWS CDK can resolve during synthesis\.
+ `stack.region` and `stack.account` – Return the AWS Region and account, respectively, into which this stack will be deployed\. These properties return either the account or Region explicitly specified when the stack was defined, or a string\-encoded token that resolves to the AWS CloudFormation pseudo\-parameters for account and Region to indicate that this stack is environment agnostic\. See [Environments](environments.md) for information about how environments are determined for stacks\.
+ `stack.addDependency(stack)` \(Python: `stack.add_dependency(stack)` – Can be used to explicitly define dependency order between two stacks\. This order is respected by the cdk deploy command when deploying multiple stacks at once\.
+ `stack.tags` – Returns a [TagManager](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.TagManager.html) that you can use to add or remove stack\-level tags\. This tag manager tags all resources within the stack, and also tags the stack itself when it's created through AWS CloudFormation\.
+ `stack.partition`, `stack.urlSuffix` \(Python: `url_suffix`\), `stack.stackId` \(Python: `stack_id`\), and `stack.notificationArn` \(Python: `notification_arn`\) – Return tokens that resolve to the respective AWS CloudFormation pseudo\-parameters, such as `{ "Ref": "AWS::Partition" }`\. These tokens are associated with the specific stack object so that the AWS CDK framework can identify cross\-stack references\.
+ `stack.availabilityZones` \(Python: `availability_zones`\) – Returns the set of Availability Zones available in the environment in which this stack is deployed\. For environment\-agnostic stacks, this always returns an array with two Availability Zones, but for environment\-specific stacks, the AWS CDK queries the environment and returns the exact set of Availability Zones available in the region you specified\.
+ `stack.parseArn(arn)` and `stack.formatArn(comps)` \(Python: `parse_arn`, `format_arn`\) – Can be used to work with Amazon Resource Names \(ARNs\)\.
+ `stack.toJsonString(obj)` \(Python: `to_json_string`\) – Can be used to format an arbitrary object as a JSON string that can be embedded in an AWS CloudFormation template\. The object can include tokens, attributes, and references, which are only resolved during deployment\.
+ `stack.templateOptions` \(Python: `template_options`\) – Enables you to specify AWS CloudFormation template options, such as Transform, Description, and Metadata, for your stack\.

## Nested stacks<a name="stack_nesting"></a>

The [NestedStack](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-cloudformation.NestedStack.html) construct offers a way around the AWS CloudFormation 500\-resource limit for stacks\. A nested stack counts as only one resource in the stack that contains it, but can itself contain up to 500 resources, including additional nested stacks\.

The scope of a nested stack must be a `Stack` or `NestedStack` construct\. The nested stack needn't be declared lexically inside its parent stack; it is necessary only to pass the parent stack as the first parameter \(`scope`\) when instantiating the nested stack\. Aside from this restriction, defining constructs in a nested stack works exactly the same as in an ordinary stack\.

At synthesis time, the nested stack is synthesized to its own AWS CloudFormation template, which is uploaded to the AWS CDK staging bucket at deployment\. Nested stacks are bound to their parent stack and are not treated as independent deployment artifacts; they are not listed by `cdk list` nor can they be deployed by `cdk deploy`\.

References between parent stacks and nested stacks are automatically translated to stack parameters and outputs in the generated AWS CloudFormation templates, as with any [cross\-stack reference](resources.md#resource_stack)\.

**Warning**  
Changes in security posture are not displayed before deployment for nested stacks\. This information is displayed only for top\-level stacks\.