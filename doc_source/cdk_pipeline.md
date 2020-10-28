# Continuous integration and delivery \(CI/CD\) using CDK Pipelines<a name="cdk_pipeline"></a>

[CDK Pipelines](https://docs.aws.amazon.com/cdk/api/latest/docs/pipelines-readme.html) is a construct library module for painless continuous delivery of AWS CDK applications\. Whenever you check your AWS CDK app's source code in to AWS CodeCommit, GitHub, or BitBucket, CDK Pipelines can automatically build, test, and deploy your new version\.

CDK Pipelines are self\-updating: if you add new application stages or new stacks, the pipeline automatically reconfigures itself to deploy those new stages and/or stacks\.

If you've looked at our [AWS CodePipeline example](codepipeline_example.md), CDK Pipelines can do everything that example does, and more, with less code\. Going forward, we anticipate widespread adoption of CDK Pipelines by AWS CDK users\.

**Note**  
CDK Pipelines is currently in developer preview, and its API is subject to change\. Breaking API changes will be announced in the AWS CDK [Release Notes](https://github.com/aws/aws-cdk/releases)\.

## Bootstrap your AWS environments<a name="cdk_pipeline_bootstrap"></a>

Before you can use CDK Pipelines, you must bootstrap the AWS environment\(s\) to which you will deploy your stacks\. An [environment](environments.md) is an account/region pair to which you want to deploy a CDK stack\. A CDK Pipeline involves at least two environments: the environment where the pipeline is provisioned, and the environment where you want to deploy the application's stacks \(or its stages, which are groups of related stacks\)\. These environments can be the same, though best practices recommend you isolate stages from each other in different AWS accounts or regions\.

**Note**  
See [Bootstrapping](bootstrapping.md) for more information on the kinds of resources created by bootstrapping and how to customize the bootstrap stack\.

You may have already bootstrapped one or more environments so you can deploy assets and Lambda functions using the AWS CDK\. Continuous deployment with CDK Pipelines requires that the CDK Toolkit stack include additional resources, so the bootstrap stack has been extended to include an additional Amazon S3 bucket, an Amazon ECR repository, and IAM roles to give the various parts of a pipeline the permissions they need\. This new style of CDK Toolkit stack will eventually become the default, but at this writing, you must opt in\. The AWS CDK Toolkit will upgrade your existing bootstrap stack or create a new one, as necessary\.

To bootstrap an environment that can provision an AWS CDK pipeline, set the environment variable `CDK_NEW_BOOTSTRAP` before invoking `cdk bootstrap`, as shown below\. Invoking the AWS CDK Toolkit via the `npx` command installs it if necessary, and will use the version of the Toolkit installed in the current project if one exists\. 

\-\-cloudformation\-execution\-policies specifies the ARN of a policy under which future CDK Pipelines deployments will execute\. The `AdministratorAccess` policy is the default; if you're using it, you may omit this option\. Your organization may require a more restrictive policy\.

You may omit the \-\-profile option if your default AWS profile contains the necessary credentials or to instead use the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` to provide your AWS account credentials\.

------
#### [ Mac OS X/Linux ]

```
export CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap --profile ADMIN-PROFILE \
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess \
    aws://ACCOUNT-ID/REGION
```

------
#### [ Windows ]

```
set CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap --profile ADMIN-PROFILE ^
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess ^
    aws://ACCOUNT-ID/REGION
```

------

To bootstrap additional environments into which AWS CDK applications will be deployed by the pipeline, use the commands below instead\. The `--trust` option indicates which other account should have permissions to deploy AWS CDK applications into this environment; specify the pipeline's AWS account ID\.

Again, you may omit the \-\-profile option if your default AWS profile contains the necessary credentials or if you are using the `AWS_*` environment variables to provide your AWS account credentials\.

------
#### [ Mac OS X/Linux ]

```
export CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap --profile ADMIN-PROFILE \
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess \
    --trust PIPELINE-ACCOUNT-ID \
    aws://ACCOUNT-ID/REGION
```

------
#### [ Windows ]

```
set CDK_NEW_BOOTSTRAP=1 
npx cdk bootstrap --profile ADMIN-PROFILE ^
    --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess ^
    --trust PIPELINE-ACCOUNT-ID ^
    aws://ACCOUNT-ID/REGION
```

------

**Tip**  
Use administrative credentials only to bootstrap and to provision the initial pipeline\. Drop administrative credentials as soon as possible\.

If you are upgrading an existing bootstrapped environment, the old Amazon S3 bucket is orphaned when the new bucket is created\. Delete it manually using the Amazon S3 console\.

## Initialize project<a name="cdk_pipeline_init"></a>

Create a new, empty GitHub project and clone it to your workstation in the `my-pipeline` directory\. \(Our code examples in this topic use GitHub; you can also use BitBucket or AWS CodeCommit\.\)

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

After the app has been created, also enter the following two commands to activate the app's Python virtual environment and install its dependencies\.

```
source .env/bin/activate
python -m pip install -r requirements.txt
```

------
#### [ Java ]

```
cdk init app --language java
```

If you are using an IDE, you can now open or import the project\. In Eclipse, for example, choose **File** > **Import** > **Maven** > **Existing Maven Projects**\. Make sure that the project settings are set ta use Java 8 \(1\.8\)\.

------
#### [ C\# ]

```
cdk init app --language csharp
```

If you are using Visual Studio, open the solution file in the `src` directory\.

------

Install the CDK Pipelines module along with others you'll be using\.

------
#### [ TypeScript ]

```
npm install @aws-cdk/pipelines @aws-cdk/aws-codebuild 
npm install @aws-cdk/aws-codepipeline @aws-cdk/aws-codepipeline-actions
```

------
#### [ JavaScript ]

```
npm install @aws-cdk/pipelines @aws-cdk/aws-codebuild 
npm install @aws-cdk/aws-codepipeline @aws-cdk/aws-codepipeline-actions
```

------
#### [ Python ]

```
python -m pip install aws_cdk.pipelines aws_cdk.aws_codebuild
python -m pip install aws_cdk.aws_codepipeline aws_cdk.aws_codepipeline_actions
```

Freeze your dependencies in `requirements.txt`\.

**Mac OS X/Linux**  

```
python -m pip freeze | grep -v '^[-#]' > requirements.txt
```

**Windows**  

```
python -m pip freeze | findstr /R /B /V "[-#]" > requirements.txt
```

------
#### [ Java ]

Edit your project's `pom.xml` and add a `<dependency>` element for the `pipeline` module and a few others you'll need\. Follow the template below for each module, placing each inside the existing `<dependencies>` container\.

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>cdk-pipelines</artifactId>
    <version>${cdk.version}</version>
</dependency>
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>codebuild</artifactId>
    <version>${cdk.version}</version>
</dependency>
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>codepipeline</artifactId>
    <version>${cdk.version}</version>
</dependency>
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>codepipeline-actions</artifactId>
    <version>${cdk.version}</version>
</dependency>
```

------
#### [ C\# ]

In Visual Studio, choose **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio and add the following packages\. Make sure the **Include prerelease** checkbox is marked, since the CDK Pipelines module is in developer preview\.

```
Amazon.CDK.Pipelines
Amazon.CDK.AWS.CodeBuild
Amazon.CDK.AWS.CodePipeline
Amazon.CDK.AWS.CodePipeline.Actions
```

------

Finally, add the `@aws-cdk/core:newStyleStackSynthesis` [feature flag](featureflags.md) to the new project's `cdk.json` file\. The file will already contain some context values; add this new one inside the `context` object\.

```
{
  ...
  "context": {
    ...
    "@aws-cdk/core:newStyleStackSynthesis": "true"
  }
}
```

In a future release of the AWS CDK, "new style" stack synthesis will become the default, but for now we need to opt in using the feature flag\.

## Define pipelines<a name="cdk_pipeline_define"></a>

The construct [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CdkPipeline.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CdkPipeline.html) is the construct that represents a CDK Pipeline\. When you instantiate `CdkPipeline` in a stack, you define the source location for the pipeline as well as the build commands\. For example, the following defines a pipeline whose source is stored in a GitHub repository, and includes a build step for a TypeScript application\. The Pipeline will be provisioned in account `111111111111` and region `eu-west-1`\.

------
#### [ TypeScript ]

In `lib/my-pipeline-stack.ts` \(may vary if your project folder isn't named `my-pipeline`\):

```
import { Stack, StackProps, Construct, SecretValue } from '@aws-cdk/core';
import { CdkPipeline, SimpleSynthAction } from '@aws-cdk/pipelines';

import * as codepipeline from '@aws-cdk/aws-codepipeline';
import * as codepipeline_actions from '@aws-cdk/aws-codepipeline-actions';

export class MyPipelineStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const sourceArtifact = new codepipeline.Artifact();
    const cloudAssemblyArtifact = new codepipeline.Artifact();

    const pipeline = new CdkPipeline(this, 'Pipeline', {
      pipelineName: 'MyAppPipeline',
      cloudAssemblyArtifact,

      sourceAction: new codepipeline_actions.GitHubSourceAction({
        actionName: 'GitHub',
        output: sourceArtifact,
        oauthToken: SecretValue.secretsManager('GITHUB_TOKEN_NAME'),
        trigger: codepipeline_actions.GitHubTrigger.POLL,
        // Replace these with your actual GitHub project info
        owner: 'GITHUB-OWNER',
        repo: 'GITHUB-REPO',
      }),

      synthAction: SimpleSynthAction.standardNpmSynth({
        sourceArtifact,
        cloudAssemblyArtifact,

        // Use this if you need a build step (if you're not using ts-node
        // or if you have TypeScript Lambdas that need to be compiled).
        buildCommand: 'npm run build',
      }),
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
new MyPipelineStack(app, 'PipelineStack', {
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
const { Stack, SecretValue } = require('@aws-cdk/core');
const { CdkPipeline, SimpleSynthAction } = require('@aws-cdk/pipelines');

const codepipeline = require('@aws-cdk/aws-codepipeline');
const codepipeline_actions = require('@aws-cdk/aws-codepipeline-actions');

class MyPipelineStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const sourceArtifact = new codepipeline.Artifact();
    const cloudAssemblyArtifact = new codepipeline.Artifact();

    const pipeline = new CdkPipeline(this, 'Pipeline', {
      pipelineName: 'MyAppPipeline',
      cloudAssemblyArtifact,

      sourceAction: new codepipeline_actions.GitHubSourceAction({
        actionName: 'GitHub',
        output: sourceArtifact,
        oauthToken: SecretValue.secretsManager('GITHUB_TOKEN_NAME'),
        trigger: codepipeline_actions.GitHubTrigger.POLL,
        // Replace these with your actual GitHub project info
        owner: 'GITHUB-OWNER',
        repo: 'GITHUB-REPO'
      }),

      synthAction: SimpleSynthAction.standardNpmSynth({
        sourceArtifact,
        cloudAssemblyArtifact,

        // Use this if you need a build step (if you're not using ts-node
        // or if you have TypeScript Lambdas that need to be compiled).
        buildCommand: 'npm run build'
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
new MyPipelineStack(app, 'PipelineStack', {
  env: {
    account: '111111111111',
    region: 'eu-west-1'
  }
});

app.synth();
```

------
#### [ Python ]

In `my-pipeline/my-pipeline-stack.py` \(may vary if your project folder isn't named `my-pipeline`\): 

```
from aws_cdk.core import Stack, StackProps, Construct, SecretValue
from aws_cdk.pipelines import CdkPipeline, SimpleSynthAction

import aws_cdk.aws_codepipeline as codepipeline
import aws_cdk.aws_codepipeline_actions as codepipeline_actions

class MyPipelineStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        source_artifact = codepipeline.Artifact()
        cloud_assembly_artifact = codepipeline.Artifact()

        pipeline = CdkPipeline(self, "Pipeline",
            pipeline_name="MyAppPipeline",
            cloud_assembly_artifact=cloud_assembly_artifact,
            source_action=codepipeline_actions.GitHubSourceAction(
                action_name="GitHub",
                output=source_artifact,
                oauth_token=SecretValue.secrets_manager("GITHUB_TOKEN_NAME"),
                trigger=codepipeline_actions.GitHubTrigger.POLL,
                # Replace these with your actual GitHub project info
                owner="GITHUB-OWNER",
                repo="GITHUB-REPO"),
            synth_action=SimpleSynthAction.standard_npm_synth(
                source_artifact=source_artifact,
                cloud_assembly_artifact=cloud_assembly_artifact,
                # Use this if you need a build step (if you're not using ts-node
                # or if you have TypeScript Lambdas that need to be compiled).
                build_command="npm run build"
            )
        )
```

In `app.py`:

```
#!/usr/bin/env python3

from aws_cdk import core
from my_pipeline.my_pipeline_stack import MyPipelineStack

app = core.App()
MyPipelineStack(app, "my-pipeline",
    env=core.Environment(account="111111111111", region="eu-west-1"))
app.synth()
```

------
#### [ Java ]

In `src/main/java/com/myorg/MyPipelineStack.java` \(may vary if your project folder isn't named `my-pipeline`\):

```
package com.myorg;

import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.SecretValue;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.pipelines.CdkPipeline;
import software.amazon.awscdk.pipelines.SimpleSynthAction;
import software.amazon.awscdk.pipelines.StandardNpmSynthOptions;
import software.amazon.awscdk.services.codepipeline.Artifact;
import software.amazon.awscdk.services.codepipeline.actions.GitHubSourceAction;
import software.amazon.awscdk.services.codepipeline.actions.GitHubTrigger;

public class MyPipelineStack extends Stack {
    public MyPipelineStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        final Artifact sourceArtifact = new Artifact();
        final Artifact cloudAssemblyArtifact = new Artifact();
        
        final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
                .pipelineName("MyAppPipeline")
                .cloudAssemblyArtifact(cloudAssemblyArtifact)
                .sourceAction(GitHubSourceAction.Builder.create()
                        .actionName("GitHub")
                        .output(sourceArtifact)
                        .oauthToken(SecretValue.secretsManager("GITHUB_TOKEN_NAME"))
                        .trigger(GitHubTrigger.POLL)
                        .owner("GITHUB-OWNER")
                        .repo("GITHUB-REPO")
                        .build())
                .synthAction(SimpleSynthAction.standardNpmSynth(
                        StandardNpmSynthOptions.builder()
                            .sourceArtifact(sourceArtifact)
                            .cloudAssemblyArtifact(cloudAssemblyArtifact)
                            .buildCommand("npm run build")
                            .build()))
                .build();
    }
}
```

In `src/main/java/com/myorg/MyPipelineApp.java` \(may vary if your project folder isn't named `my-pipeline`\):

```
package com.myorg;

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Environment;

public class MyPipelineApp {
    public static void main(final String[] args) {
        App app = new App();

        MyPipelineStack.Builder.create(app, "PipelineStack")
            .env(new Environment.Builder()
                .account("111111111111")
                .region("eu-west-1")
                .build())
            .build();

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
using Amazon.CDK.AWS.CodePipeline;
using Amazon.CDK.AWS.CodePipeline.Actions;

namespace MyPipeline
{
    public class MyPipelineStack : Stack
    {
        internal MyPipelineStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
        {
            var sourceArtifact = new Artifact_();
            var cloudAssemblyArtifact = new Artifact_();

            var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
            {
                PipelineName = "MyAppPipeline",
                CloudAssemblyArtifact = cloudAssemblyArtifact,
                SourceAction = new GitHubSourceAction(new GitHubSourceActionProps
                {
                    ActionName = "GitHub",
                    Output = sourceArtifact,
                    OauthToken = SecretValue.SecretsManager("GITHUB TOKEN_NAME"),
                    Trigger = GitHubTrigger.POLL,
                    Owner = "GITHUB-OWNER",
                    Repo = "GITHUB-REPO"
                }),
                SynthAction = SimpleSynthAction.StandardNpmSynth(new StandardNpmSynthOptions
                {
                    SourceArtifact = sourceArtifact,
                    CloudAssemblyArtifact = cloudAssemblyArtifact,
                    BuildCommand = "npm run build"
                })
            });
        }
    }
}
```

In `src/MyPipeline/Program.cs` \(may vary if your project folder isn't named `my-pipeline`\):

```
using Amazon.CDK;
using System;
using System.Collections.Generic;
using System.Linq;

namespace MyPipeline
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            var app = new App();
            new MyPipelineStack(app, "MyPipelineStack", new StackProps
            {
                Env = new Amazon.CDK.Environment
                {
                    Account = "111111111111",
                    Region = "eu-west-1"
                } 
            });
            app.Synth();
        }
    }
}
```

------

Note the following in this example:
+ The source code is stored in a GitHub repository\.
+ The GitHub access token needed to access the repo is retrieved from AWS Secrets Manager\. Provide the name of the secret where indicated\.
+ Specify the owner of the repository and the repo name where indicated\.

You must deploy a CDK Pipeline manually once\. After that, the pipeline will keep itself up to date from the source code repository\. To perform the initial deployment:

```
git add --all
git commit -m "initial commit"
git push
cdk deploy
```

**Tip**  
Now that you've done the initial deployment, you no longer need AWS administrative access\.

## Sources and synth actions<a name="cdk_pipeline_building"></a>

As we've seen in the preceding example, the basic pieces of CDK pipelines are *sources* and *synth actions*\.

Sources are places where your code lives\. Any source from the [codepipeline\-actions](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-codepipeline-actions-readme.html) module can be used\.

Synth actions \(`synthAction`\) define how to build and synth the project\. A synth action can be any AWS CodePipeline action that produces an artifact containing an AWS CDK Cloud Assembly \(the `cdk.out` directory created by `cdk synth`\)\. Pass the output artifact of the synth operation in the Pipeline's `cloudAssemblyArtifact` property\.

`SimpleSynthAction` is available for synths that can be performed by running a couple of simple shell commands \(install, build, and synth\) using AWS CodeBuild\. When using these, the source repository does not require a `buildspec.yml`\. Here's an example of using `[SimpleSynthAction](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.SimpleSynthAction.html)` to run a Maven \(Java\) build followed by a `cdk synth`:

------
#### [ TypeScript ]

```
const pipeline = new CdkPipeline(this, 'Pipeline', {
  // ...
  synthAction: new SimpleSynthAction({
    sourceArtifact,
    cloudAssemblyArtifact,
    installCommand: 'npm install -g aws-cdk',
    buildCommand: 'mvn package',
    synthCommand: 'cdk synth',
  })
});
```

------
#### [ JavaScript ]

```
const pipeline = new CdkPipeline(this, 'Pipeline', {
  // ...
  synthAction: new SimpleSynthAction({
    sourceArtifact,
    cloudAssemblyArtifact,
    installCommand: 'npm install -g aws-cdk',
    buildCommand: 'mvn package',
    synthCommand: 'cdk synth'
  })
});
```

------
#### [ Python ]

```
class MyPipeline(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        pipeline = CdkPipeline(self, "Pipeline",
            # ...
            synth_action=SimpleSynthAction(
                source_artifact=source_artifact,
                cloud_assembly_artifact=cloud_assembly_artifact,
                install_command="npm install -g aws-cdk",
                build_command="mvn package",
                synth_command="cdk synth"
            ))
```

------
#### [ Java ]

```
final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
        // ...
        .synthAction(SimpleSynthAction.Builder.create()
                .sourceArtifact(sourceArtifact)
                .cloudAssemblyArtifact(cloudAssemblyArtifact)
                .installCommand("npm install -g aws-cdk")
                .buildCommand("mvn package")
                .synthCommand("cdk synth")
                .build())
        .build();
```

------
#### [ C\# ]

```
var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
{
    SynthAction = new SimpleSynthAction(new SimpleSynthActionProps
    {
        SourceArtifact = sourceArtifact,
        CloudAssemblyArtifact = cloudAssemblyArtifact,
        InstallCommand = "npm install -g aws-cdk",
        BuildCommand = "mvn package",
        SynthCommand = "cdk synth"
    })
});
```

------

A couple of convention\-based synth operations for TypeScript or JavaScript projects are available as class methods of `SimpleSynthAction`:
+ `[standardNpmSynth](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.SimpleSynthAction.html#static-standard-wbr-npm-wbr-synthoptions-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span)()` builds using NPM conventions\. Expects a `package-lock.json`, a `cdk.json`, and expects the CDK Toolkit to be a versioned dependency in `package.json`\. Does not perform a build step by default\.
+ `[standardYarnSynth](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.SimpleSynthAction.html#static-standard-wbr-yarn-wbr-synthoptions-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span)()` builds using Yarn conventions\. Expects a `yarn.lock`, a `cdk.json`, and expects the CDK Toolkit to be a versioned dependency in `package.json`\. Does not perform a build step by default\.

If your needs are not covered by `SimpleSynthAction`, you can add a custom build/synth step by creating a custom AWS CodeBuild project and passing a corresponding `CodeBuildAction` to the pipeline\.

## Application stages<a name="cdk_pipeline_stages"></a>

To define a multi\-stack AWS application that can be added to the pipeline all at once, define a subclass of `[Stage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stage.html)` \(not to be confused with `CdkStage` in the CDK Pipelines module\)\.

The stage contains the stacks that make up your application\. If there are dependencies between the stacks, the stacks are automatically added to the pipeline in the right order\. Stacks that don't depend on each other are deployed in parallel\. You can add a dependency relationship between stacks by calling `stack1.addDependency(stack2)`\.

Stages accept a default `env` argument, which the `Stack`s inside the `Stage` will use if no environment is specified for them\.

An application is added to the pipeline by calling `[addApplicationStage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CdkPipeline.html#add-wbr-application-wbr-stageappstage-options-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span)()` with instances of the Stage\. A stage can be instantiated and added to the pipeline multiple times to define different stages of your DTAP or multi\-region application pipeline:

------
#### [ TypeScript ]

```
import { Construct, Stack, StackProps, Stage, StageProps } from '@aws-cdk/core';
import * as codepipeline from '@aws-cdk/aws-codepipeline';
import { CdkPipeline } from '@aws-cdk/pipelines';

export class DatabaseStack extends Stack {
  // ...
}

export class ComputeStack extends Stack {
  // ...
}

// Your application
// May consist of one or more Stacks
//
export class MyApplication extends Stage {
  constructor(scope: Construct, id: string, props?: StageProps) {
    super(scope, id, props);

    const dbStack = new DatabaseStack(this, 'Database');
    new ComputeStack(this, 'Compute', {
      table: dbStack.table,
    });
  }
}

// Stack to hold the pipeline
//
export class MyPipelineStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    const sourceArtifact = new codepipeline.Artifact();
    const cloudAssemblyArtifact = new codepipeline.Artifact();

    const pipeline = new CdkPipeline(this, 'Pipeline', {
      // ...source and build information here
    });

    // Do this as many times as necessary with any account and region
    // Account and region may be different from the pipeline's.
    pipeline.addApplicationStage(new MyApplication(this, 'Prod', {
      env: {
        account: '123456789012',
        region: 'eu-west-1',
      }
    }));
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Stage } = require('@aws-cdk/core');
const codepipeline = require('@aws-cdk/aws-codepipeline');
const { CdkPipeline } = require('@aws-cdk/pipelines');

class DatabaseStack extends Stack {
  // ...
}

class ComputeStack extends Stack {
  // ...
}

// Your application
// May consist of one or more Stacks
//
class MyApplication extends Stage {
  constructor(scope, id, props) {
    super(scope, id, props);

    const dbStack = new DatabaseStack(this, 'Database');
    new ComputeStack(this, 'Compute', {
      table: dbStack.table
    });
  }
}

// Stack to hold the pipeline
//
class MyPipelineStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const sourceArtifact = new codepipeline.Artifact();
    const cloudAssemblyArtifact = new codepipeline.Artifact();

    const pipeline = new CdkPipeline(this, 'Pipeline', {
      // ...source and build information here
    });

    // Do this as many times as necessary with any account and region
    // Account and region may be different from the pipeline's.
    pipeline.addApplicationStage(new MyApplication(this, 'Prod', {
      env: {
        account: '123456789012',
        region: 'eu-west-1'
      }
    }));
  }
}

module.exports = { MyApplication, MyPipelineStack, ComputeStack, DatabaseStack }
```

------
#### [ Python ]

```
from my_pipeline.my_pipeline_stack import source_artifact
from aws_cdk.core import Construct, Stack, Stage, Environment
from aws_cdk.pipelines import CdkPipeline
import aws_cdk.aws_codepipeline as code_pipeline

class DatabaseStack(Stack):
    pass    # ...

class ComputeStack(Stack):
    pass    # ...

# Your application
# May consist of one or more Stacks
#
class MyApplication(Stage):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        db_stack = DatabaseStack(self, "Database")
        ComputeStack(self, "Compute", table=db_stack.table)

# Stack to hold the pipeline
#
class MyPipelineStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        source_artifact = code_pipeline.Artifact()
        cloud_assembly_artifact = code_pipeline.Artifact()

        pipeline = CdkPipeline(self, "Pipeline",
            # ...source and build information here
        )

        # Do this as many times as necessary with any account and region
        # Account and region may be different from the pipeline's.
        pipeline.add_application_stage(MyApplication(self, 'Prod',
            env=Environment(account="123456789012", region="eu-west-1")))
```

------
#### [ Java ]

```
class DatabaseStack extends Stack {
    Table table;
    
    public DatabaseStack(Construct scope, String id) {
        super(scope, id);
        // ...
    }
    
    public Table getTable() {
        return table;
    }
}

class ComputeStack extends Stack {
    public ComputeStack(Construct scope, String id, Table table) {
        // ...        
    }
}

// Your application
// May consist of one or more Stacks
//
class MyApplication extends Stage {
    public MyApplication(Construct scope, String id, StageProps props) {
        super(scope, id, props);
        
        DatabaseStack dbStack = new DatabaseStack(this, "Database");
        new ComputeStack(this, "Compute", dbStack.getTable());
    }
    
}

// Stack to hold the pipeline
//
public class MyPipelineStack extends Stack {
    public MyPipelineStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyPipelineStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        final Artifact sourceArtifact = new Artifact();
        final Artifact cloudAssemblyArtifact = new Artifact();
                        
        final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
                // ...source and build information here
                .build();

        // Do this as many times as necessary with any account and region
        // Account and region may be different from the pipeline's.
        pipeline.addApplicationStage(new MyApplication(this, "Prod", new StackProps.Builder()
            .env(new Environment.Builder()
                .account("123456789012")
                .region("eu-west-1")
                .build())
            .build()));
    }
}
```

------
#### [ C\# ]

```
public class DatabaseStack : Stack
{
    public Table Table { get; set; }

    public DatabaseStack(Construct scope, string id) : base(scope, id)
    {
        // ...
    }

}

public class ComputeStack : Stack
{
    public ComputeStack(Construct scope, string id, Table table) : base(scope, id)
    {
        // ...        
    }
}

// Your application
// May consist of one or more Stacks
//
public class MyApplication : Stage
{
    public MyApplication(Construct scope, string id, Amazon.CDK.StageProps props) : base(scope, id, props)
    {
        var dbStack = new DatabaseStack(this, "Database");
        new ComputeStack(this, "Compute", dbStack.Table);
    }
    
}

// Stack to hold the pipeline
//
public class MyPipelineStack : Stack
{
    public MyPipelineStack(Construct scope, string id, StackProps props) : base(scope, id, props)
    {
        var sourceArtifact = new Artifact_();
        var cloudAssemblyArtifact = new Artifact_();

        var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
        {
            // ... source and build information here
        });

        // Do this as many times as necessary with any account and region
        // Account and region may be different from the pipeline's.
        pipeline.AddApplicationStage(new MyApplication(this, "Prod", new Amazon.CDK.StageProps
        {
            Env = new Amazon.CDK.Environment
            {
                Account = "123456789012",
                Region = "eu-west-1"
            }
        }));
    }
}
```

------

Every application stage added by `addApplicationStage()` leads to the addition of an individual pipeline stage, which is returned by the `addApplicationStage()` call\. This stage is represented by the [CdkStage](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CdkStage.html) construct\. You can add more actions to the stage by calling its `[addActions](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.CdkStage.html#add-wbr-actionsactions-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span)()` method\. For example:

**Note**  
`core.Stage` is a stage in an AWS CDK app containing stacks\. `pipelines.CdkStage` is a stage in a CDK pipeline\.

------
#### [ TypeScript ]

```
// import { ManualApprovalAction } from '@aws-cdk/aws-codepipeline-actions';

const testingStage = pipeline.addApplicationStage(new MyApplication(this, 'Testing', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

// Add an action -- in this case, a Manual Approval action
// (testingStage.addManualApprovalAction() is an equivalent convenience method)
testingStage.addActions(new ManualApprovalAction({
  actionName: 'ManualApproval',
  runOrder: testingStage.nextSequentialRunOrder(),
}));
```

------
#### [ JavaScript ]

```
// const { ManualApprovalAction } = require('@aws-cdk/aws-codepipeline-actions');

const testingStage = pipeline.addApplicationStage(new MyApplication(this, 'Testing', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

// Add an action -- in this case, a Manual Approval action
// (testingStage.addManualApprovalAction() is an equivalent convenience method)
testingStage.addActions(new ManualApprovalAction({
  actionName: 'ManualApproval',
  runOrder: testingStage.nextSequentialRunOrder()
}));
```

------
#### [ Python ]

```
# from aws_cdk.aws_codepipeline_actions import ManualApprovalAction

testing_stage = pipeline.add_application_stage(MyApplication(self, "Testing",
                    env=Environment(account="111111111111", region="eu-west-1")))

# Add an action -- in this case, a Manual Approval action
# (testingStage.addManualApprovalAction() is an equivalent convenience method)
testing_stage.add_actions(ManualApprovalAction(
    action_name="ManualApproval",
    run_order=testing_stage.next_sequential_run_order()
))
```

------
#### [ Java ]

```
// import software.amazon.awscdk.services.codepipeline.actions.ManualApprovalAction;

final CdkStage testingStage = pipeline.addApplicationStage(new MyApplication(this, "Testing",
     new StageProps.Builder()
         .env(new Environment.Builder()
             .account("111111111111")
             .region("eu-west-1")
             .build())
         .build()));

// Add an action -- in this case, a Manual Approval action
// (testingStage.addManualApprovalAction() is an equivalent convenience method)
testingStage.addActions(ManualApprovalAction.Builder.create()
        .actionName("ManualApproval")
        .runOrder(testingStage.nextSequentialRunOrder())
        .build());
```

------
#### [ C\# ]

```
// using Amazon.CDK.AWS.CodePipeline.Actions;

var testingStage = pipeline.AddApplicationStage(new MyApplication(this, "Testing",
        new Amazon.CDK.StageProps
        {
            Env = new Amazon.CDK.Environment
            {
                Account = "111111111111",
                Region = "eu-west-1"
            }
        }));

// Add an action -- in this case, a Manual Approval action
// (testingStage.AddManualApprovalAction() is an equivalent convenience method)
testingStage.AddActions(new ManualApprovalAction(new ManualApprovalActionProps {
    ActionName = "ManualApproval",
    RunOrder = testingStage.NextSequentialRunOrder()
}));
```

------

You can also add more than one application stage to a pipeline stage\. For example:

------
#### [ TypeScript ]

```
// Add two application stages to the same pipeline stage
testingStage.addApplication(new MyApplication1(this, 'MyApp1', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

testingStage.addApplication(new MyApplication2(this, 'MyApp2', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));
```

------
#### [ JavaScript ]

```
// Add two application stages to the same pipeline stage
testingStage.addApplication(new MyApplication1(this, 'MyApp1', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));

testingStage.addApplication(new MyApplication2(this, 'MyApp2', {
  env: { account: '111111111111', region: 'eu-west-1' }
}));
```

------
#### [ Python ]

```
# Add two application stages to the same pipeline stage
testing_stage.add_application(MyApplication1(this, 'MyApp1',
    env=Environment(account="111111111111", region="eu-west-1")))

testing_stage.add_application(MyApplication2(this, 'MyApp2', 
    env=Environment(account="111111111111", region="eu-west-1")))
```

------
#### [ Java ]

```
// Add two application stages to the same pipeline stage
testingStage.addApplication(new MyApplication1(this, "MyApp1", new StageProps.Builder()
    .env(new Environment.Builder()
        .account("111111111111")
        .region("eu-west-1")
        .build())
    .build()));

testingStage.addApplication(new MyApplication2(this, "MyApp2", new StageProps.Builder()
    .env(new Environment.Builder()
        .account("111111111111")
        .region("eu-west-1")
        .build())
    .build()));
```

------
#### [ C\# ]

```
// Add two application stages to the same pipeline stage

testingStage.AddApplication(new MyApplication1(this, "MyApp1", new Amazon.CDK.StageProps
{
    Env = new Amazon.CDK.Environment
    {
        Account = "111111111111",
        Region = "eu-west-1"
    }
}));

testingStage.AddApplication(new MyApplication2(this, "MyApp1", new Amazon.CDK.StageProps
{
    Env = new Amazon.CDK.Environment
    {
        Account = "111111111111",
        Region = "eu-west-1"
    }
}));
```

------

## Testing deployments<a name="cdk_pipeline_validation"></a>

You can add any type of AWS CodePipeline action to a CDK Pipeline to validate the deployments you are performing\. Using the CDK Pipeline library's `[ShellScriptAction](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_pipelines.ShellScriptAction.html)`, you can try to access a just\-deployed Amazon API Gateway backed by a Lambda function, for example, or issue an AWS CLI command to check some setting of a deployed resource\.

In its simplest form, adding validation actions looks like this:

------
#### [ TypeScript ]

```
// stage is a CdkStage returned by pipeline.addApplicationStage

stage.addActions(new ShellScriptAction({
  name: 'MyValidation',
  commands: ['curl -Ssf https://my.webservice.com/'],
  // ... more configuration ...
}));
```

------
#### [ JavaScript ]

```
// stage is a CdkStage returned by pipeline.addApplicationStage

stage.addActions(new ShellScriptAction({
  name: 'MyValidation',
  commands: ['curl -Ssf https://my.webservice.com/']
  // ... more configuration ...
}));
```

------
#### [ Python ]

```
# stage is a CdkStage returned by pipeline.addApplicationStage

stage.add_actions(ShellScriptAction(name="MyValidation",
  commands=['curl -Ssf https://my.webservice.com/'],
  # ... more configuration ...
))
```

------
#### [ Java ]

```
// stage is a CdkStage returned by pipeline.addApplicationStage

stage.addActions(ShellScriptAction.Builder.create()
        .actionName("MyValidation")
        .commands(Arrays.asList("curl -Ssf https://my.webservice.com/"))
        // ... more configuration ...
        .build());
```

------
#### [ C\# ]

```
// stage is a CdkStage returned by pipeline.addApplicationStage
stage.AddActions(new ShellScriptAction(new ShellScriptActionProps
{
    ActionName = "MyValidation",
    Commands = new string[]
    {
        "curl -Ssf https://my.webservice.com/"
        // ... more configuration ...
    }
}));
```

------

Because many AWS CloudFormation deployments result in the generation of resources with unpredictable names, CDK Pipelines provide a way to read AWS CloudFormation outputs after a deployment\. This makes it possible to pass \(for example\) the generated URL of a load balancer to a test action\.

To use outputs, expose the `CfnOutput` object you're interested in and pass it `pipeline.stackOutput()`\.

------
#### [ TypeScript ]

```
export class MyLbApplication extends Stage {
  public readonly loadBalancerAddress: CfnOutput;
  
  constructor(scope: Construct, id: string, props?: StageProps) {
    super(scope, id, props);
  
    const lbStack = new LoadBalancerStack(this, 'Stack');
  
    // Or create this in `LoadBalancerStack` directly
    this.loadBalancerAddress = new CfnOutput(lbStack, 'LbAddress', {
      value: `https://${lbStack.loadBalancer.loadBalancerDnsName}/`
    });
  }
}

const lbApp = new MyLbApplication(this, 'MyApp', {
  env: { /* ... */ }
});

const stage = pipeline.addApplicationStage(lbApp);
stage.addActions(new ShellScriptAction({
  // ...
  useOutputs: {
    // When the test is executed, this will make $URL contain the
    // load balancer address.
    URL: pipeline.stackOutput(lbApp.loadBalancerAddress),
  }
}));
```

------
#### [ JavaScript ]

```
class MyLbApplication extends Stage {

  constructor(scope, id, props) {
    super(scope, id, props);

    const lbStack = new LoadBalancerStack(this, 'Stack');

    // Or create this in `LoadBalancerStack` directly
    this.loadBalancerAddress = new CfnOutput(lbStack, 'LbAddress', {
      value: `https://${lbStack.loadBalancer.loadBalancerDnsName}/`
    });
  }
}

const lbApp = new MyLbApplication(this, 'MyApp', {
  env: { /* ... */ }
});

const stage = pipeline.addApplicationStage(lbApp);
stage.addActions(new ShellScriptAction({
  // ...
  useOutputs: {
    // When the test is executed, this will make $URL contain the
    // load balancer address.
    URL: pipeline.stackOutput(lbApp.loadBalancerAddress)
  }
}));
```

------
#### [ Python ]

```
class MyLbApplication(Stage):
    load_balancer_address: CfnOutput = None

    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, str, **kwargs)

        lb_stack = LoadBalancerStack(self, "Stack")

        # Or create this in `LoadBalancerStack` directly
        self.load_balancer_address = CfnOutput(lb_stack, "LbAddress",
            value=f"https://{lb_stack.load_balancer_dns_name}")

lb_app = MyLbApplication(self, "Myapp",
    env=Environment(...))

stage = pipeline.add_application_stage(lb_app)
stage.add_actions(ShellScriptAction(
    # ...
    use_outputs=pipeline.stack_output(
        # When the test is executed, this will make $URL contain the
        # load balancer address.
        URL=lb_app.load_balancer_address)
))
```

------
#### [ Java ]

```
class MyLbApplication extends Stage {
    CfnOutput loadBalancerAddress;
    
    public MyLbApplication(Construct scope, String id, StageProps props) {
        super(scope, id, props);

        LoadBalancerStack lbStack = new LoadBalancerStack(this, "Stack");

        // Or create this in `LoadBalancerStack` directly
        loadBalancerAddress = CfnOutput.Builder.create(lbStack, "LbAddress")
                .value(String.format("https://%s/", lbStack.getLoadBalancer().getDnsName()))
                .build();
    }

    public CfnOutput getLoadBalancerAddress() {
        return loadBalancerAddress;
    }
}

// some time later...
public class MyPipelineStack extends Stack {
    public MyPipelineStack(final Construct scope, final String id) {
        super(scope, id, null);
    }

    @SuppressWarnings("serial")
    public MyPipelineStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
                // ...source and build information here
                .build();

        final MyLbApplication lbApp = // ...

        final CdkStage stage = pipeline.addApplicationStage(lbApp);
        stage.addActions(ShellScriptAction.Builder.create()
                // ...
                .useOutputs(new HashMap<String, StackOutput>() {{
                    put("URL", pipeline.stackOutput(lbApp.getLoadBalancerAddress()));
                }})
                .build());
    }
}
```

------
#### [ C\# ]

```
public class MyLbApplication : Stage
{
    public CfnOutput LoadBalancerAddress { get; set; }

    public MyLbApplication(Construct scope, string id, Amazon.CDK.StageProps props) :
        base(scope, id, props)
    {

        LoadBalancerStack LbStack = new LoadBalancerStack(this, "Stack");

        // Or create this in `LoadBalancerStack` directly
        var loadBalancerAddress = new CfnOutput(LbStack, "LbAddress", new CfnOutputProps
        {
            Value = $"https://{LbStack.LoadBalancer}/"
        });
    }
}

public class MyPipelineStack : Stack
{
    public MyPipelineStack(Construct scope, string id, StackProps props = null) : base(scope, id)
    {
        var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
        {
            // ... source and build information here
        });

        MyLbApplication LbApp = new MyLbApplication(this, "App", new Amazon.CDK.StageProps
        {
            // set up your application stage
        });

        CdkStage stage = pipeline.AddApplicationStage(LbApp);
        stage.AddActions(new ShellScriptAction(new ShellScriptActionProps
        {
            // ...
            UseOutputs = new Dictionary<string, StackOutput>
            {
                ["URL"] = pipeline.StackOutput(LbApp.LoadBalancerAddress)
            }
        }));
    }
}
```

------

The `ShellScriptAction` limits you to rather small validation tests—basically whatever you can write in a few lines of shell script\. You can bring additional files \(such as complete shell scripts, or scripts in other languages\) into the test via the `additionalArtifacts` property\.

Bringing in files from the source repository is appropriate if the files are directly usable in the test \(for example, if they are themselves executable\)\. Pass the `sourceArtifact`:

------
#### [ TypeScript ]

```
const sourceArtifact = new codepipeline.Artifact();

const pipeline = new CdkPipeline(this, 'Pipeline', {
  // ...
});

const validationAction = new ShellScriptAction({
  name: 'TestUsingSourceArtifact',
  additionalArtifacts: [sourceArtifact],

  // 'test.sh' comes from the source repository
  commands: ['./test.sh'],
});
```

------
#### [ JavaScript ]

```
const sourceArtifact = new codepipeline.Artifact();

const pipeline = new CdkPipeline(this, 'Pipeline', {
  // ...
});

const validationAction = new ShellScriptAction({
  name: 'TestUsingSourceArtifact',
  additionalArtifacts: [sourceArtifact],

  // 'test.sh' comes from the source repository
  commands: ['./test.sh']
});
```

------
#### [ Python ]

```
source_artifact = code_pipeline.Artifact()

pipeline = CdkPipeline(self, "Pipeline", ...)

validation_action = ShellScriptAction(
    name="TestUsingSourceArtifact",
    additional_artifacts=[source_artifact],
    # 'test.sh' comes from the source repository
    commands=["./test'sh"]
)
```

------
#### [ Java ]

```
final Artifact sourceArtifact = new Artifact();
                
final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
        // ...source and build information here
        .build();

ShellScriptAction validationAction = ShellScriptAction.Builder.create()
        .actionName("TestUsingSourceArtifact")
        .additionalArtifacts(Arrays.asList(sourceArtifact))
        // 'test.sh' comes from the source repository
        .commands(Arrays.asList("./test.sh"))
        .build();
```

------
#### [ C\# ]

```
Artifact_ sourceArtifact = new Artifact_();

var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
{
    // define your pipeline
});

var validationAction = new ShellScriptAction(new ShellScriptActionProps {
    ActionName = "TestUsingSourceArtifact",
    AdditionalArtifacts = new Artifact_[] { sourceArtifact },
    Commands = new string[] { "./test.sh" }
});
```

------

Getting the additional files from the synth step is appropriate if your tests need the compilation step that is done as part of synthesis\. On the synthesis step, specify `additionalArtifacts` to package additional subdirectories into artifacts, and use the same artifact in the `ShellScriptAction`'s `additionalArtifacts`:

------
#### [ TypeScript ]

```
// If you are using additional output artifacts from the synth step,
// they must be named.
const cloudAssemblyArtifact = new codepipeline.Artifact('CloudAsm');
const integTestsArtifact = new codepipeline.Artifact('IntegTests');

const pipeline = new CdkPipeline(this, 'Pipeline', {
  synthAction: SimpleSynthAction.standardNpmSynth({
    sourceArtifact,
    cloudAssemblyArtifact,
    buildCommand: 'npm run build',
    additionalArtifacts: [
      {
        directory: 'test',
        artifact: integTestsArtifact,
      }
    ],
  }),
  // ...
});

const validationAction = new ShellScriptAction({
  actionName: 'TestUsingBuildArtifact',
  additionalArtifacts: [integTestsArtifact],
  // 'test.js' was produced from 'test/test.ts' during the synth step
  commands: ['node ./test.js'],
});
```

------
#### [ JavaScript ]

```
// If you are using additional output artifacts from the synth step,
// they must be named.
const cloudAssemblyArtifact = new codepipeline.Artifact('CloudAsm');
const integTestsArtifact = new codepipeline.Artifact('IntegTests');

const pipeline = new CdkPipeline(this, 'Pipeline', {
  synthAction: SimpleSynthAction.standardNpmSynth({
    sourceArtifact,
    cloudAssemblyArtifact,
    buildCommand: 'npm run build',
    additionalArtifacts: [
      {
        directory: 'test',
        artifact: integTestsArtifact
      }
    ]
  })
  // ...
});

const validationAction = new ShellScriptAction({
  actionName: 'TestUsingBuildArtifact',
  additionalArtifacts: [integTestsArtifact],
  // 'test.js' was produced from 'test/test.ts' during the synth step
  commands: ['node ./test.js']
});
```

------
#### [ Python ]

```
# If you are using additional output artifacts from the synth step,
# they must be named.
cloud_assembly_artifact = code_pipeline.Artifact("CloudAsm")
integ_tests_artifact = code_pipeline.Artifact("IntegTests")

pipeline = CdkPipeline(self, "Pipeline",
    synth_action=SimpleSynthAction.standard_npm_synth(
        source_artifact=source_artifact,
        cloud_assembly_artifact=cloud_assembly_artifact,
        build_command="tsc",
        additional_artifacts=[dict(directory='test', 
            artifact=integ_tests_artifact)]
    # ...
  ))

validation_action = ShellScriptAction(
    action_name="TestUsingBuildArtifact",
    additional_artifacts=[integ_tests_artifact],
    # 'test.js' was produced from "test/test.ts" during the synth step
    commands=["node ./test.js"]
)
```

------
#### [ Java ]

```
// If you are using additional output artifacts from the synth step,
// they must be named.
final Artifact cloudAssemblyArtifact = new Artifact("IntegTests");
final Artifact integTestsArtifact = new Artifact("IntegTests");
                
final CdkPipeline pipeline = CdkPipeline.Builder.create(this, "Pipeline")
        .synthAction(SimpleSynthAction.standardNpmSynth(new StandardNpmSynthOptions.Builder()
            .sourceArtifact(sourceArtifact)
            .cloudAssemblyArtifact(cloudAssemblyArtifact)
            .buildCommand("npm run build")
            .additionalArtifacts(Arrays.asList(new AdditionalArtifact.Builder()
                .directory("test").artifact(integTestsArtifact).build()))
                .build()))
            .build();

final ShellScriptAction validationAction = ShellScriptAction.Builder.create()
    .actionName("TestUsingBuildArtifact")
    .additionalArtifacts(Arrays.asList(integTestsArtifact))
    // 'test.js' was produced from 'test/test.ts' during the synth step
    .commands(Arrays.asList("node ./test.js"))
    .build();
```

------
#### [ C\# ]

```
// If you are using additional output artifacts from the synth step,
// they must be named.
var sourceArtifact = new Artifact_("Source");
var cloudAssemblyArtifact = new Artifact_("CloudAssembly");
var integTestsArtifact = new Artifact_("IntegTests");

var pipeline = new CdkPipeline(this, "Pipeline", new CdkPipelineProps
{
    SynthAction = SimpleSynthAction.StandardNpmSynth(new StandardNpmSynthOptions
    {
        SourceArtifact = sourceArtifact,
        CloudAssemblyArtifact = cloudAssemblyArtifact,
        BuildCommand = "npm run build",
        AdditionalArtifacts = new AdditionalArtifact[]
        {
            new AdditionalArtifact
            {
                Directory = "test",
                Artifact = integTestsArtifact
            }
        }
    }),
});

var validationAction = new ShellScriptAction(new ShellScriptActionProps
{
    ActionName = "TestUsingBuildArtifact",
    AdditionalArtifacts = new Artifact_[] { integTestsArtifact },
    Commands = new string[] { "node./test.js" }
});
```

------

## Security notes<a name="cdk_pipeline_security"></a>

Any form of continuous delivery has inherent security risks\. Under the AWS [Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/), you are responsible for the security of your information in the AWS cloud\. The CDK Pipelines library gives you a head start by incorporating secure defaults and modeling best practices, but by its very nature a library that needs a high level of access to fulfill its intended purpose cannot assure complete security\. There are many attack vectors outside of AWS and your organization\.

In particular, keep in mind the following\.
+ Be mindful of the software you depend on\. Vet all third\-party software you run on your build machine, as it has the ability to change the infrastructure that gets deployed\. 
+ Use dependency locking to prevent accidental upgrades\. The default `CdkSynth` that come with CDK Pipelines respect `package-lock.json` and `yarn.lock` to ensure your dependencies are the ones you expect\.
+ Credentials for production environments should be short\-lived\. After bootstrapping and initial provisioning, there is no need for developers to have account credentials; all changes can be deployed through the pipeline\. Eliminate the possibility of credentials leaking by not needing them in the first place\!

## Troubleshooting tips<a name="cdk_pipeline_troubleshooting"></a>

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

## Known issues and limitations<a name="cdk_pipeline_issues"></a>

We're currently aware of the following issues with CDK Pipelines\.
+ Context queries are not supported; `Vpc.fromLookup()` and similar functions do not work\.
+ Console links to other accounts will not work\. The AWS CodePipeline console assumes links are relative to the current account\. You cannot click through to a AWS CloudFormation stack in a different account\.
+ If a changeset failed to apply, the pipeline is not retried\. The pipeline must be restarted manually from the top by clicking **Release Change**\.
+ A stack that failed to deploy must be deleted manually using the CloudFormation console before starting the pipeline again by clicking **Release Change**\.

Please [report any other issues](https://github.com/aws/aws-cdk/issues/new/choose) you encounter\.