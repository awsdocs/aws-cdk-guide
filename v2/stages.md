# Introduction to AWS CDK stages<a name="stages"></a>

An AWS Cloud Development Kit \(AWS CDK\) *stage* represents a group of one or more CDK stacks that are configured to deploy together\. Use stages to deploy the same grouping of stacks to multiple environments, such as development, testing, and production\.

To configure a CDK stage, import and use the `[Stage](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stage.html)` construct\.

The following is a basic example that defines a CDK stage named `MyAppStage`\. We add two CDK stacks, named `AppStack` and `DatabaseStack` to our stage\. For this example, `AppStack` contains application resources and `DatabaseStack` contains database resources\. We then create two instances of `MyAppStage`, for development and production environments:

------
#### [ TypeScript ]

In `cdk-demo-app/lib/app-stack.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

// Define the app stack
export class AppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    // The code that defines your application goes here
  }
}
```

In `cdk-demo-app/lib/database-stack.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

// Define the database stack
export class DatabaseStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    // The code that defines your database goes here
  }
}
```

In `cdk-demo-app/lib/my-stage.ts`:

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import { Stage } from 'aws-cdk-lib';
import { AppStack } from './app-stack';
import { DatabaseStack } from './database-stack';

// Define the stage
export class MyAppStage extends Stage {
  constructor(scope: Construct, id: string, props?: cdk.StageProps) {
    super(scope, id, props);

    // Add both stacks to the stage
    new AppStack(this, 'AppStack');
    new DatabaseStack(this, 'DatabaseStack');
  }
}
```

In `cdk-demo-app/bin/cdk-demo-app.ts`:

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
import { MyAppStage } from '../lib/my-stage';

// Create a CDK app
const app = new cdk.App();

// Create the development stage
new MyAppStage(app, 'Dev', {
  env: { 
    account: '123456789012', 
    region: 'us-east-1' 
  }
});

// Create the production stage
new MyAppStage(app, 'Prod', {
  env: {
    account: '098765432109',
    region: 'us-east-1'
  }
});
```

------
#### [ JavaScript ]

In `cdk-demo-app/lib/app-stack.js`:

```
const { Stack } = require('aws-cdk-lib');

class AppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // The code that defines your application goes here
  }
}

module.exports = { AppStack }
```

In `cdk-demo-app/lib/database-stack.js`:

```
const { Stack } = require('aws-cdk-lib');

class DatabaseStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // The code that defines your database goes here
  }
}

module.exports = { DatabaseStack }
```

In `cdk-demo-app/lib/my-stage.js`:

```
const { Stage } = require('aws-cdk-lib');
const { AppStack } = require('./app-stack');
const { DatabaseStack } = require('./database-stack');

// Define the stage
class MyAppStage extends Stage {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Add both stacks to the stage
    new AppStack(this, 'AppStack');
    new DatabaseStack(this, 'DatabaseStack');
  }
} 

module.exports = { MyAppStage };
```

In `cdk-demo-app/bin/cdk-demo-app.js`:

```
#!/usr/bin/env node

const cdk = require('aws-cdk-lib');
const { MyAppStage } = require('../lib/my-stage');

// Create the CDK app
const app = new cdk.App();

// Create the development stage
new MyAppStage(app, 'Dev', {
  env: {
    account: '123456789012',
    region: 'us-east-1',
  },
});

// Create the production stage
new MyAppStage(app, 'Prod', {
  env: {
    account: '098765432109',
    region: 'us-east-1',
  },
});
```

------
#### [ Python ]

In `cdk-demo-app/cdk_demo_app/app_stack.py`:

```
from aws_cdk import Stack
from constructs import Construct

# Define the app stack
class AppStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # The code that defines your application goes here
```

In `cdk-demo-app/cdk_demo_app/database_stack.py`:

```
from aws_cdk import Stack
from constructs import Construct

# Define the database stack
class DatabaseStack(Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # The code that defines your database goes here
```

In `cdk-demo-app/cdk_demo_app/my_stage.py`:

```
from aws_cdk import Stage
from constructs import Construct
from .app_stack import AppStack
from .database_stack import DatabaseStack

# Define the stage
class MyAppStage(Stage):
    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Add both stacks to the stage
        AppStack(self, "AppStack")
        DatabaseStack(self, "DatabaseStack")
```

In `cdk-demo-app/app.py`:

```
#!/usr/bin/env python3
import os

import aws_cdk as cdk

from cdk_demo_app.my_stage import MyAppStage

#  Create a CDK app
app = cdk.App()

# Create the development stage
MyAppStage(app, 'Dev',
           env=cdk.Environment(account='123456789012', region='us-east-1'),
           )

# Create the production stage 
MyAppStage(app, 'Prod',
           env=cdk.Environment(account='098765432109', region='us-east-1'),
           )

app.synth()
```

------
#### [ Java ]

In `cdk-demo-app/src/main/java/com/myorg/AppStack.java`:

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;

public class AppStack extends Stack {
    public AppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public AppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // The code that defines your application goes here
    }
}
```

In `cdk-demo-app/src/main/java/com/myorg/DatabaseStack.java`:

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;

public class DatabaseStack extends Stack {
    public DatabaseStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public DatabaseStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // The code that defines your database goes here
    }
}
```

In `cdk-demo-app/src/main/java/com/myorg/MyAppStage.java`:

```
package com.myorg;

import software.amazon.awscdk.Stage;
import software.amazon.awscdk.StageProps;
import software.constructs.Construct;

// Define the stage
public class MyAppStage extends Stage {
    public MyAppStage(final Construct scope, final String id, final software.amazon.awscdk.Environment env) {
        super(scope, id, StageProps.builder().env(env).build());

        // Add both stacks to the stage
        new AppStack(this, "AppStack");
        new DatabaseStack(this, "DatabaseStack");
    }
}
```

In `cdk-demo-app/src/main/java/com/myorg/CdkDemoAppApp.java`:

```
package com.myorg;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Environment;
import software.amazon.awscdk.StackProps;

import java.util.Arrays;

public class CdkDemoAppApp {
    public static void main(final String[] args) {
        
        // Create a CDK app
        App app = new App();

        // Create the development stage
        new MyAppStage(app, "Dev", Environment.builder()
                .account("123456789012")
                .region("us-east-1")
                .build());

        // Create the production stage
        new MyAppStage(app, "Prod", Environment.builder()
        .account("098765432109")
        .region("us-east-1")
        .build());

        app.synth();
    }
}
```

------
#### [ C\# ]

In `cdk-demo-app/src/CdkDemoApp/AppStack.cs`:

```
using Amazon.CDK;
using Constructs;

namespace CdkDemoApp
{
    public class AppStack : Stack
    {
        internal AppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // The code that defines your application goes here
        }
    }
}
```

In `cdk-demo-app/src/CdkDemoApp/DatabaseStack.cs`:

```
using Amazon.CDK;
using Constructs;

namespace CdkDemoApp
{
    public class DatabaseStack : Stack
    {
        internal DatabaseStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // The code that defines your database goes here
        }
    }
}
```

In `cdk-demo-app/src/CdkDemoApp/MyAppStage.cs`:

```
using Amazon.CDK;
using Constructs;

namespace CdkDemoApp
{
    // Define the stage
    public class MyAppStage : Stage
    {
        internal MyAppStage(Construct scope, string id, Environment env) : base(scope, id, new StageProps { Env = env })
        {
            // Add both stacks to the stage
            new AppStack(this, "AppStack");
            new DatabaseStack(this, "DatabaseStack");
        }
    }
}
```

In `cdk-demo-app/src/CdkDemoApp/program.cs`:

```
using Amazon.CDK;
using System;
using System.Collections.Generic;
using System.Linq;

namespace CdkDemoApp
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            // Create a CDK app
            var app = new App();

            // Create the development stage
            new MyAppStage(app, "Dev", new Amazon.CDK.Environment
            {
                Account = "123456789012",
                Region = "us-east-1"
            });

            // Create the production stage
            new MyAppStage(app, "Prod", new Amazon.CDK.Environment
            {
                Account = "098765432109",
                Region = "us-east-1"
            });

            app.Synth();
        }
    }
}
```

------
#### [ Go ]

In `cdk-demo-app/cdk-demo-app.go`:

```
package main

import (
	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

// Define the app stack
type AppStackProps struct {
	awscdk.StackProps
}

func NewAppStack(scope constructs.Construct, id string, props *AppStackProps) awscdk.Stack {
	stack := awscdk.NewStack(scope, &id, &props.StackProps)

	// The code that defines your application goes here

	return stack
}

// Define the database stack
type DatabaseStackProps struct {
	awscdk.StackProps
}

func NewDatabaseStack(scope constructs.Construct, id string, props *DatabaseStackProps) awscdk.Stack {
	stack := awscdk.NewStack(scope, &id, &props.StackProps)

	// The code that defines your database goes here

	return stack
}

// Define the stage
type MyAppStageProps struct {
	awscdk.StageProps
}

func NewMyAppStage(scope constructs.Construct, id string, props *MyAppStageProps) awscdk.Stage {
	stage := awscdk.NewStage(scope, &id, &props.StageProps)

	// Add both stacks to the stage
	NewAppStack(stage, "AppStack", &AppStackProps{
		StackProps: awscdk.StackProps{
			Env: props.Env,
		},
	})

	NewDatabaseStack(stage, "DatabaseStack", &DatabaseStackProps{
		StackProps: awscdk.StackProps{
			Env: props.Env,
		},
	})

	return stage
}

func main() {
	defer jsii.Close()

	// Create a CDK app
	app := awscdk.NewApp(nil)

	// Create the development stage
	NewMyAppStage(app, "Dev", &MyAppStageProps{
		StageProps: awscdk.StageProps{
			Env: &awscdk.Environment{
				Account: jsii.String("123456789012"),
				Region:  jsii.String("us-east-1"),
			},
		},
	})

	// Create the production stage
	NewMyAppStage(app, "Prod", &MyAppStageProps{
		StageProps: awscdk.StageProps{
			Env: &awscdk.Environment{
				Account: jsii.String("098765432109"),
				Region:  jsii.String("us-east-1"),
			},
		},
	})

	app.Synth(nil)
}

func env() *awscdk.Environment {
	return nil
}
```

------

When we run `cdk synth`, two cloud assemblies are created in `cdk.out`\. These two cloud assemblies contain the synthesized AWS CloudFormation template and assets for each stage\. The following is snippet of our project directory:

------
#### [ TypeScript ]

```
cdk-demo-app
├── bin
│   └── cdk-demo-app.ts
├── cdk.out
│   ├── assembly-Dev
│   │   ├── DevAppStackunique-hash.assets.json
│   │   ├── DevAppStackunique-hash.template.json
│   │   ├── DevDatabaseStackunique-hash.assets.json
│   │   ├── DevDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── assembly-Prod
│   │   ├── ProdAppStackunique-hash.assets.json
│   │   ├── ProdAppStackunique-hash.template.json
│   │   ├── ProdDatabaseStackunique-hash.assets.json
│   │   ├── ProdDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
└── lib
    ├── app-stack.ts
    ├── database-stack.ts
    └── my-stage.ts
```

------
#### [ JavaScript ]

```
cdk-demo-app
├── bin
│   └── cdk-demo-app.js
├── cdk.out
│   ├── assembly-Dev
│   │   ├── DevAppStackunique-hash.assets.json
│   │   ├── DevAppStackunique-hash.template.json
│   │   ├── DevDatabaseStackunique-hash.assets.json
│   │   ├── DevDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── assembly-Prod
│   │   ├── ProdAppStackunique-hash.assets.json
│   │   ├── ProdAppStackunique-hash.template.json
│   │   ├── ProdDatabaseStackunique-hash.assets.json
│   │   ├── ProdDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
└── lib
    ├── app-stack.js
    ├── database-stack.js
    └── my-stage.js
```

------
#### [ Python ]

```
cdk-demo-app
├── app.py
├── cdk.out
│   ├── assembly-Dev
│   │   ├── DevAppStackunique-hash.assets.json
│   │   ├── DevAppStackunique-hash.template.json
│   │   ├── DevDatabaseStackunique-hash.assets.json
│   │   ├── DevDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── assembly-Prod
│   │   ├── ProdAppStackunique-hash.assets.json
│   │   ├── ProdAppStackunique-hash.template.json
│   │   ├── ProdDatabaseStackunique-hash.assets.json
│   │   ├── ProdDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── cdk.out
│   ├── manifest.json
│   └── tree.json
└── cdk_demo_app
    ├── __init__.py
    ├── app_stack.py
    ├── database_stack.py
    └── my_stage.py
```

------
#### [ Java ]



```
cdk-demo-app
├── cdk.out
│   ├── assembly-Dev
│   │   ├── DevAppStackunique-hash.assets.json
│   │   ├── DevAppStackunique-hash.template.json
│   │   ├── DevDatabaseStackunique-hash.assets.json
│   │   ├── DevDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── assembly-Prod
│   │   ├── ProdAppStackunique-hash.assets.json
│   │   ├── ProdAppStackunique-hash.template.json
│   │   ├── ProdDatabaseStackunique-hash.assets.json
│   │   ├── ProdDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── cdk.out
│   ├── manifest.json
│   └── tree.json
└── src
    └── main
        └── java
            └── com
                └── myorg
                    ├── AppStack.java
                    ├── CdkDemoAppApp.java
                    ├── DatabaseStack.java
                    └── MyAppStage.java
```

------
#### [ C\# ]



```
cdk-demo-app
├── cdk.out
│   ├── assembly-Dev
│   │   ├── DevAppStackunique-hash.assets.json
│   │   ├── DevAppStackunique-hash.template.json
│   │   ├── DevDatabaseStackunique-hash.assets.json
│   │   ├── DevDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── assembly-Prod
│   │   ├── ProdAppStackunique-hash.assets.json
│   │   ├── ProdAppStackunique-hash.template.json
│   │   ├── ProdDatabaseStackunique-hash.assets.json
│   │   ├── ProdDatabaseStackunique-hash.template.json
│   │   ├── cdk.out
│   │   └── manifest.json
│   ├── cdk.out
│   ├── manifest.json
│   └── tree.json
└── src
    └── CdkDemoApp
        ├── AppStack.cs
        ├── DatabaseStack.cs
        ├── MyAppStage.cs
        └── Program.cs
```

------
#### [ Go ]



```
cdk-demo-app
├── cdk-demo-app.go
└── cdk.out
    ├── assembly-Dev
    │   ├── DevAppStackunique-hash.assets.json
    │   ├── DevAppStackunique-hash.template.json
    │   ├── DevDatabaseStackunique-hash.assets.json
    │   ├── DevDatabaseStackunique-hash.template.json
    │   ├── cdk.out
    │   └── manifest.json
    ├── assembly-Prod
    │   ├── ProdAppStackunique-hash.assets.json
    │   ├── ProdAppStackunique-hash.template.json
    │   ├── ProdDatabaseStackunique-hash.assets.json
    │   ├── ProdDatabaseStackunique-hash.template.json
    │   ├── cdk.out
    │   └── manifest.json
    ├── cdk.out
    ├── manifest.json
    └── tree.json
```

------

When we list our stacks with `cdk list`, we see a total of four stacks:

```
$ cdk list
Dev/AppStack (Dev-AppStack)
Dev/DatabaseStack (Dev-DatabaseStack)
Prod/AppStack (Prod-AppStack)
Prod/DatabaseStack (Prod-DatabaseStack)
```

To deploy a specific stage, we run `cdk deploy` and provide the stacks to deploy\. The following is an example that uses the `*` wildcard to deploy both stacks in our `Dev` stage:

```
$ cdk deploy "Dev/*"

✨  Synthesis time: 3.18s

Dev/AppStack (Dev-AppStack)
Dev/AppStack (Dev-AppStack): deploying... [1/2]

 ✅  Dev/AppStack (Dev-AppStack)

✨  Deployment time: 1.11s

Stack ARN:
...
		
✨  Total time: 4.29s

Dev/DatabaseStack (Dev-DatabaseStack)
Dev/DatabaseStack (Dev-DatabaseStack): deploying... [2/2]

 ✅  Dev/DatabaseStack (Dev-DatabaseStack)

✨  Deployment time: 1.09s

Stack ARN:
...
		
✨  Total time: 4.27s
```