# Continuous integration and delivery \(CI/CD\) using CDK Pipelines<a name="cdk_pipeline"></a>

[CDK Pipelines](https://docs.aws.amazon.com/cdk/api/latest/docs/pipelines-readme.html) is a construct library module for painless continuous delivery of AWS CDK applications\. Whenever you check your AWS CDK app's source code in to AWS CodeCommit, GitHub, or CodeStar, CDK Pipelines can automatically build, test, and deploy your new version\.

CDK Pipelines are self\-updating: if you add application stages or stacks, the pipeline automatically reconfigures itself to deploy those new stages and/or stacks\.

**Note**  
CDK Pipelines supports two APIs: the original API that was made available in the Developer Preview, and a modern one that incorporates feedback from CDK customers received during the preview phase\. The examples in this topic use the modern API\. For details on the differences between the two supported APIs, see [CDK Pipelines original API](https://github.com/aws/aws-cdk/blob/master/packages/@aws-cdk/pipelines/ORIGINAL_API.md)\.

## Bootstrap your AWS environments<a name="cdk_pipeline_bootstrap"></a>

Before you can use CDK Pipelines, you must bootstrap the AWS environment\(s\) to which you will deploy your stacks\. An [environment](environments.md) is an account/region pair to which you want to deploy a CDK stack\. A CDK Pipeline involves at least two environments: the environment where the pipeline is provisioned, and the environment where you want to deploy the application's stacks \(or its stages, which are groups of related stacks\)\. These environments can be the same, though best practices recommend you isolate stages from each other in different AWS accounts or regions\.

**Note**  
See [Bootstrapping](bootstrapping.md) for more information on the kinds of resources created by bootstrapping and how to customize the bootstrap stack\.

You may have already bootstrapped one or more environments so you can deploy assets and Lambda functions using the AWS CDK\. Continuous deployment with CDK Pipelines requires that the CDK Toolkit stack include additional resources, so the bootstrap stack has been extended to include an additional Amazon S3 bucket, an Amazon ECR repository, and IAM roles to give the various parts of a pipeline the permissions they need\. This new style of CDK Toolkit stack will eventually become the default, but at this writing, you must opt in\. The AWS CDK Toolkit will upgrade your existing bootstrap stack or create a new one, as necessary\.

To bootstrap an environment that can provision an AWS CDK pipeline, set the environment variable `CDK_NEW_BOOTSTRAP` before invoking `cdk bootstrap`, as shown below\. Invoking the AWS CDK Toolkit via the `npx` command installs it if necessary, and will use the version of the Toolkit installed in the current project if one exists\. 

\-\-cloudformation\-execution\-policies specifies the ARN of a policy under which future CDK Pipelines deployments will execute\. The default `AdministratorAccess` policy ensures that your pipeline can deploy every type of AWS resource\. If you use this policy, make sure you trust all the code and dependencies that make up your AWS CDK app\.

Most organizations mandate stricter controls on what kinds of resources can be deployed by automation\. Check with the appropriate department within your organization to determine the policy your pipeline should use\.

You may omit the \-\-profile option if your default AWS profile contains the necessary credentials or to instead use the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` to provide your AWS account credentials\.

------
#### [ macOS/Linux ]

```
export CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap aws://ACCOUNT-NUMBER/REGION --profile ADMIN-PROFILE \
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess \
    aws://ACCOUNT-ID/REGION
```

------
#### [ Windows ]

```
set CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap aws://ACCOUNT-NUMBER/REGION --profile ADMIN-PROFILE ^
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess ^
    aws://ACCOUNT-ID/REGION
```

------

To bootstrap additional environments into which AWS CDK applications will be deployed by the pipeline, use the commands below instead\. The \-\-trust option indicates which other account should have permissions to deploy AWS CDK applications into this environment; specify the pipeline's AWS account ID\.

Again, you may omit the \-\-profile option if your default AWS profile contains the necessary credentials or if you are using the `AWS_*` environment variables to provide your AWS account credentials\.

------
#### [ macOS/Linux ]

```
export CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap aws://ACCOUNT-NUMBER/REGION --profile ADMIN-PROFILE \
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess \
    --trust PIPELINE-ACCOUNT-ID \
    aws://ACCOUNT-ID/REGION
```

------
#### [ Windows ]

```
set CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap aws://ACCOUNT-NUMBER/REGION --profile ADMIN-PROFILE ^
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess ^
    --trust PIPELINE-ACCOUNT-ID ^
    aws://ACCOUNT-ID/REGION
```

------

**Tip**  
Use administrative credentials only to bootstrap and to provision the initial pipeline\. Afterward, use the pipeline itself, not your local machine, to deploy changes\.

If you are upgrading a legacy\-bootstrapped environment, the old Amazon S3 bucket is orphaned when the new bucket is created\. Delete it manually using the Amazon S3 console\.

## Initialize project<a name="cdk_pipeline_init"></a>

Create a new, empty GitHub project and clone it to your workstation in the `my-pipeline` directory\. \(Our code examples in this topic use GitHub; you can also use CodeStar or AWS CodeCommit\.\)

```
git clone GITHUB-CLONE-URL my-pipeline
cd my-pipeline
```

**Note**  
You may use a name other than `my-pipeline` for your app's main directory, but since the AWS CDK Toolkit bases some file and class names on the name of the main directory, you'll need to tweak these later in this topic\.

After cloning, initialize the project as usual\.

------
#### [ TypeScript ]

```
cdk init app --language typescript
```

------
#### [ JavaScript ]

```
cdk init app --language javascript
```

------
#### [ Python ]

```
cdk init app --language python
```

After the app has been created, also enter the following two commands to activate the app's Python virtual environment and install the AWS CDK core dependencies\.

```
source .venv/bin/activate
python -m pip install -r requirements.txt
```

------
#### [ Java ]

```
cdk init app --language java
```

If you are using an IDE, you can now open or import the project\. In Eclipse, for example, choose **File** > **Import** > **Maven** > **Existing Maven Projects**\. Make sure that the project settings are set to use Java 8 \(1\.8\)\.

------
#### [ C\# ]

```
cdk init app --language csharp
```

If you are using Visual Studio, open the solution file in the `src` directory\.

------

Install the CDK Pipelines module along with any others you'll be using\.

**Tip**  
Be sure to commit your `cdk.json` and `cdk.context.json` files in source control\. The context information \(such as feature flags and cached values retrieved from your AWS account\) are part of your project's state\. The values may be different in another environment, which can cause instability \(unexpected changes\) in your results\.

------
#### [ TypeScript ]

```
npm install @aws-cdk/pipelines @aws-cdk/aws-lambda
```

------
#### [ JavaScript ]

```
npm install @aws-cdk/pipelines @aws-cdk/aws-lambda
```

------
#### [ Python ]

```
python -m pip install aws_cdk.pipelines aws_cdk.aws_lambda
```

Add the project's dependencies to `requirements.txt` so they can be installed in the CI/CD environment\. It is convenient to use `pip freeze` for this\.

**macOS/Linux**  

```
python -m pip freeze | grep -v '^[-#]' > requirements.txt
```

**Windows**  

```
python -m pip freeze | findstr /R /B /V "[-#]" > requirements.txt
```

------
#### [ Java ]

Edit your project's `pom.xml` and add a `<dependency>` element for the `pipeline` module and the others you'll need\. Follow the template below for each module, placing each inside the existing `<dependencies>` container\.

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>cdk-pipelines</artifactId>
    <version>${cdk.version}</version>
</dependency>
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>lambda</artifactId>
    <version>${cdk.version}</version>
</dependency>
```

After updating `pom.xml`, issue `mvn package` to install the new modules\.

------
#### [ C\# ]

In Visual Studio, choose **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio and add the following packages\.

```
Amazon.CDK.Pipelines
Amazon.CDK.AWS.Lambda
```

------

Finally, add the `@aws-cdk/core:newStyleStackSynthesis` [feature flag](featureflags.md) to the new project's `cdk.json` file\. The file will already contain some context values; add this new one inside the `context` object if it's not already there\.

```
{
  ...
  "context": {
    ...
    "@aws-cdk/core:newStyleStackSynthesis": true
  }
}
```

In a future release of the AWS CDK, "new style" stack synthesis will become the default, but for now we need to opt in using the feature flag\.

## Define a pipeline<a name="cdk_pipeline_define"></a>

Your CDK Pipelines application will include at least two stacks: one that represents the pipeline itself, and one or more stacks that represent the application deployed through it\. Stacks can also be grouped into *stages*, which you can use to deploy copies of infrastructure stacks to different environments\. For now, we'll consider the pipeline, and later delve into the application it will deploy\.

The construct [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CodePipeline.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CodePipeline.html) is the construct that represents a CDK Pipeline that uses AWS CodePipeline as its deployment engine\. When you instantiate `CodePipeline` in a stack, you define the source location for the pipeline \(e\.g\. a GitHub repository\) and the commands to build the app\. For example, the following defines a pipeline whose source is stored in a GitHub repository, and includes a build step for a TypeScript CDK application\. Fill in the information about your GitHub repo where indicated\.

**Note**  
By default, the pipeline authenticates to GitHub using a personal access token stored in Secrets Manager under the name `github-token`\.

You'll also need to update the instantiation of the pipeline stack to specify the AWS account and region\.

------
#### [ TypeScript ]

In `lib/my-pipeline-stack.ts` \(may vary if your project folder isn't named `my-pipeline`\):

```
import * as cdk from '@aws-cdk/core';
import { CodePipeline, CodePipelineSource, ShellStep } from '@aws-cdk/pipelines';

export class MyPipelineStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
        commands: ['npm ci', 'npm run build', 'npx cdk synth']
      })
    });
  }
}
```

In `bin/my-pipeline.ts` \(may vary if your project folder isn't named `my-pipeline`\):

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { MyPipelineStack } from '../lib/my-pipeline-stack';

const app = new cdk.App();
new MyPipelineStack(app, 'MyPipelineStack', {
  env: {
    account: '111111111111',
    region: 'eu-west-1',
  }
});

app.synth();
```

------
#### [ JavaScript ]

In `lib/my-pipeline-stack.js` \(may vary if your project folder isn't named `my-pipeline`\):

```
const cdk = require('@aws-cdk/core');
const { CodePipeline, CodePipelineSource, ShellStep } = require('@aws-cdk/pipelines');

 class MyPipelineStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
        commands: ['npm ci', 'npm run build', 'npx cdk synth']
      })
    });
  }
}

module.exports = { MyPipelineStack }
```

In `bin/my-pipeline.js` \(may vary if your project folder isn't named `my-pipeline`\):

```
#!/usr/bin/env node

const cdk = require('@aws-cdk/core');
const { MyPipelineStack } = require('../lib/my-pipeline-stack');

const app = new cdk.App();
new MyPipelineStack(app, 'MyPipelineStack', {
  env: {
    account: '111111111111',
    region: 'eu-west-1',
  }
});

app.synth();
```

------
#### [ Python ]

In `my-pipeline/my-pipeline-stack.py` \(may vary if your project folder isn't named `my-pipeline`\): 

```
from aws_cdk import core as cdk
from aws_cdk.pipelines import CodePipeline, CodePipelineSource, ShellStep

class MyPipelineStack(cdk.Stack):

    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        pipeline =  CodePipeline(self, "Pipeline", 
                        pipeline_name="MyPipeline",
                        synth=ShellStep("Synth", 
                            input=CodePipelineSource.git_hub("OWNER/REPO", "main"),
                            commands=["npm install -g aws-cdk", 
                                "python -m pip install -r requirements.txt", 
                                "cdk synth"]
                        )
                    )
```

In `app.py`:

```
#!/usr/bin/env python3
from aws_cdk import core as cdk
from my_pipeline.my_pipeline_stack import MyPipelineStack

app = cdk.App()
MyPipelineStack(app, "MyPipelineStack", 
    env=cdk.Environment(account="111111111111", region="eu-west-1")
)

app.synth()
```

------
#### [ Java ]

In `src/main/java/com/myorg/MyPipelineStack.java` \(may vary if your project folder isn't named `my-pipeline`\):

```
package com.myorg;

import java.util.Arrays;
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.pipelines.CodePipeline;
import software.amazon.awscdk.pipelines.CodePipelineSource;
import software.amazon.awscdk.pipelines.ShellStep;

public class MyPipelineStack extends Stack {
    public MyPipelineStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        CodePipeline pipeline = CodePipeline.Builder.create(this, "pipeline")
             .pipelineName("MyPipeline")
             .synth(ShellStep.Builder.create("Synth")
                .input(CodePipelineSource.gitHub("OWNER/REPO", "main"))
                .commands(Arrays.asList("npm install -g aws-cdk", "cdk synth"))
                .build())
             .build();
    }
}
```

In `src/main/java/com/myorg/MyPipelineApp.java` \(may vary if your project folder isn't named `my-pipeline`\):

```
package com.myorg;

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Environment;
import software.amazon.awscdk.core.StackProps;

public class MyPipelineApp {
    public static void main(final String[] args) {
        App app = new App();

        new MyPipelineStack(app, "PipelineStack", StackProps.builder()
            .env(new Environment.builder()
                .account("111111111111")
                .region("eu-west-1")
                .build())
            .build());

        app.synth();
    }
}
```

------
#### [ C\# ]

In `src/MyPipeline/MyPipelineStack.cs` \(may vary if your project folder isn't named `my-pipeline`\):

```
using Amazon.CDK;
using Amazon.CDK.Pipelines;

namespace MyPipeline
{
    public class MyPipelineStack : Stack
    {
        internal MyPipelineStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var pipeline = new CodePipeline(this, "pipeline", new CodePipelineProps
            {
                PipelineName = "MyPipeline",
                Synth = new ShellStep("Synth", new ShellStepProps
                {
                    Input = CodePipelineSource.GitHub("OWNER/REPO", "main"),
                    Commands = new string[] { "npm install -g aws-cdk", "cdk synth" }
                })
            });
        }
    }
}
```

In `src/MyPipeline/Program.cs` \(may vary if your project folder isn't named `my-pipeline`\):

```
using Amazon.CDK;

namespace MyPipeline
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            var app = new App();
            new MyPipelineStack(app, "MyPipelineStack", new StackProps
            {
                Env = new Amazon.CDK.Environment { 
                    Account = "111111111111", Region = "eu-west-1" }
            });

            app.Synth();
        }
    }
}
```

------

You must deploy a pipeline manually once\. After that, the pipeline will keep itself up to date from the source code repository, so make sure the code in the repo is the code you want deployed\. Check in your changes and push to GitHub, then deploy:

```
git add --all
git commit -m "initial commit"
git push
cdk deploy
```

**Tip**  
Now that you've done the initial deployment, your local AWS account no longer needs administrative access, because all changes to your app will be deployed via the pipeline\. All you need to be able to do is push to GitHub\.

## Application stages<a name="cdk_pipeline_stages"></a>

To define a multi\-stack AWS application that can be added to the pipeline all at once, define a subclass of `[Stage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stage.html)` \(not to be confused with `CdkStage` in the CDK Pipelines module\)\.

The stage contains the stacks that make up your application\. If there are dependencies between the stacks, the stacks are automatically added to the pipeline in the right order\. Stacks that don't depend on each other are deployed in parallel\. You can add a dependency relationship between stacks by calling `stack1.addDependency(stack2)`\.

Stages accept a default `env` argument, which becomes the default environment for the stacks inside it\. \(Stacks can still have their own environment specified\.\)\.

An application is added to the pipeline by calling `[addStage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CodePipeline.html#addwbrstagestage-options)()` with instances of `Stage`\. A stage can be instantiated and added to the pipeline multiple times to define different stages of your DTAP or multi\-region application pipeline:

We will create a stack containing a simple Lambda function and place that stack in a stage\. Then we will add the stage to the pipeline so it can be deployed\.

------
#### [ TypeScript ]

Create the new file `lib/my-pipeline-lambda-stack.ts` to hold our application stack containing a Lambda function\.

```
import * as cdk from '@aws-cdk/core';
import { Function, InlineCode, Runtime } from '@aws-cdk/aws-lambda';

export class MyLambdaStack extends cdk.Stack {
    constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
      super(scope, id, props);
  
      new Function(this, 'LambdaFunction', {
        runtime: Runtime.NODEJS_12_X,
        handler: 'index.handler',
        code: new InlineCode('exports.handler = _ => "Hello, CDK";')
      });
    }
}
```

Create the new file `lib/my-pipeline-app-stage.ts` to hold our stage\.

```
import * as cdk from '@aws-cdk/core';
import { MyLambdaStack } from './my-pipeline-lambda-stack';

export class MyPipelineAppStage extends cdk.Stage {
    
    constructor(scope: cdk.Construct, id: string, props?: cdk.StageProps) {
      super(scope, id, props);
  
      const lambdaStack = new MyLambdaStack(this, 'LambdaStack');      
    }
}
```

Edit `lib/my-pipeline-stack.ts` to add the stage to our pipeline\.

```
import * as cdk from '@aws-cdk/core';
import { CodePipeline, CodePipelineSource, ShellStep } from '@aws-cdk/pipelines';
import { MyPipelineAppStage } from './my-pipeline-app-stage';

export class MyPipelineStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
        commands: ['npm ci', 'npm run build', 'npx cdk synth']
      })
    });

    pipeline.addStage(new MyPipelineAppStage(this, "test", {
      env: { account: "111111111111", region: "eu-west-1" }
    }));
  }
}
```

------
#### [ JavaScript ]

Create the new file `lib/my-pipeline-lambda-stack.js` to hold our application stack containing a Lambda function\.

```
const cdk = require('@aws-cdk/core');
const { Function, InlineCode, Runtime } = require('@aws-cdk/aws-lambda');

class MyLambdaStack extends cdk.Stack {
    constructor(scope, id, props) {
      super(scope, id, props);
  
      new Function(this, 'LambdaFunction', {
        runtime: Runtime.NODEJS_12_X,
        handler: 'index.handler',
        code: new InlineCode('exports.handler = _ => "Hello, CDK";')
      });
    }
}

module.exports = { MyLambdaStack }
```

Create the new file `lib/my-pipeline-app-stage.js` to hold our stage\.

```
const cdk = require('@aws-cdk/core');
const { MyLambdaStack } = require('./my-pipeline-lambda-stack');

class MyPipelineAppStage extends cdk.Stage {
    
    constructor(scope, id, props) {
      super(scope, id, props);
  
      const lambdaStack = new MyLambdaStack(this, 'LambdaStack');      
    }
}

module.exports = { MyPipelineAppStage };
```

Edit `lib/my-pipeline-stack.ts` to add the stage to our pipeline\.

```
const cdk = require('@aws-cdk/core');
const { CodePipeline, CodePipelineSource, ShellStep } = require('@aws-cdk/pipelines');
const { MyPipelineAppStage } = require('./my-pipeline-app-stage');

 class MyPipelineStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const pipeline = new CodePipeline(this, 'Pipeline', {
      pipelineName: 'MyPipeline',
      synth: new ShellStep('Synth', {
        input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
        commands: ['npm ci', 'npm run build', 'npx cdk synth']
      })
    });

    pipeline.addStage(new MyPipelineAppStage(this, "test", {
      env: { account: "111111111111", region: "eu-west-1" }
    }));

  }
}

module.exports = { MyPipelineStack }
```

------
#### [ Python ]

Create the new file `my_pipeline/my_pipeline_lambda_stack.py` to hold our application stack containing a Lambda function\. 

```
from aws_cdk import core as cdk
from aws_cdk.aws_lambda import Function, InlineCode, Runtime

class MyLambdaStack(cdk.Stack):
    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        Function(self, "LambdaFunction", 
            runtime=Runtime.NODEJS_12_X,
            handler="index.handler",
            code=InlineCode("exports.handler = _ => 'Hello, CDK';")
        )
```

Create the new file `my_pipeline/my_pipeline_app_stage.py` to hold our stage\.

```
from aws_cdk import core as cdk
from my_pipeline.my_pipeline_lambda_stack import MyLambdaStack

class MyPipelineAppStage(cdk.Stage):
    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        lambdaStack = MyLambdaStack(self, "LambdaStack")
```

Edit `my_pipeline/my_pipeline_stack.py` to add the stage to our pipeline\.

```
from aws_cdk import core as cdk
from aws_cdk.pipelines import CodePipeline, CodePipelineSource, ShellStep
from my_pipeline.my_pipeline_app_stage import MyPipelineAppStage

class MyPipelineStack(cdk.Stack):

    def __init__(self, scope: cdk.Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        pipeline =  CodePipeline(self, "Pipeline", 
                        pipeline_name="MyPipeline",
                        synth=ShellStep("Synth", 
                            input=CodePipelineSource.git_hub("OWNER/REPO", "main"),
                            commands=["npm install -g aws-cdk",
                                "python -m pip install -r requirements.txt",
                                "cdk synth"]))

        pipeline.add_stage(MyPipelineAppStage(self, "test",
            env=cdk.Environment(account="111111111111", region="eu-west-1")))
```

------
#### [ Java ]

Create the new file `src/main/java/com.myorg/MyPipelineLambdaStack.java` to hold our application stack containing a Lambda function\.

```
package com.myorg;

import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;

import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.lambda.InlineCode;

public class MyPipelineLambdaStack extends Stack {
    public MyPipelineLambdaStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineLambdaStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Function.Builder.create(this, "LambdaFunction")
          .runtime(Runtime.NODEJS_12_X)
          .handler("index.handler")
          .code(new InlineCode("exports.handler = _ => 'Hello, CDK';"))
          .build();

    }

}
```

Create the new file `src/main/java/com.myorg/MyPipelineAppStage.java` to hold our stage\.

```
package com.myorg;

import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.Stage;
import software.amazon.awscdk.core.StageProps;

public class MyPipelineAppStage extends Stage {
    public MyPipelineAppStage(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineAppStage(final Construct scope, final String id, final StageProps props) {
        super(scope, id, props);
        
        Stack lambdaStack = new MyPipelineLambdaStack(this, "LambdaStack");
    }

}
```

Edit `src/main/java/com.myorg/MyPipelineStack.java` to add the stage to our pipeline\.

```
package com.myorg;

import java.util.Arrays;
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Environment;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.core.StageProps;
import software.amazon.awscdk.pipelines.CodePipeline;
import software.amazon.awscdk.pipelines.CodePipelineSource;
import software.amazon.awscdk.pipelines.ShellStep;

public class MyPipelineStack extends Stack {
    public MyPipelineStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        final CodePipeline pipeline = CodePipeline.Builder.create(this, "pipeline")
            .pipelineName("MyPipeline")
            .synth(ShellStep.Builder.create("Synth")
                .input(CodePipelineSource.gitHub("OWNER/REPO", "main"))
                .commands(Arrays.asList("npm install -g aws-cdk", "cdk synth"))
                .build())
            .build();
        
        pipeline.addStage(new MyPipelineAppStage(this, "test", StageProps.builder()
            .env(Environment.builder()
                .account("111111111111")
                .region("eu-west-1")
                .build())
            .build()));
    }
}
```

------
#### [ C\# ]

Create the new file `src/MyPipeline/MyPipelineLambdaStack.cs` to hold our application stack containing a Lambda function\.

```
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;

namespace MyPipeline
{
    class MyPipelineLambdaStack : Stack
    {
        public MyPipelineLambdaStack(Construct scope, string id, StackProps props=null) : base(scope, id, props)
        {
            new Function(this, "LambdaFunction", new FunctionProps
            {
                Runtime = Runtime.NODEJS_12_X,
                Handler = "index.handler",
                Code = new InlineCode("exports.handler = _ => 'Hello, CDK';")
            });
        }
    }
}
```

Create the new file `src/MyPipeline/MyPipelineAppStage.cs` to hold our stage\.

```
using Amazon.CDK;

namespace MyPipeline
{
    class MyPipelineAppStage : Stage
    {
        public MyPipelineAppStage(Construct scope, string id, StageProps props=null) : base(scope, id, props)
        {
            Stack lambdaStack = new MyPipelineLambdaStack(this, "LambdaStack");
        }
    }
}
```

Edit `src/MyPipeline/MyPipelineStack.cs` to add the stage to our pipeline\. 

```
using Amazon.CDK;
using Amazon.CDK.Pipelines;

namespace MyPipeline
{
    public class MyPipelineStack : Stack
    {
        internal MyPipelineStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var pipeline = new CodePipeline(this, "pipeline", new CodePipelineProps
            {
                PipelineName = "MyPipeline",
                Synth = new ShellStep("Synth", new ShellStepProps
                {
                    Input = CodePipelineSource.GitHub("OWNER/REPO", "main"),
                    Commands = new string[] { "npm install -g aws-cdk", "cdk synth" }
                })
            });

            pipeline.AddStage(new MyPipelineAppStage(this, "test", new StageProps
            {
                Env = new Environment
                {
                    Account = "111111111111", Region = "eu-west-1"
                }
            }));
        }
    }
}
```

------

Every application stage added by `addStage()` results in the addition of a corresponding pipeline stage, represented by a [StageDeployment](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.StageDeployment.html) instance returned by the `addStage()` call\. You can add pre\-deployment or post\-deployment actions to the stage by calling its `addPre()` or `addPost()` method\.

------
#### [ TypeScript ]

```
// import { ManualApprovalStep } from '@aws-cdk/pipelines';

const testingStage = pipeline.addStage(new MyPipelineAppStage(this, 'testing', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

    testingStage.addPost(new ManualApprovalStep('approval'));
```

------
#### [ JavaScript ]

```
// const { ManualApprovalStep } = require('@aws-cdk/pipelines');

const testingStage = pipeline.addStage(new MyPipelineAppStage(this, 'testing', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

testingStage.addPost(new ManualApprovalStep('approval'));
```

------
#### [ Python ]

```
# from aws_cdk.pipelines import ManualApprovalStep

testing_stage = pipeline.add_stage(MyPipelineAppStage(self, "testing",
    env=cdk.Environment(account="111111111111", region="eu-west-1")))

testing_stage.add_post(ManualApprovalStep('approval'))
```

------
#### [ Java ]

```
// import software.amazon.awscdk.pipelines.StageDeployment;
// import software.amazon.awscdk.pipelines.ManualApprovalStep;

StageDeployment testingStage = 
        pipeline.addStage(new MyPipelineAppStage(this, "test", StageProps.builder()
                .env(Environment.builder()
                        .account("111111111111")
                        .region("eu-west-1")
                        .build())
                .build()));

testingStage.addPost(new ManualApprovalStep("approval"));
```

------
#### [ C\# ]

```
var testingStage = pipeline.AddStage(new MyPipelineAppStage(this, "test", new StageProps
{
    Env = new Environment
    {
        Account = "111111111111", Region = "eu-west-1"
    }
}));

testingStage.AddPost(new ManualApprovalStep("approval"));
```

------

You can add stages to a [Wave](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.Wave.html) to deploy them in parallel, for example when deploying a stage to multiple accounts or regions\.

------
#### [ TypeScript ]

```
const wave = pipeline.addWave('wave');
wave.addStage(new MyApplicationStage(this, 'MyAppEU', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));
wave.addStage(new MyApplicationStage(this, 'MyAppUS', {
  env: { account: '111111111111', region: 'us-west-1' }
}));
```

------
#### [ JavaScript ]

```
const wave = pipeline.addWave('wave');
wave.addStage(new MyApplicationStage(this, 'MyAppEU', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));
wave.addStage(new MyApplicationStage(this, 'MyAppUS', {
  env: { account: '111111111111', region: 'us-west-1' }
}));
```

------
#### [ Python ]

```
wave = pipeline.add_wave("wave")
wave.add_stage(MyApplicationStage(self, "MyAppEU", 
    env=cdk.Environment(account="111111111111", region="eu-west-1")))
wave.add_stage(MyApplicationStage(self, "MyAppUS", 
    env=cdk.Environment(account="111111111111", region="us-west-1")))
```

------
#### [ Java ]

```
// import software.amazon.awscdk.pipelines.Wave;
final Wave wave = pipeline.addWave("wave");
wave.addStage(new MyPipelineAppStage(this, "MyAppEU", StageProps.builder()
        .env(Environment.builder()
                .account("111111111111")
                .region("eu-west-1")
                .build())
        .build()));
wave.addStage(new MyPipelineAppStage(this, "MyAppUS", StageProps.builder()
        .env(Environment.builder()
                .account("111111111111")
                .region("us-west-1")
                .build())
        .build()));
```

------
#### [ C\# ]

```
var wave = pipeline.AddWave("wave");
wave.AddStage(new MyPipelineAppStage(this, "MyAppEU", new StageProps
{
    Env = new Environment
    {
        Account = "111111111111", Region = "eu-west-1"
    }
}));
wave.AddStage(new MyPipelineAppStage(this, "MyAppUS", new StageProps
{
    Env = new Environment
    {
        Account = "111111111111", Region = "us-west-1"
    }
}));
```

------

## Testing deployments<a name="cdk_pipeline_validation"></a>

You can add steps to a CDK Pipeline to validate the deployments you are performing\. Using the CDK Pipeline library's `[ShellStep](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.ShellStep.html)`, you can try to access a just\-deployed Amazon API Gateway backed by a Lambda function, for example, or issue an AWS CLI command to check some setting of a deployed resource\.

In its simplest form, adding validation actions looks like this:

------
#### [ TypeScript ]

```
// stage was returned by pipeline.addStage

stage.addPost(new ShellStep("validate", {
  commands: ['curl -Ssf https://my.webservice.com/'],
}));
```

------
#### [ JavaScript ]

```
// stage was returned by pipeline.addStage

stage.addPost(new ShellStep("validate", {
  commands: ['curl -Ssf https://my.webservice.com/'],
}));
```

------
#### [ Python ]

```
# stage was returned by pipeline.add_stage

stage.add_post(ShellStep("validate",
  commands=['curl -Ssf https://my.webservice.com/']
))
```

------
#### [ Java ]

```
// stage was returned by pipeline.addStage

stage.addPost(ShellStep.Builder.create("validate")
        .commands(Arrays.asList("curl -Ssf https://my.webservice.com/"))
        .build());
```

------
#### [ C\# ]

```
// stage was returned by pipeline.addStage

stage.AddPost(new ShellStep("validate", new ShellStepProps
{
    Commands = new string[] { "curl -Ssf https://my.webservice.com/" }
}));
```

------

Because many AWS CloudFormation deployments result in the generation of resources with unpredictable names, CDK Pipelines provide a way to read AWS CloudFormation outputs after a deployment\. This makes it possible to pass \(for example\) the generated URL of a load balancer to a test action\.

To use outputs, expose the `CfnOutput` object you're interested in and pass it in a step's `envFromCfnOutputs` property to make it available as an environment variable within that step\.

------
#### [ TypeScript ]

```
// given a stack lbStack that exposes a load balancer construct as loadBalancer
this.loadBalancerAddress = new cdk.CfnOutput(lbStack, 'LbAddress', {
  value: `https://${lbStack.loadBalancer.loadBalancerDnsName}/`
});

// pass the load balancer address to a shell step
stage.addPost(new ShellStep("lbaddr", {
  envFromCfnOutputs: {lb_addr: lbStack.loadBalancerAddress},
  commands: ['echo $lb_addr']
}));
```

------
#### [ JavaScript ]

```
// given a stack lbStack that exposes a load balancer construct as loadBalancer
this.loadBalancerAddress = new cdk.CfnOutput(lbStack, 'LbAddress', {
  value: `https://${lbStack.loadBalancer.loadBalancerDnsName}/`
});

// pass the load balancer address to a shell step
stage.addPost(new ShellStep("lbaddr", {
  envFromCfnOutputs: {lb_addr: lbStack.loadBalancerAddress},
  commands: ['echo $lb_addr']
}));
```

------
#### [ Python ]

```
# given a stack lb_stack that exposes a load balancer construct as load_balancer
self.load_balancer_address = cdk.CfnOutput(lb_stack, "LbAddress", 
    value=f"https://{lb_stack.load_balancer.load_balancer_dns_name}/")

# pass the load balancer address to a shell step
stage.add_post(ShellStep("lbaddr", 
    env_from_cfn_outputs={"lb_addr": lb_stack.load_balancer_address}
    commands=["echo $lb_addr"]))
```

------
#### [ Java ]

```
// given a stack lbStack that exposes a load balancer construct as loadBalancer
loadBalancerAddress = CfnOutput.Builder.create(lbStack, "LbAddress")
                            .value(String.format("https://%s/", 
                                    lbStack.loadBalancer.loadBalancerDnsName))
                            .build();

stage.addPost(ShellStep.Builder.create("lbaddr")
    .envFromCfnOutputs(new HashMap<String, CfnOutput>() {{
         put("lbAddr", loadBalancerAddress); }})
    .commands(Arrays.asList("echo $lbAddr"))
    .build());
```

------
#### [ C\# ]

```
// given a stack lbStack that exposes a load balancer construct as loadBalancer
loadBalancerAddress = new CfnOutput(lbStack, "LbAddress", new CfnOutputProps
{
    Value = string.Format("https://{0}/", lbStack.loadBalancer.LoadBalancerDnsName)
});

stage.AddPost(new ShellStep("lbaddr", new ShellStepProps
{
    EnvFromCfnOutputs = new Dictionary<string, CfnOutput>
    {
        {  "lbAddr", loadBalancerAddress }
    },
    Commands = new string[] { "echo $lbAddr" }
}));
```

------

You can write simple validation tests right in the `ShellStep`, but this approach becomes unwieldy when the test is more than a few lines\. For more complex tests, you can bring additional files \(such as complete shell scripts, or programs in other languages\) into the `ShellStep` via the `inputs` property\. The inputs can be any step that has an output, including a source \(such as a GitHub repo\) or another `ShellStep`\.

Bringing in files from the source repository is appropriate if the files are directly usable in the test \(for example, if they are themselves executable\)\. In this example, we declare our GitHub repo as `source` \(rather than instantiating it inline as part of the `CodePipeline`\), then pass this fileset to both the pipeline and the validation test\.

------
#### [ TypeScript ]

```
const source = CodePipelineSource.gitHub('OWNER/REPO', 'main');

const pipeline = new CodePipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  synth: new ShellStep('Synth', {
    input: source,
    commands: ['npm ci', 'npm run build', 'npx cdk synth']
  })
});

const stage = pipeline.addStage(new MyPipelineAppStage(this, 'test', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

stage.addPost(new ShellStep('validate', {
  input: source,
  commands: ['sh ./tests/validate.sh']
}));
```

------
#### [ JavaScript ]

```
const source = CodePipelineSource.gitHub('OWNER/REPO', 'main');

const pipeline = new CodePipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  synth: new ShellStep('Synth', {
    input: source,
    commands: ['npm ci', 'npm run build', 'npx cdk synth']
  })
});

const stage = pipeline.addStage(new MyPipelineAppStage(this, 'test', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

stage.addPost(new ShellStep('validate', {
  input: source,
  commands: ['sh ./tests/validate.sh']
}));
```

------
#### [ Python ]

```
source   = CodePipelineSource.git_hub("OWNER/REPO", "main")

pipeline =  CodePipeline(self, "Pipeline", 
                pipeline_name="MyPipeline",
                synth=ShellStep("Synth", 
                    input=source,
                    commands=["npm install -g aws-cdk",
                        "python -m pip install -r requirements.txt",
                        "cdk synth"]))

stage = pipeline.add_stage(MyApplicationStage(self, "test",
            env=cdk.Environment(account="111111111111", region="eu-west-1")))

stage.add_post(ShellStep("validate", input=source,
    commands=["curl -Ssf https://my.webservice.com/"],
))
```

------
#### [ Java ]

```
final CodePipelineSource source = CodePipelineSource.gitHub("OWNER/REPO", "main");

final CodePipeline pipeline = CodePipeline.Builder.create(this, "pipeline")
        .pipelineName("MyPipeline")
        .synth(ShellStep.Builder.create("Synth")
                .input(source)
                .commands(Arrays.asList("npm install -g aws-cdk", "cdk synth"))
                .build())
        .build();

final StageDeployment stage = 
        pipeline.addStage(new MyPipelineAppStage(this, "test", StageProps.builder()
                .env(Environment.builder()
                        .account("111111111111")
                        .region("eu-west-1")
                        .build())
                .build()));

stage.addPost(ShellStep.Builder.create("validate")
        .input(source)
        .commands(Arrays.asList("sh ./tests/validate.sh"))
        .build());
```

------
#### [ C\# ]

```
var source = CodePipelineSource.GitHub("OWNER/REPO", "main");

var pipeline = new CodePipeline(this, "pipeline", new CodePipelineProps
{
    PipelineName = "MyPipeline",
    Synth = new ShellStep("Synth", new ShellStepProps
    {
        Input = source,
        Commands = new string[] { "npm install -g aws-cdk", "cdk synth" }
    })
});

var stage = pipeline.AddStage(new MyPipelineAppStage(this, "test", new StageProps
{
    Env = new Environment
    {
        Account = "111111111111", Region = "eu-west-1"
    }
}));

stage.AddPost(new ShellStep("validate", new ShellStepProps
{
    Input = source,
    Commands = new string[] { "sh ./tests/validate.sh" }
}));
```

------

Getting the additional files from the synth step is appropriate if your tests need to be compiled, which is done as part of synthesis\.

------
#### [ TypeScript ]

```
const synthStep = new ShellStep('Synth', {
  input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
  commands: ['npm ci', 'npm run build', 'npx cdk synth'],
});

const pipeline = new CodePipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  synth: synthStep
});

const stage = pipeline.addStage(new MyPipelineAppStage(this, 'test', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

// run a script that was transpiled from TypeScript during synthesis
stage.addPost(new ShellStep('validate', {
  input: synthStep,
  commands: ['node tests/validate.js']
}));
```

------
#### [ JavaScript ]

```
const synthStep = new ShellStep('Synth', {
  input: CodePipelineSource.gitHub('OWNER/REPO', 'main'),
  commands: ['npm ci', 'npm run build', 'npx cdk synth'],
});

const pipeline = new CodePipeline(this, 'Pipeline', {
  pipelineName: 'MyPipeline',
  synth: synthStep
});

const stage = pipeline.addStage(new MyPipelineAppStage(this, "test", {
  env: { account: "111111111111", region: "eu-west-1" }
}));

// run a script that was transpiled from TypeScript during synthesis
stage.addPost(new ShellStep('validate', {
  input: synthStep,
  commands: ['node tests/validate.js']
}));
```

------
#### [ Python ]

```
synth_step = ShellStep("Synth", 
                input=CodePipelineSource.git_hub("OWNER/REPO", "main"),
                commands=["npm install -g aws-cdk",
                  "python -m pip install -r requirements.txt",
                  "cdk synth"])

pipeline   = CodePipeline(self, "Pipeline", 
                pipeline_name="MyPipeline",
                synth=synth_step)

stage = pipeline.add_stage(MyApplicationStage(self, "test",
            env=cdk.Environment(account="111111111111", region="eu-west-1")))

# run a script that was compiled during synthesis
stage.add_post(ShellStep("validate",
    input=synth_step,
    commands=["node test/validate.js"],
))
```

------
#### [ Java ]

```
final ShellStep synth = ShellStep.Builder.create("Synth")
                            .input(CodePipelineSource.gitHub("OWNER/REPO", "main"))
                            .commands(Arrays.asList("npm install -g aws-cdk", "cdk synth"))
                            .build();   
        
final CodePipeline pipeline = CodePipeline.Builder.create(this, "pipeline")
        .pipelineName("MyPipeline")
        .synth(synth)
        .build();

final StageDeployment stage = 
        pipeline.addStage(new MyPipelineAppStage(this, "test", StageProps.builder()
                .env(Environment.builder()
                        .account("111111111111")
                        .region("eu-west-1")
                        .build())
                .build()));

stage.addPost(ShellStep.Builder.create("validate")
        .input(synth)
        .commands(Arrays.asList("node ./tests/validate.js"))
        .build());
```

------
#### [ C\# ]

```
var synth = new ShellStep("Synth", new ShellStepProps
{
    Input = CodePipelineSource.GitHub("OWNER/REPO", "main"),
    Commands = new string[] { "npm install -g aws-cdk", "cdk synth" }
});

var pipeline = new CodePipeline(this, "pipeline", new CodePipelineProps
{
    PipelineName = "MyPipeline",
    Synth = synth
});

var stage = pipeline.AddStage(new MyPipelineAppStage(this, "test", new StageProps
{
    Env = new Environment
    {
        Account = "111111111111", Region = "eu-west-1"
    }
}));

stage.AddPost(new ShellStep("validate", new ShellStepProps
{
    Input = synth,
    Commands = new string[] { "node ./tests/validate.js" }
}));
```

------

## Security notes<a name="cdk_pipeline_security"></a>

Any form of continuous delivery has inherent security risks\. Under the AWS [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/), you are responsible for the security of your information in the AWS cloud\. The CDK Pipelines library gives you a head start by incorporating secure defaults and modeling best practices, but by its very nature a library that needs a high level of access to fulfill its intended purpose cannot assure complete security\. There are many attack vectors outside of AWS and your organization\.

In particular, keep in mind the following\.
+ Be mindful of the software you depend on\. Vet all third\-party software you run in your pipeline, as it has the ability to change the infrastructure that gets deployed\. 
+ Use dependency locking to prevent accidental upgrades\. CDK Pipelines respects `package-lock.json` and `yarn.lock` to ensure your dependencies are the ones you expect\.
+ Credentials for production environments should be short\-lived\. After bootstrapping and initial provisioning, there is no need for developers to have account credentials at all; changes can be deployed through the pipeline\. Eliminate the possibility of credentials leaking by not needing them in the first place\!

## Troubleshooting<a name="cdk_pipeline_troubleshooting"></a>

The following issues are commonly encountered while getting started with CDK Pipelines\.

**Pipeline: Internal Failure**  

```
CREATE_FAILED  | AWS::CodePipeline::Pipeline | Pipeline/Pipeline
Internal Failure
```
 Check your GitHub access token\. It might be missing, or might not have the permissions to access the repository\. 

**Key: Policy contains a statement with one or more invalid principals**  

```
CREATE_FAILED | AWS::KMS::Key | Pipeline/Pipeline/ArtifactsBucketEncryptionKey
Policy contains a statement with one or more invalid principals.
```
 One of the target environments has not been bootstrapped with the new bootstrap stack\. Make sure all your target environments are bootstrapped\. 

**Stack is in ROLLBACK\_COMPLETE state and can not be updated\.**  

```
Stack STACK_NAME is in ROLLBACK_COMPLETE state and can not be updated. (Service:
AmazonCloudFormation; Status Code: 400; Error Code: ValidationError; Request
ID: ...)
```
The stack failed its previous deployment and is in a non\-retryable state\. Delete the stack from the AWS CloudFormation console and retry the deployment\. 