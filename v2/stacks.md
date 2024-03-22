# Stacks<a name="stacks"></a>

An AWS Cloud Development Kit \(AWS CDK\) *stack* is a collection of one or more constructs, which define AWS resources\. Each CDK stack represents an AWS CloudFormation stack in your CDK app\. At deployment, constructs within a stack are provisioned as a single unit, called an AWS CloudFormation stack\. To learn more about AWS CloudFormation stacks, see [Working with stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) in the *AWS CloudFormation User Guide*\.

Since CDK stacks are implemented through AWS CloudFormation stacks, AWS CloudFormation quotas and limitations apply\. To learn more, see [AWS CloudFormation quotas](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)\.

**Topics**
+ [Defining stacks](#stacks-define)
+ [Working with stacks](#stacks-work)

## Defining stacks<a name="stacks-define"></a>

Stacks are defined within the context of an app\. You define a stack using the `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` class from the AWS Construct Library\. Stacks can be defined in any of the following ways:
+ Directly within the scope of the app\.
+ Indirectly by any construct within the tree\.

The following example defines a CDK app that contains two stacks:

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

The following example is a common pattern for defining a stack on a separate file\. Here, we extend or inherit the `Stack` class and define a constructor that accepts `scope`, `id`, and `props`\. Then, we invoke the base `Stack` class constructor using `super` with the received `scope`, `id`, and `props`\.

------
#### [ TypeScript ]

```
class HelloCdkStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

    //...
  }
}
```

------
#### [ JavaScript ]

```
class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    //...
  }
}
```

------
#### [ Python ]

```
class HelloCdkStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # ...
```

------
#### [ Java ]

```
public class HelloCdkStack extends Stack {
    public HelloCdkStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // ...
    }
}
```

------
#### [ C\# ]

```
public class HelloCdkStack : Stack
{
    public HelloCdkStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
    {
        //...
    }
}
```

------
#### [ Go ]

```
func HelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
 var sprops awscdk.StackProps
 if props != nil {
  sprops = props.StackProps
 }
 stack := awscdk.NewStack(scope, &id, &sprops)

  return stack
}
```

------

The following example declares a stack class named `MyFirstStack` that includes a single Amazon S3 bucket\.

------
#### [ TypeScript ]

```
class MyFirstStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket');
  }
}
```

------
#### [ JavaScript ]

```
class MyFirstStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket');
  }
}
```

------
#### [ Python ]

```
class MyFirstStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        s3.Bucket(self, "MyFirstBucket")
```

------
#### [ Java ]

```
public class MyFirstStack extends Stack {
    public MyFirstStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyFirstStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);
        
        new Bucket(this, "MyFirstBucket");
    }
}
```

------
#### [ C\# ]

```
public class MyFirstStack : Stack
{
    public MyFirstStack(Stack scope, string id, StackProps props = null) : base(scope, id, props)
    {
        new Bucket(this, "MyFirstBucket");
    }
}
```

------
#### [ Go ]

```
func MyFirstStack(scope constructs.Construct, id string, props *MyFirstStackProps) awscdk.Stack {
  var sprops awscdk.StackProps
  if props != nil {
    sprops = props.StackProps
  }
  stack := awscdk.NewStack(scope, &id, &sprops)
	
  s3.NewBucket(stack, jsii.String("MyFirstBucket"), &s3.BucketProps{})
  return stack
}
```

------

However, this code has only *declared* a stack\. For the stack to actually be synthesized into an AWS CloudFormation template and deployed, it must be instantiated\. And, like all CDK constructs, it must be instantiated in some context\. The `App` is that context\.

If you're using the standard AWS CDK development template, your stacks are instantiated in the same file where you instantiate the `App` object\.

------
#### [ TypeScript ]

The file named after your project \(for example, `hello-cdk.ts`\) in your project's `bin` folder\.

------
#### [ JavaScript ]

The file named after your project \(for example, `hello-cdk.js`\) in your project's `bin` folder\.

------
#### [ Python ]

The file `app.py` in your project's main directory\.

------
#### [ Java ]

The file named `ProjectNameApp.java`, for example `HelloCdkApp.java`, nested deep under the `src/main` directory\.

------
#### [ C\# ]

The file named `Program.cs` under `src\ProjectName`, for example `src\HelloCdk\Program.cs`\.

------

### The stack API<a name="stack_api"></a>

The [Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html) object provides a rich API, including the following:
+ `Stack.of(construct)` – A static method that returns the **Stack** in which a construct is defined\. This is useful if you need to interact with a stack from within a reusable construct\. The call fails if a stack cannot be found in scope\.
+ `stack.stackName` \(Python: `stack_name`\) – Returns the physical name of the stack\. As mentioned previously, all AWS CDK stacks have a physical name that the AWS CDK can resolve during synthesis\.
+ `stack.region` and `stack.account` – Return the AWS Region and account, respectively, into which this stack will be deployed\. These properties return one of the following:
  + The account or Region explicitly specified when the stack was defined
  + A string\-encoded token that resolves to the AWS CloudFormation pseudo parameters for account and Region to indicate that this stack is environment agnostic

  For information about how environments are determined for stacks, see [Environments](environments.md)\.
+ `stack.addDependency(stack)` \(Python: `stack.add_dependency(stack)` – Can be used to explicitly define dependency order between two stacks\. This order is respected by the cdk deploy command when deploying multiple stacks at once\.
+ `stack.tags` – Returns a [TagManager](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.TagManager.html) that you can use to add or remove stack\-level tags\. This tag manager tags all resources within the stack, and also tags the stack itself when it's created through AWS CloudFormation\.
+ `stack.partition`, `stack.urlSuffix` \(Python: `url_suffix`\), `stack.stackId` \(Python: `stack_id`\), and `stack.notificationArn` \(Python: `notification_arn`\) – Return tokens that resolve to the respective AWS CloudFormation pseudo parameters, such as `{ "Ref": "AWS::Partition" }`\. These tokens are associated with the specific stack object so that the AWS CDK framework can identify cross\-stack references\.
+ `stack.availabilityZones` \(Python: `availability_zones`\) – Returns the set of Availability Zones available in the environment in which this stack is deployed\. For environment\-agnostic stacks, this always returns an array with two Availability Zones\. For environment\-specific stacks, the AWS CDK queries the environment and returns the exact set of Availability Zones available in the Region that you specified\.
+ `stack.parseArn(arn)` and `stack.formatArn(comps)` \(Python: `parse_arn`, `format_arn`\) – Can be used to work with Amazon Resource Names \(ARNs\)\.
+ `stack.toJsonString(obj)` \(Python: `to_json_string`\) – Can be used to format an arbitrary object as a JSON string that can be embedded in an AWS CloudFormation template\. The object can include tokens, attributes, and references, which are only resolved during deployment\.
+ `stack.templateOptions` \(Python: `template_options`\) – Use to specify AWS CloudFormation template options, such as Transform, Description, and Metadata, for your stack\.

## Working with stacks<a name="stacks-work"></a>

To list all stacks in a CDK app, use the cdk ls command\. The previous example would output the following:

```
stack1
stack2
```

Stacks are deployed as part of an AWS CloudFormation stack into an AWS *[environment](environments.md)*\. The environment covers a specific AWS account and AWS Region\.

When you run the cdk synth command for an app with multiple stacks, the cloud assembly includes a separate template for each stack instance\. Even if the two stacks are instances of the same class, the AWS CDK emits them as two individual templates\.

You can synthesize each template by specifying the stack name in the cdk synth command\. The following example synthesizes the template for **stack1**\.

```
$ cdk synth stack1
```

This approach is conceptually different from how AWS CloudFormation templates are normally used, where a template can be deployed multiple times and parameterized through [AWS CloudFormation parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)\. Although AWS CloudFormation parameters can be defined in the AWS CDK, they are generally discouraged because AWS CloudFormation parameters are resolved only during deployment\. This means that you cannot determine their value in your code\.

For example, to conditionally include a resource in your app based on a parameter value, you must set up an [AWS CloudFormation condition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html) and tag the resource with it\. The AWS CDK takes an approach where concrete templates are resolved at synthesis time\. Therefore, you can use an **if** statement to check the value to determine whether a resource should be defined or some behavior should be applied\.

**Note**  
The AWS CDK provides as much resolution as possible during synthesis time to enable idiomatic and natural usage of your programming language\.

Like any other construct, stacks can be composed together into groups\. The following code shows an example of a service that consists of three stacks: a control plane, a data plane, and monitoring stacks\. The service construct is defined twice: once for the beta environment and once for the production environment\.

------
#### [ TypeScript ]

```
import { App, Stack } from 'aws-cdk-lib';
import { Construct } from 'constructs';

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
const { App, Stack } = require('aws-cdk-lib');
const { Construct } = require('constructs');

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
from aws_cdk import App, Stack
from constructs import Construct

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

import software.amazon.awscdk.App;
import software.amazon.awscdk.Stack;
import software.constructs.Construct;

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
using Constructs;

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

The physical names of the AWS CloudFormation stacks are automatically determined by the AWS CDK based on the stack's construct path in the tree\. By default, a stack's name is derived from the construct ID of the `Stack` object\. However, you can specify an explicit name by using the `stackName` prop \(in Python, `stack_name`\), as follows\.

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

### Nested stacks<a name="stack_nesting"></a>

The [NestedStack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.NestedStack.html) construct offers a way around the AWS CloudFormation 500\-resource limit for stacks\. A nested stack counts as only one resource in the stack that contains it\. However, it can contain up to 500 resources, including additional nested stacks\.

The scope of a nested stack must be a `Stack` or `NestedStack` construct\. The nested stack doesn't need to be declared lexically inside its parent stack\. It is necessary only to pass the parent stack as the first parameter \(`scope`\) when instantiating the nested stack\. Aside from this restriction, defining constructs in a nested stack works exactly the same as in an ordinary stack\.

At synthesis time, the nested stack is synthesized to its own AWS CloudFormation template, which is uploaded to the AWS CDK staging bucket at deployment\. Nested stacks are bound to their parent stack and are not treated as independent deployment artifacts\. They aren't listed by `cdk list`, and they can't be deployed by `cdk deploy`\.

References between parent stacks and nested stacks are automatically translated to stack parameters and outputs in the generated AWS CloudFormation templates, as with any [cross\-stack reference](resources.md#resource_stack)\.

**Warning**  
Changes in security posture are not displayed before deployment for nested stacks\. This information is displayed only for top\-level stacks\.