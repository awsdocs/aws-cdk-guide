# Introduction to AWS CDK stacks<a name="stacks"></a>

An AWS CDK stack is the smallest single unit of deployment\. It represents a collection of AWS resources that you define using CDK constructs\. When you deploy CDK apps, the resources within a CDK stack are deployed together as an AWS CloudFormation stack\. To learn more about AWS CloudFormation stacks, see [Managing AWS resources as a single unit with AWS CloudFormation stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) in the *AWS CloudFormation User Guide*\.

You define a stack by extending or inheriting from the `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` construct\. The following example is a common pattern for defining a CDK stack on a separate file, known as a *stack file*\. Here, we extend or inherit the `Stack` class and define a constructor that accepts `scope`, `id`, and `props`\. Then, we invoke the base `Stack` class constructor using `super` with the received `scope`, `id`, and `props`:

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class MyCdkStack extends cdk.Stack { 
  constructor(scope: Construct, id: string, props?: cdk.StackProps) { 
    super(scope, id, props); 
    
    // Define your constructs here

  }
}
```

------
#### [ JavaScript ]

```
const { Stack } = require('aws-cdk-lib');

class MyCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your constructs here

  }
}

module.exports = { MyCdkStack }
```

------
#### [ Python ]

```
from aws_cdk import (
  Stack,
)
from constructs import Construct

class MyCdkStack(Stack):

  def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
    super().__init__(scope, construct_id, **kwargs)

    # Define your constructs here
```

------
#### [ Java ]

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;

public class MyCdkStack extends Stack {
  public MyCdkStack(final Construct scope, final String id) {  
    this(scope, id, null);
  }
  
  public MyCdkStack(final Construct scope, final String id, final StackProps props) {
    super(scope, id, props);

    // Define your constructs here
  }
}
```

------
#### [ C\# ]

```
using Amazon.CDK; 
using Constructs;

namespace MyCdk
{
  public class MyCdkStack : Stack
  {
    internal MyCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
    {
      // Define your constructs here
    }
  }
}
```

------
#### [ Go ]

```
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type CdkDemoAppStackProps struct {
	awscdk.StackProps
}

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// The code that defines your stack goes here

	return stack
}

func main() {
	defer jsii.Close()

	app := awscdk.NewApp(nil)

	NewCdkDemoAppStack(app, "CdkDemoAppStack", &CdkDemoAppStackProps{
		awscdk.StackProps{
			Env: env(),
		},
	})

	app.Synth(nil)
} 

//...
```

------

The previous example has only defined a stack\. To create the stack, it must be instantiated within the context of your CDK app\. A common pattern is to define your CDK app and initialize your stack on a separate file, known as an *application file*\.

The following is an example that creates a CDK stack named `MyCdkStack`\. Here, the CDK app is created and `MyCdkStack` is instantiated in the context of the app:

------
#### [ TypeScript ]

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { MyCdkStack } from '../lib/my-cdk-stack';

const app = new cdk.App();
new MyCdkStack(app, 'MyCdkStack', {
});
```

------
#### [ JavaScript ]

```
#!/usr/bin/env node

const cdk = require('aws-cdk-lib');
const { MyCdkStack } = require('../lib/my-cdk-stack');

const app = new cdk.App();
new MyCdkStack(app, 'MyCdkStack', {
});
```

------
#### [ Python ]

Located in `app.py`:

```
#!/usr/bin/env python3
import os

import aws_cdk as cdk

from my_cdk.my_cdk_stack import MyCdkStack


app = cdk.App()
MyCdkStack(app, "MyCdkStack",)

app.synth()
```

------
#### [ Java ]

```
package com.myorg;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Environment;
import software.amazon.awscdk.StackProps;

import java.util.Arrays;

public class MyCdkApp {
  public static void main(final String[] args) {
    App app = new App();

    new MyCdkStack(app, "MyCdkStack", StackProps.builder()
      .build());

    app.synth();
  }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using System;
using System.Collections.Generic;
using System.Linq;

namespace MyCdk
{
  sealed class Program
  {
    public static void Main(string[] args)
    {
      var app = new App();
      new MyCdkStack(app, "MyCdkStack", new StackProps
      {});
      app.Synth();
    }
  }
}
```

------
#### [ Go ]

```
package main

import (
  "github.com/aws/aws-cdk-go/awscdk/v2"
  "github.com/aws/constructs-go/constructs/v10"
  "github.com/aws/jsii-runtime-go"
)

// ...

func main() {
  defer jsii.Close()

  app := awscdk.NewApp(nil)

  NewMyCdkStack(app, "MyCdkStack", &MyCdkStackProps{
    awscdk.StackProps{
      Env: env(),
    },
  })

  app.Synth(nil)
}

// ...
```

------

The following example creates a CDK app that contains two stacks:

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
#### [ Go ]

```
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type MyFirstStackProps struct {
	awscdk.StackProps
}

func NewMyFirstStack(scope constructs.Construct, id string, props *MyFirstStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	myFirstStack := awscdk.NewStack(scope, &id, &sprops)

	// The code that defines your stack goes here

	return myFirstStack
}

type MySecondStackProps struct {
	awscdk.StackProps
}

func NewMySecondStack(scope constructs.Construct, id string, props *MySecondStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	mySecondStack := awscdk.NewStack(scope, &id, &sprops)

	// The code that defines your stack goes here

	return mySecondStack
}

func main() {
	defer jsii.Close()

	app := awscdk.NewApp(nil)

	NewMyFirstStack(app, "MyFirstStack", &MyFirstStackProps{
		awscdk.StackProps{
			Env: env(),
		},
	})

	NewMySecondStack(app, "MySecondStack", &MySecondStackProps{
		awscdk.StackProps{
			Env: env(),
		},
	})

	app.Synth(nil)
}

// ...
```

------

## About the stack API<a name="stack_api"></a>

The `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` object provides a rich API, including the following:
+ `Stack.of(construct)` – A static method that returns the **Stack** in which a construct is defined\. This is useful if you need to interact with a stack from within a reusable construct\. The call fails if a stack cannot be found in scope\.
+ `stack.stackName` \(Python: `stack_name`\) – Returns the physical name of the stack\. As mentioned previously, all AWS CDK stacks have a physical name that the AWS CDK can resolve during synthesis\.
+ `stack.region` and `stack.account` – Return the AWS Region and account, respectively, into which this stack will be deployed\. These properties return one of the following:
  + The account or Region explicitly specified when the stack was defined
  + A string\-encoded token that resolves to the AWS CloudFormation pseudo parameters for account and Region to indicate that this stack is environment agnostic

  For information about how environments are determined for stacks, see [Environments for the AWS CDK](environments.md)\.
+ `stack.addDependency(stack)` \(Python: `stack.add_dependency(stack)` – Can be used to explicitly define dependency order between two stacks\. This order is respected by the cdk deploy command when deploying multiple stacks at once\.
+ `stack.tags` – Returns a [TagManager](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.TagManager.html) that you can use to add or remove stack\-level tags\. This tag manager tags all resources within the stack, and also tags the stack itself when it's created through AWS CloudFormation\.
+ `stack.partition`, `stack.urlSuffix` \(Python: `url_suffix`\), `stack.stackId` \(Python: `stack_id`\), and `stack.notificationArn` \(Python: `notification_arn`\) – Return tokens that resolve to the respective AWS CloudFormation pseudo parameters, such as `{ "Ref": "AWS::Partition" }`\. These tokens are associated with the specific stack object so that the AWS CDK framework can identify cross\-stack references\.
+ `stack.availabilityZones` \(Python: `availability_zones`\) – Returns the set of Availability Zones available in the environment in which this stack is deployed\. For environment\-agnostic stacks, this always returns an array with two Availability Zones\. For environment\-specific stacks, the AWS CDK queries the environment and returns the exact set of Availability Zones available in the Region that you specified\.
+ `stack.parseArn(arn)` and `stack.formatArn(comps)` \(Python: `parse_arn`, `format_arn`\) – Can be used to work with Amazon Resource Names \(ARNs\)\.
+ `stack.toJsonString(obj)` \(Python: `to_json_string`\) – Can be used to format an arbitrary object as a JSON string that can be embedded in an AWS CloudFormation template\. The object can include tokens, attributes, and references, which are only resolved during deployment\.
+ `stack.templateOptions` \(Python: `template_options`\) – Use to specify AWS CloudFormation template options, such as Transform, Description, and Metadata, for your stack\.

## Working with stacks<a name="stacks-work"></a>

Stacks are deployed as an AWS CloudFormation stack into an AWS *[environment](environments.md)*\. The environment covers a specific AWS account and AWS Region\.

When you run the `cdk synth` command for an app with multiple stacks, the cloud assembly includes a separate template for each stack instance\. Even if the two stacks are instances of the same class, the AWS CDK emits them as two individual templates\.

You can synthesize each template by specifying the stack name in the `cdk synth` command\. The following example synthesizes the template for `stack1`:

```
$ cdk synth stack1
```

This approach is conceptually different from how AWS CloudFormation templates are normally used, where a template can be deployed multiple times and parameterized through [AWS CloudFormation parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)\. Although AWS CloudFormation parameters can be defined in the AWS CDK, they are generally discouraged because AWS CloudFormation parameters are resolved only during deployment\. This means that you cannot determine their value in your code\.

For example, to conditionally include a resource in your app based on a parameter value, you must set up an [AWS CloudFormation condition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html) and tag the resource with it\. The AWS CDK takes an approach where concrete templates are resolved at synthesis time\. Therefore, you can use an `if` statement to check the value to determine whether a resource should be defined or some behavior should be applied\.

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
#### [ Go ]

```
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type ControlPlaneStackProps struct {
	awscdk.StackProps
}

func NewControlPlaneStack(scope constructs.Construct, id string, props *ControlPlaneStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	ControlPlaneStack := awscdk.NewStack(scope, jsii.String(id), &sprops)

	// The code that defines your stack goes here

	return ControlPlaneStack
}

type DataPlaneStackProps struct {
	awscdk.StackProps
}

func NewDataPlaneStack(scope constructs.Construct, id string, props *DataPlaneStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	DataPlaneStack := awscdk.NewStack(scope, jsii.String(id), &sprops)

	// The code that defines your stack goes here

	return DataPlaneStack
}

type MonitoringStackProps struct {
	awscdk.StackProps
}

func NewMonitoringStack(scope constructs.Construct, id string, props *MonitoringStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	MonitoringStack := awscdk.NewStack(scope, jsii.String(id), &sprops)

	// The code that defines your stack goes here

	return MonitoringStack
}

type MyServiceStackProps struct {
	awscdk.StackProps
	Prod bool
}

func NewMyServiceStack(scope constructs.Construct, id string, props *MyServiceStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	MyServiceStack := awscdk.NewStack(scope, jsii.String(id), &sprops)

	NewControlPlaneStack(MyServiceStack, "cp", &ControlPlaneStackProps{
		StackProps: sprops,
	})
	NewDataPlaneStack(MyServiceStack, "data", &DataPlaneStackProps{
		StackProps: sprops,
	})
	NewMonitoringStack(MyServiceStack, "mon", &MonitoringStackProps{
		StackProps: sprops,
	})

	return MyServiceStack
}

func main() {
	defer jsii.Close()

	app := awscdk.NewApp(nil)

	betaProps := MyServiceStackProps{
		StackProps: awscdk.StackProps{
			Env: env(),
		},
		Prod: false,
	}

	NewMyServiceStack(app, "beta", &betaProps)

	prodProps := MyServiceStackProps{
		StackProps: awscdk.StackProps{
			Env: env(),
		},
		Prod: true,
	}

	NewMyServiceStack(app, "prod", &prodProps)

	app.Synth(nil)
}

// ...
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

### Working with nested stacks<a name="stack_nesting"></a>

A *nested stack* is a CDK stack that you create inside another stack, known as the parent stack\. You create nested stacks using the `[NestedStack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.NestedStack.html)` construct\.

By using nested stacks, you can organize resources across multiple stacks\. Nested stacks also offer a way around the AWS CloudFormation 500\-resource limit for stacks\. A nested stack counts as only one resource in the stack that contains it\. However, it can contain up to 500 resources, including additional nested stacks\.

The scope of a nested stack must be a `Stack` or `NestedStack` construct\. The nested stack doesn't need to be declared lexically inside its parent stack\. It is necessary only to pass the parent stack as the first parameter \(`scope`\) when instantiating the nested stack\. Aside from this restriction, defining constructs in a nested stack works exactly the same as in an ordinary stack\.

At synthesis time, the nested stack is synthesized to its own AWS CloudFormation template, which is uploaded to the AWS CDK staging bucket at deployment\. Nested stacks are bound to their parent stack and are not treated as independent deployment artifacts\. They aren't listed by `cdk list`, and they can't be deployed by `cdk deploy`\.

References between parent stacks and nested stacks are automatically translated to stack parameters and outputs in the generated AWS CloudFormation templates, as with any [cross\-stack reference](resources.md#resource_stack)\.

**Warning**  
Changes in security posture are not displayed before deployment for nested stacks\. This information is displayed only for top\-level stacks\.