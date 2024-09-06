# Tutorial: Create your first AWS CDK app<a name="hello_world"></a>

Get started with using the AWS Cloud Development Kit \(AWS CDK\) by using the AWS CDK Command Line Interface \(AWS CDK CLI\) to develop your first CDK app, bootstrap your AWS environment, and deploy your application on AWS\.

## Prerequisites<a name="hello_world_prerequisites"></a>

Before starting this tutorial, complete all set up steps in [Getting started with the AWS CDK](getting_started.md)\.

## About this tutorial<a name="hello_world_about"></a>

In this tutorial, you will create and deploy a simple application on AWS using the AWS CDK\. The application consists of an [AWS Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) that returns a `Hello World!` message when invoked\. The function will be invoked through a [Lambda function URL](https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html) that serves as a dedicated HTTP\(S\) endpoint for your Lambda function\.

Through this tutorial, you will perform the following:
+ **Create your project** – Create a CDK project using the CDK CLI `cdk init` command\.
+ **Configure your AWS environment** – Configure the AWS environment that you will deploy your application into\.
+ **Bootstrap your AWS environment** – Prepare your AWS environment for deployment by bootstrapping it using the CDK CLI `cdk bootstrap` command\.
+ **Develop your app** – Use constructs from the AWS Construct Library to define your Lambda function and Lambda function URL resources\.
+ **Prepare your app for deployment** – Use the CDK CLI to build your app and synthesize an AWS CloudFormation template\.
+ **Deploy your app** – Use the CDK CLI `cdk deploy` command to deploy your application and provision your AWS resources\.
+ **Interact with your application** – Interact with your deployed Lambda function on AWS by invoking it and receiving a response\.
+ **Modify your app** – Modify your Lambda function and deploy to implement your changes\.
+ **Delete your app** – Delete all resources that you created by using the CDK CLI `cdk destroy` command\.

## Step 1: Create your CDK project<a name="hello_world_create"></a>

In this step, you create a new CDK project\. A CDK project should be in its own directory, with its own local module dependencies\.

**To create a CDK project**

1. From a starting directory of your choice, create and navigate to a directory named `hello-cdk`:

   ```
   $ mkdir hello-cdk && cd hello-cdk
   ```
**Important**  
Be sure to name your project directory `hello-cdk`, *exactly as shown here*\. The CDK CLI uses this directory name to name things within your CDK code\. If you use a different directory name, you will run into issues during this tutorial\.

1. From the `hello-cdk` directory, initialize a new CDK project using the CDK CLI `cdk init` command\. Specify the `app` template and your preferred programming language with the `--language` option:

------
#### [ TypeScript ]

   ```
   $ cdk init app --language typescript
   ```

------
#### [ JavaScript ]

   ```
   $ cdk init app --language javascript
   ```

------
#### [ Python ]

   ```
   $ cdk init app --language python
   ```

   After the app has been created, also enter the following two commands\. These activate the app's Python virtual environment and installs the AWS CDK core dependencies\.

   ```
   $ source .venv/bin/activate # On Windows, run `.\venv\Scripts\activate` instead
   $ python -m pip install -r requirements.txt
   ```

------
#### [ Java ]

   ```
   $ cdk init app --language java
   ```

   If you are using an IDE, you can now open or import the project\. In Eclipse, for example, choose **File** > **Import** > **Maven** > **Existing Maven Projects**\. Make sure that the project settings are set to use Java 8 \(1\.8\)\.

------
#### [ C\# ]

   ```
   $ cdk init app --language csharp
   ```

   If you are using Visual Studio, open the solution file in the `src` directory\.

------
#### [ Go ]

   ```
   $ cdk init app --language go
   ```

   After the app has been created, also enter the following command to install the AWS Construct Library modules that the app requires\.

   ```
   $ go get
   ```

------

The `cdk init` command creates a structure of files and folders within the `hello-cdk` directory to help organize the source code for your CDK app\. This structure of files and folders is called your CDK *project*\. Take a moment to explore your CDK project\.

If you have Git installed, each project you create using `cdk init` is also initialized as a Git repository\.

During project initialization, the CDK CLI creates a CDK app containing a single CDK stack\. The CDK app instance is created using the `[App](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html)` construct\. The following is a portion of this code from your CDK application file:

------
#### [ TypeScript ]

Located in `bin/hello-cdk.ts`:

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { HelloCdkStack } from '../lib/hello-cdk-stack';

const app = new cdk.App();
new HelloCdkStack(app, 'HelloCdkStack', {
});
```

------
#### [ JavaScript ]

Located in `bin/hello-cdk.js`:

```
#!/usr/bin/env node

const cdk = require('aws-cdk-lib');
const { HelloCdkStack } = require('../lib/hello-cdk-stack');

const app = new cdk.App();
new HelloCdkStack(app, 'HelloCdkStack', {
});
```

------
#### [ Python ]

Located in `app.py`:

```
#!/usr/bin/env python3
import os

import aws_cdk as cdk

from hello_cdk.hello_cdk_stack import HelloCdkStack


app = cdk.App()
HelloCdkStack(app, "HelloCdkStack",)

app.synth()
```

------
#### [ Java ]

Located in `src/main/java/.../HelloCdkApp.java`:

```
package com.myorg;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Environment;
import software.amazon.awscdk.StackProps;

import java.util.Arrays;

public class HelloCdkApp {
  public static void main(final String[] args) {
    App app = new App();

    new HelloCdkStack(app, "HelloCdkStack", StackProps.builder()
      .build());

    app.synth();
  }
}
```

------
#### [ C\# ]

Located in `src/HelloCdk/Program.cs`:

```
using Amazon.CDK;
using System;
using System.Collections.Generic;
using System.Linq;

namespace HelloCdk
{
  sealed class Program
  {
    public static void Main(string[] args)
    {
      var app = new App();
      new HelloCdkStack(app, "HelloCdkStack", new StackProps
      {});
      app.Synth();
    }
  }
}
```

------
#### [ Go ]

Located in `hello-cdk.go`:

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

  NewHelloCdkStack(app, "HelloCdkStack", &HelloCdkStackProps{
    awscdk.StackProps{
      Env: env(),
    },
  })

  app.Synth(nil)
}

// ...
```

------

The CDK stack is created using the `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` construct\. The following is a portion of this code from your CDK stack file:

------
#### [ TypeScript ]

Located in `lib/hello-cdk-stack.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    
    // Define your constructs here

  }
}
```

------
#### [ JavaScript ]

Located in `lib/hello-cdk-stack.js`:

```
const { Stack } = require('aws-cdk-lib');

class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your constructs here

  }
}

module.exports = { HelloCdkStack }
```

------
#### [ Python ]

Located in `hello_cdk/hello_cdk_stack.py`:

```
from aws_cdk import (
  Stack,
)
from constructs import Construct

class HelloCdkStack(Stack):

  def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
    super().__init__(scope, construct_id, **kwargs)

    # Define your constructs here
```

------
#### [ Java ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;

public class HelloCdkStack extends Stack {
  public HelloCdkStack(final Construct scope, final String id) {
    this(scope, id, null);
  }

  public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
    super(scope, id, props);

    // Define your constructs here
  }
}
```

------
#### [ C\# ]

Located in `src/HelloCdk/HelloCdkStack.cs`:

```
using Amazon.CDK;
using Constructs;

namespace HelloCdk
{
  public class HelloCdkStack : Stack
  {
    internal HelloCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
    {
      // Define your constructs here
    }
  }
}
```

------
#### [ Go ]

Located in `hello-cdk.go`:

```
package main

import (
  "github.com/aws/aws-cdk-go/awscdk/v2"
  "github.com/aws/constructs-go/constructs/v10"
  "github.com/aws/jsii-runtime-go"
)

type HelloCdkStackProps struct {
  awscdk.StackProps
}

func NewHelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
  var sprops awscdk.StackProps
  if props != nil {
    sprops = props.StackProps
  }
  stack := awscdk.NewStack(scope, &id, &sprops)

  return stack
}

// ...
```

------

## Step 2: Configure your AWS environment<a name="hello_world_configure"></a>

In this step, you configure the AWS environment for your CDK stack\. By doing this, you specify which environment your CDK stack will be deployed to\.

First, determine the AWS environment that you want to use\. An AWS environment consists of an AWS account and AWS Region\.

When you use the AWS CLI to configure security credentials on your local machine, you can then use the AWS CLI to obtain AWS environment information for a specific profile\.

**To use the AWS CLI to obtain your AWS account ID**

1. Run the following AWS CLI command to get the AWS account ID for your `default` profile:

   ```
   $ aws sts get-caller-identity --query "Account" --output text
   ```

1. If you prefer to use a named profile, provide the name of your profile using the `--profile` option:

   ```
   $ aws sts get-caller-identity --profile your-profile-name --query "Account" --output text
   ```

**To use the AWS CLI to obtain your AWS Region**

1. Run the following AWS CLI command to get the Region that you configured for your `default` profile:

   ```
   $ aws configure get region
   ```

1. If you prefer to use a named profile, provide the name of your profile using the `--profile` option:

   ```
   $ aws configure get region --profile your-profile-name
   ```

Next, you will configure the AWS environment for your CDK stack by modifying the `HelloCdkStack` instance in your *application file*\. For this tutorial, you will hard code your AWS environment information\. This is recommended for production environments\. For information on other ways to configure environments, see [Configure environments to use with the AWS CDK](configure-env.md)\.

**To configure the environment for your CDK stack**
+ In your *application file*, use the `env` property of the `Stack` construct to configure your environment\. The following is an example:

------
#### [ TypeScript ]

  Located in `bin/hello-cdk.ts`:

  ```
  #!/usr/bin/env node
  import 'source-map-support/register';
  import * as cdk from 'aws-cdk-lib';
  import { HelloCdkStack } from '../lib/hello-cdk-stack';
  
  const app = new cdk.App();
  new HelloCdkStack(app, 'HelloCdkStack', {
    env: { account: '123456789012', region: 'us-east-1' },
  });
  ```

------
#### [ JavaScript ]

  Located in `bin/hello-cdk.js`:

  ```
  #!/usr/bin/env node
  
  const cdk = require('aws-cdk-lib');
  const { HelloCdkStack } = require('../lib/hello-cdk-stack');
  
  const app = new cdk.App();
  new HelloCdkStack(app, 'HelloCdkStack', {
    env: { account: '123456789012', region: 'us-east-1' },
  });
  ```

------
#### [ Python ]

  Located in `app.py`:

  ```
  #!/usr/bin/env python3
  import os
  
  import aws_cdk as cdk
  
  from hello_cdk.hello_cdk_stack import HelloCdkStack
  
  
  app = cdk.App()
  HelloCdkStack(app, "HelloCdkStack",
    env=cdk.Environment(account='123456789012', region='us-east-1'),
    )
    
  app.synth()
  ```

------
#### [ Java ]

  Located in `src/main/java/.../HelloCdkApp.java`:

  ```
  package com.myorg;
  
  import software.amazon.awscdk.App;
  import software.amazon.awscdk.Environment;
  import software.amazon.awscdk.StackProps;
  
  import java.util.Arrays;
  
  public class HelloCdkApp {
      public static void main(final String[] args) {
          App app = new App();
  
          new HelloCdkStack(app, "HelloCdkStack", StackProps.builder()
                  .env(Environment.builder()
                          .account("123456789012")
                          .region("us-east-1")
                          .build())
  
                  .build());
  
          app.synth();
      }
  }
  ```

------
#### [ C\# ]

  Located in `src/HelloCdk/Program.cs`:

  ```
  using Amazon.CDK;
  using System;
  using System.Collections.Generic;
  using System.Linq;
  
  namespace HelloCdk
  {
      sealed class Program
      {
          public static void Main(string[] args)
          {
              var app = new App();
              new HelloCdkStack(app, "HelloCdkStack", new StackProps
              {
                  Env = new Amazon.CDK.Environment
                  {
                      Account = "123456789012",
                      Region = "us-east-1",
                  }
              });
              app.Synth();
          }
      }
  }
  ```

------
#### [ Go ]

  Located in `hello-cdk.go`:

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
  
    NewHelloCdkStack(app, "HelloCdkStack", &HelloCdkStackProps{
      awscdk.StackProps{
        Env: env(),
      },
    })
  
    app.Synth(nil)
  }
  
  func env() *awscdk.Environment {
  	return &awscdk.Environment{
  		Account: jsii.String("123456789012"),
  		Region:  jsii.String("us-east-1"),
  	}
  }
  ```

------

## Step 3: Bootstrap your AWS environment<a name="hello_world_bootstrap"></a>

In this step, you bootstrap the AWS environment that you configured in the previous step\. This prepares your environment for CDK deployments\.

To bootstrap your environment, run the following from the root of your CDK project:

```
$ cdk bootstrap
```

By bootstrapping from the root of your CDK project, you don't have to provide any additional information\. The CDK CLI obtains environment information from your project\. When you bootstrap outside of a CDK project, you must provide environment information with the `cdk bootstrap` command\. For more information, see [Bootstrap your environment for use with the AWS CDK](bootstrapping-env.md)\.

## Step 4: Build your CDK app<a name="hello_world_build"></a>

In most programming environments, you build or compile code after making changes\. This isn't necessary with the AWS CDK since the CDK CLI will automatically perform this step\. However, you can still build manually when you want to catch syntax and type errors\. The following is an example:

------
#### [ TypeScript ]

```
$ npm run build
            
> hello-cdk@0.1.0 build
> tsc
```

------
#### [ JavaScript ]

No build step is necessary\.

------
#### [ Python ]

No build step is necessary\.

------
#### [ Java ]

```
$ mvn compile -q
```

Or press `Control-B` in Eclipse \(other Java IDEs may vary\)

------
#### [ C\# ]

```
$ dotnet build src
```

Or press F6 in Visual Studio

------
#### [ Go ]

```
$ go build
```

------

## Step 5: List the CDK stacks in your app<a name="hello_world_list"></a>

At this point, you should have a CDK app containing a single CDK stack\. To verify, use the CDK CLI `cdk list` command to display your stacks\. The output should display a single stack named `HelloCdkStack`:

```
$ cdk list
HelloCdkStack
```

If you don't see this output, verify that you are in the correct working directory of your project and try again\. If you still don't see your stack, repeat [Step 1: Create your CDK project](#hello_world_create) and try again\.

## Step 6: Define your Lambda function<a name="hello_world_function"></a>

In this step, you import the `[aws\_lambda](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda-readme.html)` module from the AWS Construct Library and use the `[Function](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda.Function.html)` L2 construct\.

Modify your CDK stack file as follows:

------
#### [ TypeScript ]

Located in `lib/hello-cdk-stack.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
// Import the Lambda module
import * as lambda from 'aws-cdk-lib/aws-lambda';

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define the Lambda function resource
    const myFunction = new lambda.Function(this, "HelloWorldFunction", {
      runtime: lambda.Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
      handler: "index.handler",
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
    });
  }
}
```

------
#### [ JavaScript ]

Located in `lib/hello-cdk-stack.js`:

```
const { Stack } = require('aws-cdk-lib');
// Import the Lambda module
const lambda = require('aws-cdk-lib/aws-lambda');

class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define the Lambda function resource
    const myFunction = new lambda.Function(this, "HelloWorldFunction", {
      runtime: lambda.Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
      handler: "index.handler",
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
    });

  }
}

module.exports = { HelloCdkStack }
```

------
#### [ Python ]

Located in `hello_cdk/hello_cdk_stack.py`:

```
from aws_cdk import (
  Stack,
  aws_lambda as _lambda, # Import the Lambda module
)
from constructs import Construct

class HelloCdkStack(Stack):

  def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
    super().__init__(scope, construct_id, **kwargs)

    # Define the Lambda function resource
    my_function = _lambda.Function(
      self, "HelloWorldFunction", 
      runtime = _lambda.Runtime.NODEJS_20_X, # Provide any supported Node.js runtime
      handler = "index.handler",
      code = _lambda.Code.from_inline(
        """
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
        """
      ),
    )
```

------
#### [ Java ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
// Import Lambda function
import software.amazon.awscdk.services.lambda.Code;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;

public class HelloCdkStack extends Stack {
  public HelloCdkStack(final Construct scope, final String id) {
    this(scope, id, null);
  }

  public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
    super(scope, id, props);

    // Define the Lambda function resource
    Function myFunction = Function.Builder.create(this, "HelloWorldFunction")
      .runtime(Runtime.NODEJS_20_X) // Provide any supported Node.js runtime
      .handler("index.handler")
      .code(Code.fromInline(
        "exports.handler = async function(event) {" +
        " return {" +
        " statusCode: 200," +
        " body: JSON.stringify('Hello World!')" +
        " };" +
        "};"))
      .build();

  }
}
```

------
#### [ C\# ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
using Amazon.CDK;
using Constructs;
// Import the Lambda module
using Amazon.CDK.AWS.Lambda;

namespace HelloCdk
{
  public class HelloCdkStack : Stack
  {
    internal HelloCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
    {
      // Define the Lambda function resource
      var myFunction = new Function(this, "HelloWorldFunction", new FunctionProps
      {
        Runtime = Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
        Handler = "index.handler",
        Code = Code.FromInline(@"
          exports.handler = async function(event) {
            return {
              statusCode: 200,
              body: JSON.stringify('Hello World!'),
            };
          };
        "),
      });
    }
  }
}
```

------
#### [ Go ]

Located in `hello-cdk.go`:

```
package main

import (
  "github.com/aws/aws-cdk-go/awscdk/v2"
  "github.com/aws/constructs-go/constructs/v10"
  "github.com/aws/jsii-runtime-go"
  // Import the Lambda module
  "github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
)

type HelloCdkStackProps struct {
  awscdk.StackProps
}

func NewHelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
  var sprops awscdk.StackProps
  if props != nil {
    sprops = props.StackProps
  }
  stack := awscdk.NewStack(scope, &id, &sprops)

  // Define the Lambda function resource
  myFunction := awslambda.NewFunction(stack, jsii.String("HelloWorldFunction"), &awslambda.FunctionProps{
    Runtime: awslambda.Runtime_NODEJS_20_X(), // Provide any supported Node.js runtime
    Handler: jsii.String("index.handler"),
    Code: awslambda.Code_FromInline(jsii.String(`
      exports.handler = async function(event) {
        return {
          statusCode: 200,
          body: JSON.stringify('Hello World!'),
        };
      };
    `)),
  })

  return stack
}

// ...
```

------

Let's take a closer look at the `Function` construct\. Like all constructs, the `Function` class takes three parameters:
+ **scope** – Defines your `Stack` instance as the parent of the `Function` construct\. All constructs that define AWS resources are created within the scope of a stack\. You can define constructs inside of constructs, creating a hierarchy \(tree\)\. Here, and in most cases, the scope is `this` \(`self` in Python\)\.
+ **Id** – The logical ID of the `Function` within your AWS CDK app\. This ID, plus a hash based on the function's location within the stack, uniquely identifies the function during deployment\. The AWS CDK also references this ID when you update the construct in your app and re\-deploy to update the deployed resource\. Here, your logical ID is `HelloWorldFunction`\. Functions can also have a name, specified with the `functionName` property\. This is different from the logical ID\.
+ **props** – A bundle of values that define properties of the function\. Here you define the `runtime`, `handler`, and `code` properties\.

  Props are represented differently in the languages supported by the AWS CDK\.
  + In TypeScript and JavaScript, `props` is a single argument and you pass in an object containing the desired properties\.
  + In Python, props are passed as keyword arguments\.
  + In Java, a Builder is provided to pass the props\. There are two: one for `FunctionProps`, and a second for `Function` to let you build the construct and its props object in one step\. This code uses the latter\.
  + In C\#, you instantiate a `FunctionProps` object using an object initializer and pass it as the third parameter\.

  If a construct's props are optional, you can omit the `props` parameter entirely\.

All constructs take these same three arguments, so it's easy to stay oriented as you learn about new ones\. And as you might expect, you can subclass any construct to extend it to suit your needs, or if you want to change its defaults\.

## Step 7: Define your Lambda function URL<a name="hello_world_url"></a>

In this step, you use the `addFunctionUrl` helper method of the `Function` construct to define a Lambda function URL\. To output the value of this URL at deployment, you will create an AWS CloudFormation output using the `[CfnOutput](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnOutput.html)` construct\.

Add the following to your CDK stack file:

------
#### [ TypeScript ]

Located in `lib/hello-cdk-stack.ts`:

```
// ...

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define the Lambda function resource
    // ...

    // Define the Lambda function URL resource
    const myFunctionUrl = myFunction.addFunctionUrl({
      authType: lambda.FunctionUrlAuthType.NONE,
    });

    // Define a CloudFormation output for your URL
    new cdk.CfnOutput(this, "myFunctionUrlOutput", {
      value: myFunctionUrl.url,
    })

  }
}
```

------
#### [ JavaScript ]

Located in `lib/hello-cdk-stack.js`:

```
const { Stack, CfnOutput } = require('aws-cdk-lib');  // Import CfnOutput

class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define the Lambda function resource
    // ...

    // Define the Lambda function URL resource
    const myFunctionUrl = myFunction.addFunctionUrl({
      authType: lambda.FunctionUrlAuthType.NONE,
    });

    // Define a CloudFormation output for your URL
    new CfnOutput(this, "myFunctionUrlOutput", {
      value: myFunctionUrl.url,
    })

  }
}

module.exports = { HelloCdkStack }
```

------
#### [ Python ]

Located in `hello_cdk/hello_cdk_stack.py`:

```
from aws_cdk import (
  # ...
  CfnOutput # Import CfnOutput
)
from constructs import Construct

class HelloCdkStack(Stack):

  def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
    super().__init__(scope, construct_id, **kwargs)

    # Define the Lambda function resource
    # ...

    # Define the Lambda function URL resource
    my_function_url = my_function.add_function_url(
      auth_type = _lambda.FunctionUrlAuthType.NONE,
    )

    # Define a CloudFormation output for your URL
    CfnOutput(self, "myFunctionUrlOutput", value=my_function_url.url)
```

------
#### [ Java ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
package com.myorg;

// ...
// Import Lambda function URL
import software.amazon.awscdk.services.lambda.FunctionUrl;
import software.amazon.awscdk.services.lambda.FunctionUrlAuthType;
import software.amazon.awscdk.services.lambda.FunctionUrlOptions;
// Import CfnOutput
import software.amazon.awscdk.CfnOutput;

public class HelloCdkStack extends Stack {
  public HelloCdkStack(final Construct scope, final String id) {
    this(scope, id, null);
  }

  public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
    super(scope, id, props);

    // Define the Lambda function resource
    // ...

    // Define the Lambda function URL resource
    FunctionUrl myFunctionUrl = myFunction.addFunctionUrl(FunctionUrlOptions.builder()
      .authType(FunctionUrlAuthType.NONE)
      .build());

    // Define a CloudFormation output for your URL
    CfnOutput.Builder.create(this, "myFunctionUrlOutput")
      .value(myFunctionUrl.getUrl())
      .build();
  }
}
```

------
#### [ C\# ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
// ...

namespace HelloCdk
{
  public class HelloCdkStack : Stack
  {
    internal HelloCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
    {
      // Define the Lambda function resource
      // ...

      // Define the Lambda function URL resource
      var myFunctionUrl = myFunction.AddFunctionUrl(new FunctionUrlOptions
      {
        AuthType = FunctionUrlAuthType.NONE
      });

      // Define a CloudFormation output for your URL
      new CfnOutput(this, "myFunctionUrlOutput", new CfnOutputProps
      {
        Value = myFunctionUrl.Url
      });
    }
  }
}
```

------
#### [ Go ]

Located in `hello-cdk.go`:

```
// ...

func NewHelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
  var sprops awscdk.StackProps
  if props != nil {
    sprops = props.StackProps
  }
  stack := awscdk.NewStack(scope, &id, &sprops)

  // Define the Lambda function resource
  // ...

  // Define the Lambda function URL resource
  myFunctionUrl := myFunction.AddFunctionUrl(&awslambda.FunctionUrlOptions{
    AuthType: awslambda.FunctionUrlAuthType_NONE,
  })

  // Define a CloudFormation output for your URL
  awscdk.NewCfnOutput(stack, jsii.String("myFunctionUrlOutput"), &awscdk.CfnOutputProps{
    Value: myFunctionUrl.Url(),
  })

  return stack
}

// ...
```

------

**Warning**  
To keep this tutorial simple, your Lambda function URL is defined without authentication\. When deployed, this creates a publicly accessible endpoint that can be used to invoke your function\. When you are done with this tutorial, follow [Step 12: Delete your application](#hello_world_delete) to delete these resources\.

## Step 8: Synthesize a CloudFormation template<a name="hello_world_synth"></a>

In this step, you prepare for deployment by synthesizing a CloudFormation template with the CDK CLI `cdk synth` command\. This command performs basic validation of your CDK code, runs your CDK app, and generates a CloudFormation template from your CDK stack\.

If your app contains more than one stack, you must specify which stacks to synthesize\. Since your app contains a single stack, the CDK CLI automatically detects the stack to synthesize\.

If you don't synthesize a template, the CDK CLI will automatically perform this step when you deploy\. However, we recommend that you run this step before each deployment to check for synthesis errors\.

Before synthesizing a template, you can optionally build your application to catch syntax and type errors\. For instructions, see [Step 4: Build your CDK app](#hello_world_build)\.

To synthesize a CloudFormation template, run the following from the root of your project:

```
$ cdk synth
```

**Note**  
If you receive an error like the following, verify that you are in the `hello-cdk` directory and try again:  

```
--app is required either in command-line, in cdk.json or in ~/.cdk.json
```

If successful, the CDK CLI will output a YAML–formatted CloudFormation template to `stdout` and save a JSON–formatted template in the `cdk.out` directory of your project\.

The following is an example output of the CloudFormation template:

### AWS CloudFormation template<a name="hello_world_synth_template"></a>

```
Resources:
  HelloWorldFunctionServiceRoleunique-identifier:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Statement:
          - Action: sts:AssumeRole
            Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
        Version: "2012-10-17"
      ManagedPolicyArns:
        - Fn::Join:
            - ""
            - - "arn:"
              - Ref: AWS::Partition
              - :iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
    Metadata:
      aws:cdk:path: HelloCdkStack/HelloWorldFunction/ServiceRole/Resource
  HelloWorldFunctionunique-identifier:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        ZipFile: "

          \        exports.handler = async function(event) {

          \          return {

          \            statusCode: 200,

          \            body: JSON.stringify('Hello World!'),

          \          };

          \        };

          \      "
      Handler: index.handler
      Role:
        Fn::GetAtt:
          - HelloWorldFunctionServiceRoleunique-identifier
          - Arn
      Runtime: nodejs20.x
    DependsOn:
      - HelloWorldFunctionServiceRoleunique-identifier
    Metadata:
      aws:cdk:path: HelloCdkStack/HelloWorldFunction/Resource
  HelloWorldFunctionFunctionUrlunique-identifier:
    Type: AWS::Lambda::Url
    Properties:
      AuthType: NONE
      TargetFunctionArn:
        Fn::GetAtt:
          - HelloWorldFunctionunique-identifier
          - Arn
    Metadata:
      aws:cdk:path: HelloCdkStack/HelloWorldFunction/FunctionUrl/Resource
  HelloWorldFunctioninvokefunctionurlunique-identifier:
    Type: AWS::Lambda::Permission
    Properties:
      Action: lambda:InvokeFunctionUrl
      FunctionName:
        Fn::GetAtt:
          - HelloWorldFunctionunique-identifier
          - Arn
      FunctionUrlAuthType: NONE
      Principal: "*"
    Metadata:
      aws:cdk:path: HelloCdkStack/HelloWorldFunction/invoke-function-url
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Analytics: v2:deflate64:unique-identifier
    Metadata:
      aws:cdk:path: HelloCdkStack/CDKMetadata/Default
    Condition: CDKMetadataAvailable
Outputs:
  myFunctionUrlOutput:
    Value:
      Fn::GetAtt:
        - HelloWorldFunctionFunctionUrlunique-identifier
        - FunctionUrl
Parameters:
  BootstrapVersion:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /cdk-bootstrap/unique-identifier/version
    Description: Version of the CDK Bootstrap resources in this environment, automatically retrieved from SSM Parameter Store. [cdk:skip]
Rules:
  CheckBootstrapVersion:
    Assertions:
      - Assert:
          Fn::Not:
            - Fn::Contains:
                - - "1"
                  - "2"
                  - "3"
                  - "4"
                  - "5"
                - Ref: BootstrapVersion
        AssertDescription: CDK bootstrap stack version 6 required. Please run 'cdk bootstrap' with a recent version of the CDK CLI.
```

**Note**  
Every generated template contains an `AWS::CDK::Metadata` resource by default\. The AWS CDK team uses this metadata to gain insight into AWS CDK usage and find ways to improve it\. For details, including how to opt out of version reporting, see [Version reporting](cli.md#version_reporting)\.

By defining a single L2 construct, the AWS CDK creates an extensive CloudFormation template containing your Lambda resources, along with the permissions and glue logic required for your resources to interact within your application\.

## Step 9: Deploy your CDK stack<a name="hello_world_deploy"></a>

In this step, you use the CDK CLI `cdk deploy` command to deploy your CDK stack\. This command retrieves your generated CloudFormation template and deploys it through AWS CloudFormation, which provisions your resources as part of a CloudFormation stack\.

From the root of your project, run the following\. Confirm changes if prompted:

```
$ cdk deploy

✨  Synthesis time: 2.69s

HelloCdkStack:  start: Building unique-identifier:current_account-current_region
HelloCdkStack:  success: Built unique-identifier:current_account-current_region
HelloCdkStack:  start: Publishing unique-identifier:current_account-current_region
HelloCdkStack:  success: Published unique-identifier:current_account-current_region
This deployment will make potentially sensitive changes according to your current security approval level (--require-approval broadening).
Please confirm you intend to make the following modifications:

IAM Statement Changes
┌───┬───────────────────────────────────────┬────────┬──────────────────────────┬──────────────────────────────┬───────────┐
│   │ Resource                              │ Effect │ Action                   │ Principal                    │ Condition │
├───┼───────────────────────────────────────┼────────┼──────────────────────────┼──────────────────────────────┼───────────┤
│ + │ ${HelloWorldFunction.Arn}             │ Allow  │ lambda:InvokeFunctionUrl │ *                            │           │
├───┼───────────────────────────────────────┼────────┼──────────────────────────┼──────────────────────────────┼───────────┤
│ + │ ${HelloWorldFunction/ServiceRole.Arn} │ Allow  │ sts:AssumeRole           │ Service:lambda.amazonaws.com │           │
└───┴───────────────────────────────────────┴────────┴──────────────────────────┴──────────────────────────────┴───────────┘
IAM Policy Changes
┌───┬───────────────────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
│   │ Resource                          │ Managed Policy ARN                                                             │
├───┼───────────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
│ + │ ${HelloWorldFunction/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole │
└───┴───────────────────────────────────┴────────────────────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Do you wish to deploy these changes (y/n)? y
```

Similar to `cdk synth`, you don't have to specify the AWS CDK stack since the app contains a single stack\.

During deployment, the CDK CLI displays progress information as your stack is deployed\. When complete, you can go to the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home) to view your `HelloCdkStack` stack\. You can also go to the Lambda console to view your `HelloWorldFunction` resource\.

When deployment completes, the CDK CLI will output your endpoint URL\. Copy this URL for the next step\. The following is an example:

```
...
HelloCdkStack: deploying... [1/1]
HelloCdkStack: creating CloudFormation changeset...

 ✅  HelloCdkStack

✨  Deployment time: 41.65s

Outputs:
HelloCdkStack.myFunctionUrlOutput = https://<api-id>.lambda-url.<Region>.on.aws/
Stack ARN:
arn:aws:cloudformation:Region:account-id:stack/HelloCdkStack/unique-identifier

✨  Total time: 44.34s
```

## Step 10: Interact with your application on AWS<a name="hello_world_interact"></a>

In this step, you interact with your application on AWS by invoking your Lambda function through the function URL\. When you access the URL, your Lambda function returns the `Hello World!` message\.

To invoke your function, access the function URL through your browser or from the command line\. The following is an example:

```
$ curl https://<api-id>.lambda-url.<Region>.on.aws/
"Hello World!"%
```

## Step 11: Modify your application<a name="hello_world_modify"></a>

In this step, you modify the message that the Lambda function returns when invoked\. You perform a diff using the CDK CLI `cdk diff` command to preview your changes and deploy to update your application\. You then interact with your application on AWS to see your new message\.

Modify the `myFunction` instance in your CDK stack file as follows:

------
#### [ TypeScript ]

Located in `lib/hello-cdk-stack.ts`:

```
// ...

export class HelloCdkStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Modify the Lambda function resource
    const myFunction = new lambda.Function(this, "HelloWorldFunction", {
      runtime: lambda.Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
      handler: "index.handler",
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello CDK!'),
          };
        };
      `),
    });

    // ...
```

------
#### [ JavaScript ]

Located in `lib/hello-cdk-stack.js`:

```
// ...

class HelloCdkStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Modify the Lambda function resource
    const myFunction = new lambda.Function(this, "HelloWorldFunction", {
      runtime: lambda.Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
      handler: "index.handler",
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello CDK!'),
          };
        };
      `),
    });

    // ...

  }
}

module.exports = { HelloCdkStack }
```

------
#### [ Python ]

Located in `hello_cdk/hello_cdk_stack.py`:

```
# ...

class HelloCdkStack(Stack):

  def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
    super().__init__(scope, construct_id, **kwargs)

    # Modify the Lambda function resource
    my_function = _lambda.Function(
      self, "HelloWorldFunction", 
      runtime = _lambda.Runtime.NODEJS_20_X, # Provide any supported Node.js runtime
      handler = "index.handler",
      code = _lambda.Code.from_inline(
        """
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello CDK!'),
          };
        };
        """
      ),
    )

    # ...
```

------
#### [ Java ]

Located in `src/main/java/.../HelloCdkStack.java`:

```
// ...

public class HelloCdkStack extends Stack {
  public HelloCdkStack(final Construct scope, final String id) {
    this(scope, id, null);
  }

  public HelloCdkStack(final Construct scope, final String id, final StackProps props) {
    super(scope, id, props);

    // Modify the Lambda function resource
    Function myFunction = Function.Builder.create(this, "HelloWorldFunction")
      .runtime(Runtime.NODEJS_20_X) // Provide any supported Node.js runtime
      .handler("index.handler")
      .code(Code.fromInline(
        "exports.handler = async function(event) {" +
        " return {" +
        " statusCode: 200," +
        " body: JSON.stringify('Hello CDK!')" +
        " };" +
        "};"))
      .build();

    // ...
  }
}
```

------
#### [ C\# ]



```
// ...

namespace HelloCdk
{
  public class HelloCdkStack : Stack
  {
    internal HelloCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
    {
      // Modify the Lambda function resource
      var myFunction = new Function(this, "HelloWorldFunction", new FunctionProps
      {
        Runtime = Runtime.NODEJS_20_X, // Provide any supported Node.js runtime
        Handler = "index.handler",
        Code = Code.FromInline(@"
          exports.handler = async function(event) {
            return {
              statusCode: 200,
              body: JSON.stringify('Hello CDK!'),
            };
          };
        "),
      });

      // ...
    }
  }
}
```

------
#### [ Go ]



```
// ...

type HelloCdkStackProps struct {
  awscdk.StackProps
}

func NewHelloCdkStack(scope constructs.Construct, id string, props *HelloCdkStackProps) awscdk.Stack {
  var sprops awscdk.StackProps
  if props != nil {
    sprops = props.StackProps
  }
  stack := awscdk.NewStack(scope, &id, &sprops)

  // Modify the Lambda function resource
  myFunction := awslambda.NewFunction(stack, jsii.String("HelloWorldFunction"), &awslambda.FunctionProps{
    Runtime: awslambda.Runtime_NODEJS_20_X(), // Provide any supported Node.js runtime
    Handler: jsii.String("index.handler"),
    Code: awslambda.Code_FromInline(jsii.String(`
      exports.handler = async function(event) {
        return {
          statusCode: 200,
          body: JSON.stringify('Hello CDK!'),
        };
      };
    `)),
  })

// ...
```

------

Currently, your code changes have not made any direct updates to your deployed Lambda resource\. Your code defines the desired state of your resource\. To modify your deployed resource, you will use the CDK CLI to synthesize the desired state into a new AWS CloudFormation template\. Then, you will deploy your new CloudFormation template as a change set\. Change sets make only the necessary changes to reach your new desired state\.

To preview your changes, run the `cdk diff` command\. The following is an example:

```
$ cdk diff
Stack HelloCdkStack
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --no-change-set to use a less accurate but faster template-only diff)
Resources
[~] AWS::Lambda::Function HelloWorldFunction HelloWorldFunctionunique-identifier
 └─ [~] Code
     └─ [~] .ZipFile:
         ├─ [-] 
                exports.handler = async function(event) {
                    return {
                      statusCode: 200,
                      body: JSON.stringify('Hello World!'),
                    };
                };
                
         └─ [+] 
                exports.handler = async function(event) {
                    return {
                      statusCode: 200,
                      body: JSON.stringify('Hello CDK!'),
                    };
                };
                

✨  Number of stacks with differences: 1
```

To create this diff, the CDK CLI queries your AWS account account for the latest AWS CloudFormation template for the `HelloCdkStack` stack\. Then, it compares the latest template with the template it just synthesized from your app\.

To implement your changes, run the `cdk deploy` command\. The following is an example:

```
$ cdk deploy

✨  Synthesis time: 2.12s

HelloCdkStack:  start: Building unique-identifier:current_account-current_region
HelloCdkStack:  success: Built unique-identifier:current_account-current_region
HelloCdkStack:  start: Publishing unique-identifier:current_account-current_region
HelloCdkStack:  success: Published unique-identifier:current_account-current_region
HelloCdkStack: deploying... [1/1]
HelloCdkStack: creating CloudFormation changeset...

 ✅  HelloCdkStack

✨  Deployment time: 26.96s

Outputs:
HelloCdkStack.myFunctionUrlOutput = https://unique-identifier.lambda-url.<Region>.on.aws/
Stack ARN:
arn:aws:cloudformation:Region:account-id:stack/HelloCdkStack/unique-identifier

✨  Total time: 29.07s
```

To interact with your application, repeat [Step 10: Interact with your application on AWS](#hello_world_interact)\. The following is an example:

```
$ curl https://<api-id>.lambda-url.<Region>.on.aws/
"Hello CDK!"%
```

## Step 12: Delete your application<a name="hello_world_delete"></a>

In this step, you use the CDK CLI `cdk destroy` command to delete your application\. This command deletes the CloudFormation stack associated with your CDK stack, which includes the resources you created\.

To delete your application, run the `cdk destroy` command and confirm your request to delete the application\. The following is an example:

```
$ cdk destroy
Are you sure you want to delete: HelloCdkStack (y/n)? y
HelloCdkStack: destroying... [1/1]

 ✅  HelloCdkStack: destroyed
```

## Next steps<a name="hello_world_next_steps"></a>

Congratulations\! You've completed this tutorial and have used the AWS CDK to successfully create, modify, and delete resources in the AWS Cloud\. You're now ready to begin using the AWS CDK\.

To learn more about using the AWS CDK in your preferred programming language, see [Work with the AWS CDK library](work-with.md)\.

For additional resources, see the following:
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) to begin exploring the CDK constructs available for your favorite AWS services\.
+ Visit [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0) to discover constructs created by AWS and others\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples) of using the AWS CDK\.

The AWS CDK is an open\-source project\. To contribute, see to [Contributing to the AWS Cloud Development Kit \(AWS CDK\)](https://github.com/aws/aws-cdk/blob/main/CONTRIBUTING.md)\.