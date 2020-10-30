# Creating a pipeline using the AWS CDK<a name="codepipeline_example"></a>

The AWS CDK lets you easily define applications in the AWS Cloud using your programming language of choice\. But creating an application is just the start of the journey\. You also want to make changes to it and deploy them\. You can do this through the **Code** suite of tools: AWS CodeCommit, AWS CodeBuild, AWS CodeDeploy, and AWS CodePipeline\. Together, they allow you to build what's called a [deployment pipeline](https://aws.amazon.com/getting-started/tutorials/continuous-deployment-pipeline/) for your application\. This example shows how to deploy an AWS Lambda function using such a pipeline\.

## How it works<a name="codepipeline_example.how"></a>

Our application contains two AWS CDK stacks\. The first stack, `PipelineStack`, defines the pipeline itself\. The second, `LambdaStack`, is used to deploy the Lambda function\.

The key to this example is that you deploy `PipelineStack` from your own workstation, but `LambdaStack` is deployed by the pipeline; you never deploy it yourself\.

Since the `LambdaStack` is deployed by the pipeline, it must be available to the pipeline \(along with the Lambda code\)\. Therefore, this app and the Lambda function are stored in a CodeCommit repository\.

The `PipelineStack` contains the definitions of the pipeline, which includes build steps for both the Lambda function and the `LambdaStack`\.

## Prerequisites<a name="codepipeline_example.prerequisites"></a>

Beyond having the AWS CDK set up and configured, your workstation needs to be able to push to AWS CodeCommit using Git, which means you need some way of identifying yourself to CodeCommit\. The easiest way to do this is to configure Git credentials for an IAM user, as described in [Setup for HTTPS users using Git credentials](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html)\.

You can also use the `git-remote-codecommit` Git add\-on or other methods of connecting and authenticating [supported by CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up.html)\.

Also, make sure you have issued `cdk bootstrap`, as the Amazon S3 bucket in the bootstrap stack is required to deploy a Lambda function with the AWS CDK\.

## Setting up the project<a name="codepipeline_example.setup"></a>

To set up a new AWS CDK project in CodeCommit;

1. [Create a new CodeCommit repository](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-create-repository.html) named `pipeline` using the CodeCommit console or the AWS CLI\.

   if you already have a CodeCommit repository named `pipeline`, you can use another name\. Just make sure you clone it to a directory named pipeline on your local system\.

1. Clone this new repository to your local computer in a directory named pipeline\. If you are authenticating with an IAM user with Git credentials, copy the HTTPS URL from the CodeCommit console\. \(Other authentication methods require a different URL\.\)

   ```
   git clone CODECOMMIT-REPO-URL pipeline
   ```

   Enter your credentials if prompted for them\. 
**Note**  
During cloning, Git will warn you that you appear to have cloned an empty repository; this is normal and expected\.

1. Change to the pipeline directory and initialize it as a new CDK project, then install the AWS Construct Libraries we'll use in our app\.

------
#### [ TypeScript ]

   ```
   cd pipeline
   cdk init --language typescript
   npm install @aws-cdk/aws-codedeploy @aws-cdk/aws-lambda @aws-cdk/aws-codebuild @aws-cdk/aws-codepipeline
   npm install @aws-cdk/aws-codecommit @aws-cdk/aws-codepipeline-actions @aws-cdk/aws-s3
   ```

------
#### [ JavaScript ]

   ```
   cd pipeline
   cdk init ‐-language javascript
   npm install @aws-cdk/aws-codedeploy @aws-cdk/aws-lambda @aws-cdk/aws-codebuild @aws-cdk/aws-codepipeline
   npm install @aws-cdk/aws-codecommit @aws-cdk/aws-codepipeline-actions @aws-cdk/aws-s3
   ```

------
#### [ Python ]

   A couple of commands differ between Windows and Mac OS X or Linux\. 

   **Mac OS X/Linux**

   ```
   cd pipeline
   cdk init --language python
   source .venv/bin/activate
   git commit -m "project started"
   pip install -r requirements.txt
   pip install aws_cdk.aws_codedeploy aws_cdk.aws_lambda aws_cdk.aws_codebuild aws_cdk.aws_codepipeline
   pip install aws_cdk.aws_codecommit aws_cdk.aws_codepipeline_actions aws_cdk.aws_s3
   pip freeze | grep -v '-e git' > requirements.txt
   ```

   **Windows**

   ```
   cd pipeline
   cdk init --language python
   .venv\Scripts\activate.bat
   pip install -r requirements.txt
   pip install aws_cdk.aws_codedeploy aws_cdk.aws_lambda aws_cdk.aws_codebuild aws_cdk.aws_codepipeline
   pip install aws_cdk.aws_codecommit aws_cdk.aws_codepipeline_actions aws_cdk.aws_s3
   pip freeze | find /V "-e git" > requirements.txt
   ```

------
#### [ Java ]

   ```
   cd pipeline
   cdk init --language java
   ```

   You can import the resulting Maven project into your Java IDE\.

   Using the Maven integration in your IDE \(for example, in Eclipse, right\-click the project and choose **Maven** > **Add Dependency**\), add the following packages in the group `software.amazon.awscdk`\.

   ```
   lambda
   codedeploy
   codebuild
   codecommit
   codepipeline
   codepipeline-actions
   s3
   ```

   Alternatively, add `<dependency>` elements like the following to `pom.xml`\. You can copy the existing dependency for the AWS CDK core module and modify it\. For example, a dependency for the AWS Lambda module looks like this\.

   ```
           <dependency>
               <groupId>software.amazon.awscdk</groupId>
               <artifactId>lambda</artifactId>
               <version>${cdk.version}</version>
           </dependency>
   ```

------
#### [ C\# ]

   ```
   cd pipeline
   cdk init --language csharp
   ```

   You can open the file `src/Pipeline.sln` in Visual Studio\.

   Choose **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio and add the following packages\.

   ```
   Amazon.CDK.AWS.CodeDeploy
   Amazon.CDK.AWS.CodeBuild
   Amazon.CDK.AWS.CodeCommit
   Amazon.CDK.AWS.CodePipeline
   Amazon.CDK.AWS.CodePipeline.Actions
   Amazon.CDK.AWS.Lambda
   Amazon.CDK.AWS.S3
   ```

------

1. If a directory named `test` exists, delete it\. We won't be using it in this example, and some of the code in the tests will cause errors because of other changes we'll be making\.

------
#### [ Mac OS X/Linux ]

   ```
   rm -rf test
   ```

------
#### [ Windows ]

   ```
   cd test
   del /f /q /s *.*
   cd ..
   rmdir test
   ```

------

1. Stage all the files in the directory, commit them to your local repository, and push to CodeCommit\.

   ```
   git add --all
   git commit -m "initial commit"
   git push
   ```

## Add Lambda code<a name="codepipeline_example_lambda"></a>

1. Create a directory for your AWS Lambda code\.

   ```
   mkdir lambda
   ```

1. Place your AWS Lambda function in the new directory\. Our CDK app expects a Lambda function written in TypeScript, with a main file of `index.ts` and a main function named `main()`, regardless of what language the rest of the app is written in\. The function will be built \(transpiled to JavaScript\) by the TypeScript compiler as part of the pipeline\. Some changes will be needed in the Lambda build process if your function is written in another language\.

   If you don't have a function handy to play with, this one will do:

   ```
   // index.ts
   const GREETING = "Hello, AWS!";
   export async function main(event: any, context: any) {
     console.log(GREETING);
     return GREETING;
   }
   ```

1. Commit your changes and push\.

   ```
   git add --all
   git commit -m "add lambda function"
   git push
   ```

## Define Lambda stack<a name="codepipeline_example_lambdastack"></a>

Let's define the AWS CloudFormation stack that will create the Lambda function, the stack that we'll deploy in our pipeline\. We'll create a new file to hold this stack\.

We need some way to get a reference to the Lambda function we'll be deploying\. This code is built by the pipeline, and the pipeline passes us a reference to it as AWS CloudFormation parameters\. We get it using the `fromCfnParameters()` method and store it as an attribute named `lambdaCode`, where it can be picked up by the deployment stage of the pipeline\.

The example also uses the CodeDeploy support for blue\-green deployments to Lambda, transferring traffic to the new version in 10\-percent increments every minute\. As blue\-green deployment can only operate on aliases, not on the function directly, we create an alias for our function, named `Prod`\.

The alias uses a Lambda version obtained using the function's `currentVersion` property\. This ensures that every invocation of the AWS CDK code publishes a new version of the function\.

If the Lambda function needs any other resources when executing, such as an Amazon S3 bucket, Amazon DynamoDB table, or Amazon API Gateway, you'd declare those resources here\.

------
#### [ TypeScript ]

File: `lib/lambda-stack.ts`

```
import * as codedeploy from '@aws-cdk/aws-codedeploy';
import * as lambda from '@aws-cdk/aws-lambda';
import { App, Stack, StackProps } from '@aws-cdk/core';
      
export class LambdaStack extends Stack {
  public readonly lambdaCode: lambda.CfnParametersCode;
      
  constructor(app: App, id: string, props?: StackProps) {
    super(app, id, props);
      
    this.lambdaCode = lambda.Code.fromCfnParameters();
      
    const func = new lambda.Function(this, 'Lambda', {
      code: this.lambdaCode,
      handler: 'index.main',
      runtime: lambda.Runtime.NODEJS_10_X,
    });
      
    const alias = new lambda.Alias(this, 'LambdaAlias', {
      aliasName: 'Prod',
      version: func.currentVersion,
    });
      
    new codedeploy.LambdaDeploymentGroup(this, 'DeploymentGroup', {
      alias,
      deploymentConfig: codedeploy.LambdaDeploymentConfig.LINEAR_10PERCENT_EVERY_1MINUTE,
    });
  }
}
```

------
#### [ JavaScript ]

File: `lib/lambda-stack.js`

```
const codedeploy = require('@aws-cdk/aws-codedeploy');
const lambda = require('@aws-cdk/aws-lambda');
const { Stack } = require('@aws-cdk/core');
      
class LambdaStack extends Stack {

  constructor(app, id, props) {
    super(app, id, props);
      
    this.lambdaCode = lambda.Code.fromCfnParameters();
      
    const func = new lambda.Function(this, 'Lambda', {
      code: this.lambdaCode,
      handler: 'index.main',
      runtime: lambda.Runtime.NODEJS_10_X
    });
      
    const alias = new lambda.Alias(this, 'LambdaAlias', {
      aliasName: 'Prod',
      version: func.currentVersion
    });
      
    new codedeploy.LambdaDeploymentGroup(this, 'DeploymentGroup', {
      alias,
      deploymentConfig: codedeploy.LambdaDeploymentConfig.LINEAR_10PERCENT_EVERY_1MINUTE
    });
  }
}

module.exports = { LambdaStack }
```

------
#### [ Python ]

File: `pipeline/lambda_stack.py`

```
from aws_cdk import core, aws_codedeploy as codedeploy, aws_lambda as lambda_

class LambdaStack(core.Stack):
  def __init__(self, app: core.App, id: str, **kwargs):
    super().__init__(app, id, **kwargs)

    self.lambda_code = lambda_.Code.from_cfn_parameters()
      
    func = lambda_.Function(self, "Lambda",
                            code=self.lambda_code,
                            handler="index.main",
                            runtime=lambda_.Runtime.NODEJS_10_X,
    )
      
    alias = lambda_.Alias(self, "LambdaAlias", alias_name="Prod",
                            version=func.current_version)
      
    codedeploy.LambdaDeploymentGroup(self, "DeploymentGroup",
        alias=alias,
        deployment_config=
            codedeploy.LambdaDeploymentConfig.LINEAR_10_PERCENT_EVERY_1_MINUTE
    )
```

------
#### [ Java ]

File: `src/main/java/com/myorg/LambdaStack.java`

```
package com.myorg;

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;

import software.amazon.awscdk.services.codedeploy.*;
import software.amazon.awscdk.services.lambda.*;
import software.amazon.awscdk.services.lambda.Runtime;

public class LambdaStack extends Stack {

    // private attribute to hold our Lambda's code, with public getters
    private CfnParametersCode lambdaCode;
    
    public CfnParametersCode getLambdaCode() {
        return lambdaCode;
    }
    
    // Constructor without props argument
    public LambdaStack(final App scope, final String id) {
        this(scope, id, null);
    }
    
    public LambdaStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);
        
        lambdaCode = CfnParametersCode.fromCfnParameters();
        
        Function func = Function.Builder.create(this, "Lambda")
                .code(lambdaCode)
                .handler("index.main")
                .runtime(Runtime.NODEJS_10_X).build();
        
        Version version = func.getCurrentVersion();
        Alias alias = Alias.Builder.create(this, "LambdaAlias")
                .aliasName("LambdaAlias")
                .version(version).build();

        LambdaDeploymentGroup.Builder.create(this, "DeploymentGroup")
                .alias(alias)
                .deploymentConfig(LambdaDeploymentConfig.LINEAR_10_PERCENT_EVERY_1_MINUTE).build();   
    }    
}
```

------
#### [ C\# ]

File: `src/Pipeline/LambdaStack.cs`

```
using Amazon.CDK;
using Amazon.CDK.AWS.CodeDeploy;
using Amazon.CDK.AWS.Lambda;

namespace Pipeline
{
    public class LambdaStack : Stack
    {
        public readonly CfnParametersCode lambdaCode;

        public LambdaStack(Construct scope, string id, StackProps props = null) :
            base(scope, id, props)
        {
            lambdaCode = Code.FromCfnParameters();

            var func = new Function(this, "Lambda", new FunctionProps
            {
                Code = lambdaCode,
                Handler = "index.main",
                Runtime = Runtime.NODEJS_10_X
            });

            var version = func.currentVersion;
            var alias = new Alias(this, "LambdaAlias", new AliasProps
            {
                AliasName = "Prod",
                Version = version
            });

            new LambdaDeploymentGroup(this, "DeploymentGroup", new LambdaDeploymentGroupProps
            {
                Alias = alias,
                DeploymentConfig = LambdaDeploymentConfig.LINEAR_10PERCENT_EVERY_1MINUTE
            });
        }
    }
}
```

------

## Define pipeline stack<a name="codepipeline_example_stack"></a>

Our second stack, `PipelineStack`, contains the code that defines our pipeline\.

First it needs a reference to the Lambda code it's deploying\. For that, we define a new props interface for it, `PipelineStackProps`\. This extends the standard `StackProps` and is how clients of this class \(including ourselves\) pass the Lambda code that the class needs\.

The name of the CodeCommit repo hosting our source code is also passed in the stack's props\. The `Repository.fromRepositoryName` method is a standard AWS CDK idiom for referencing a resource, such as a CodeCommit repository, that lives outside the AWS CDK code\.

The pipeline has two CodeBuild projects\. The first project synthesizes the AWS CloudFormation template to deploy the Lambda function from the AWS CDK code\. To do that, it installs the AWS CDK Toolkit using `npm`, then any dependencies, and then issues cdk synth command to produce AWS CloudFormation templates in the target directory `dist`\. The `dist/LambdaStack.template.json` file is this step's output\.

The second project builds the Lambda code\. It begins by changing the current directory to `lambda`, which is where the Lambda code lives\. It then installs any dependencies and the TypeScript compiler, then builds the code\. The output is the contents of the `node_modules` directory, plus the `index.js` file\. The Lambda runtime will call the `handler\(\)` function in this file to handle requests\.

**Tip**  
This is where you'll need some changes if you use a Lambda function written in a language other than TypeScript\.

Finally, we define our pipeline\. It has a source Action targeting the CodeCommit repository, two build Actions using the previously defined projects, and finally a deploy Action that uses AWS CloudFormation\. It takes the template generated by the AWS CDK build Project \(stored in the `LambdaStack.template.json` file\), passes it to AWS CloudFormation for deployment\. To make the Lambda build output is an input to the AWS CloudFormation action, we pass it in the `extraInputs` property\.

We also change the name of the stack that will be deployed, from `LambdaStack` to `LambdaDeploymentStack`\. This isn't required; it's just an example of how you'd do this if you wanted\.

------
#### [ TypeScript ]

File: `lib/pipeline-stack.ts`

```
import * as codebuild from '@aws-cdk/aws-codebuild';
import * as codecommit from '@aws-cdk/aws-codecommit';
import * as codepipeline from '@aws-cdk/aws-codepipeline';
import * as codepipeline_actions from '@aws-cdk/aws-codepipeline-actions';
import * as lambda from '@aws-cdk/aws-lambda';
import * as s3 from '@aws-cdk/aws-s3';
import { App, Stack, StackProps } from '@aws-cdk/core';

export interface PipelineStackProps extends StackProps {
  readonly lambdaCode: lambda.CfnParametersCode;
  readonly repoName: string
}

export class PipelineStack extends Stack {
  constructor(app: App, id: string, props: PipelineStackProps) {
    super(app, id, props);

    const code = codecommit.Repository.fromRepositoryName(this, 'ImportedRepo',
      props.repoName);

    const cdkBuild = new codebuild.PipelineProject(this, 'CdkBuild', {
      buildSpec: codebuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            commands: 'npm install',
          },
          build: {
            commands: [
              'npm run build',
              'npm run cdk synth -- -o dist'
            ],
          },
        },
        artifacts: {
          'base-directory': 'dist',
          files: [
            'LambdaStack.template.json',
          ],
        },
      }),
      environment: {
        buildImage: codebuild.LinuxBuildImage.STANDARD_2_0,
      },
    });
    const lambdaBuild = new codebuild.PipelineProject(this, 'LambdaBuild', {
      buildSpec: codebuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            commands: [
              'cd lambda',
              'npm install',
            ],
          },
          build: {
            commands: 'npm run build',
          },
        },
        artifacts: {
          'base-directory': 'lambda',
          files: [
            'index.js',
            'node_modules/**/*',
          ],
        },
      }),
      environment: {
        buildImage: codebuild.LinuxBuildImage.STANDARD_2_0,
      },
    });

    const sourceOutput = new codepipeline.Artifact();
    const cdkBuildOutput = new codepipeline.Artifact('CdkBuildOutput');
    const lambdaBuildOutput = new codepipeline.Artifact('LambdaBuildOutput');
    new codepipeline.Pipeline(this, 'Pipeline', {
      stages: [
        {
          stageName: 'Source',
          actions: [
            new codepipeline_actions.CodeCommitSourceAction({
              actionName: 'CodeCommit_Source',
              repository: code,
              output: sourceOutput,
            }),
          ],
        },
        {
          stageName: 'Build',
          actions: [
            new codepipeline_actions.CodeBuildAction({
              actionName: 'Lambda_Build',
              project: lambdaBuild,
              input: sourceOutput,
              outputs: [lambdaBuildOutput],
            }),
            new codepipeline_actions.CodeBuildAction({
              actionName: 'CDK_Build',
              project: cdkBuild,
              input: sourceOutput,
              outputs: [cdkBuildOutput],
            }),
          ],
        },
        {
          stageName: 'Deploy',
          actions: [
            new codepipeline_actions.CloudFormationCreateUpdateStackAction({
              actionName: 'Lambda_CFN_Deploy',
              templatePath: cdkBuildOutput.atPath('LambdaStack.template.json'),
              stackName: 'LambdaDeploymentStack',
              adminPermissions: true,
              parameterOverrides: {
                ...props.lambdaCode.assign(lambdaBuildOutput.s3Location),
              },
              extraInputs: [lambdaBuildOutput],
            }),
          ],
        },
      ],
    });
  }
}
```

------
#### [ JavaScript ]

File: `lib/pipeline-stack.js`

```
const codebuild = require('@aws-cdk/aws-codebuild');
const codecommit = require('@aws-cdk/aws-codecommit');
const codepipeline = require('@aws-cdk/aws-codepipeline');
const codepipeline_actions = require('@aws-cdk/aws-codepipeline-actions');

const { Stack } = require('@aws-cdk/core');

class PipelineStack extends Stack {
  constructor(app, id, props) {
    super(app, id, props);

    const code = codecommit.Repository.fromRepositoryName(this, 'ImportedRepo',
      props.repoName);

    const cdkBuild = new codebuild.PipelineProject(this, 'CdkBuild', {
      buildSpec: codebuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            commands: 'npm install'
          },
          build: {
            commands: 'npm run cdk synth -- -o dist'
          }
        },
        artifacts: {
          'base-directory': 'dist',
          files: [
            'LambdaStack.template.json'
          ]
        }
      }),
      environment: {
        buildImage: codebuild.LinuxBuildImage.STANDARD_2_0
      }
    });

    const lambdaBuild = new codebuild.PipelineProject(this, 'LambdaBuild', {
      buildSpec: codebuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            commands: [
              'cd lambda',
              'npm install',
              'npm install typescript'
            ]
          },
          build: {
            commands: 'npx tsc index.ts'
          }
        },
        artifacts: {
          'base-directory': 'lambda',
          files: [
            'index.js',
            'node_modules/**/*'
          ]
        }
      }),
      environment: {
        buildImage: codebuild.LinuxBuildImage.STANDARD_2_0
      }
    });

    const sourceOutput = new codepipeline.Artifact();
    const cdkBuildOutput = new codepipeline.Artifact('CdkBuildOutput');
    const lambdaBuildOutput = new codepipeline.Artifact('LambdaBuildOutput');
    new codepipeline.Pipeline(this, 'Pipeline', {
      stages: [
        {
          stageName: 'Source',
          actions: [
            new codepipeline_actions.CodeCommitSourceAction({
              actionName: 'CodeCommit_Source',
              repository: code,
              output: sourceOutput
            })
          ]
        },
        {
          stageName: 'Build',
          actions: [
            new codepipeline_actions.CodeBuildAction({
              actionName: 'Lambda_Build',
              project: lambdaBuild,
              input: sourceOutput,
              outputs: [lambdaBuildOutput]
            }),
            new codepipeline_actions.CodeBuildAction({
              actionName: 'CDK_Build',
              project: cdkBuild,
              input: sourceOutput,
              outputs: [cdkBuildOutput]
            })
          ]
        },
        {
          stageName: 'Deploy',
          actions: [
            new codepipeline_actions.CloudFormationCreateUpdateStackAction({
              actionName: 'Lambda_CFN_Deploy',
              templatePath: cdkBuildOutput.atPath('LambdaStack.template.json'),
              stackName: 'LambdaDeploymentStack',
              adminPermissions: true,
              parameterOverrides: {
                ...props.lambdaCode.assign(lambdaBuildOutput.s3Location)
              },
              extraInputs: [lambdaBuildOutput]
            })
          ]
        }
      ]
    });
  }
}

module.exports = { PipelineStack }
```

------
#### [ Python ]

File: `pipeline/pipeline_stack.py`

```
from aws_cdk import (core, aws_codebuild as codebuild,
                     aws_codecommit as codecommit,
                     aws_codepipeline as codepipeline,
                     aws_codepipeline_actions as codepipeline_actions,
                     aws_lambda as lambda_, aws_s3 as s3)

class PipelineStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, *, repo_name: str=None,
                 lambda_code: lambda_.CfnParametersCode=None, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        code = codecommit.Repository.from_repository_name(self, "ImportedRepo",
                  repo_name)

        cdk_build = codebuild.PipelineProject(self, "CdkBuild",
                        build_spec=codebuild.BuildSpec.from_object(dict(
                            version="0.2",
                            phases=dict(
                                install=dict(
                                    commands=[
                                        "npm install aws-cdk",
                                        "npm update",
                                        "python -m pip install -r requirements.txt"
                                    ]),
                                build=dict(commands=[
                                    "npx cdk synth -o dist"])),
                            artifacts={
                                "base-directory": "dist",
                                "files": [
                                    "LambdaStack.template.json"]},
                            environment=dict(buildImage=
                                codebuild.LinuxBuildImage.STANDARD_2_0))))

        lambda_build = codebuild.PipelineProject(self, 'LambdaBuild',
                        build_spec=codebuild.BuildSpec.from_object(dict(
                            version="0.2",
                            phases=dict(
                                install=dict(
                                    commands=[
                                        "cd lambda",
                                        "npm install",
                                        "npm install typescript"]),
                                build=dict(
                                    commands=[
                                        "npx tsc index.ts"])),
                            artifacts={
                                "base-directory": "lambda",
                                "files": [
                                    "index.js",
                                    "node_modules/**/*"]},
                            environment=dict(buildImage=
                                codebuild.LinuxBuildImage.STANDARD_2_0))))

        source_output = codepipeline.Artifact()
        cdk_build_output = codepipeline.Artifact("CdkBuildOutput")
        lambda_build_output = codepipeline.Artifact("LambdaBuildOutput")

        lambda_location = lambda_build_output.s3_location

        codepipeline.Pipeline(self, "Pipeline",
            stages=[
                codepipeline.StageProps(stage_name="Source",
                    actions=[
                        codepipeline_actions.CodeCommitSourceAction(
                            action_name="CodeCommit_Source",
                            repository=code,
                            output=source_output)]),
                codepipeline.StageProps(stage_name="Build",
                    actions=[
                        codepipeline_actions.CodeBuildAction(
                            action_name="Lambda_Build",
                            project=lambda_build,
                            input=source_output,
                            outputs=[lambda_build_output]),
                        codepipeline_actions.CodeBuildAction(
                            action_name="CDK_Build",
                            project=cdk_build,
                            input=source_output,
                            outputs=[cdk_build_output])]),
                codepipeline.StageProps(stage_name="Deploy",
                    actions=[
                        codepipeline_actions.CloudFormationCreateUpdateStackAction(
                            action_name="Lambda_CFN_Deploy",
                            template_path=cdk_build_output.at_path(
                                "LambdaStack.template.json"),
                            stack_name="LambdaDeploymentStack",
                            admin_permissions=True,
                            parameter_overrides=dict(
                                lambda_code.assign(
                                    bucket_name=lambda_location.bucket_name,
                                    object_key=lambda_location.object_key,
                                    object_version=lambda_location.object_version)),
                            extra_inputs=[lambda_build_output])])
                ]
            )
```

------
#### [ Java ]

File: `src/main/java/com/myorg/PipelineStack.java`

```
package com.myorg;

import java.util.Arrays;
import java.util.List;
import java.util.HashMap;

import software.amazon.awscdk.core.*;
import software.amazon.awscdk.services.codebuild.*;
import software.amazon.awscdk.services.codecommit.*;
import software.amazon.awscdk.services.codepipeline.*;
import software.amazon.awscdk.services.codepipeline.StageProps;
import software.amazon.awscdk.services.codepipeline.actions.*;
import software.amazon.awscdk.services.lambda.*;

public class PipelineStack extends Stack {
    // alternate constructor for calls without props.
    // lambdaCode and repoName are both required. 
    public PipelineStack(final App scope, final String id, 
            final CfnParametersCode lambdaCode, final String repoName) {
        this(scope, id, null, lambdaCode, repoName);
    }
    
    @SuppressWarnings("serial")
    public PipelineStack(final App scope, final String id, final StackProps props,
        final CfnParametersCode lambdaCode, final String repoName) {
        super(scope, id, props);

        IRepository code = Repository.fromRepositoryName(this, "ImportedRepo", repoName);

        PipelineProject cdkBuild = PipelineProject.Builder.create(this, "CDKBuild") 
                    .buildSpec(BuildSpec.fromObject(new HashMap<String, Object>() {{
                        put("version", "0.2");
                        put("phases", new HashMap<String, Object>() {{
                            put("install", new HashMap<String, String>() {{
                                put("commands", "npm install aws-cdk");
                            }});
                            put("build", new HashMap<String, Object>() {{
                                put("commands", Arrays.asList("mvn compile -q -DskipTests",
                                        "npx cdk synth -o dist"));
                            }});
                        }});
                        put("artifacts", new HashMap<String, Object>() {{
                            put("base-directory", "dist");
                            put("files", Arrays.asList("LambdaStack.template.json"));
                        }});
                    }}))
                    .environment(BuildEnvironment.builder().buildImage(
                            LinuxBuildImage.STANDARD_2_0).build())
                    .build();

        PipelineProject lambdaBuild = PipelineProject.Builder.create(this, "LambdaBuild") 
                    .buildSpec(BuildSpec.fromObject(new HashMap<String, Object>() {{
                        put("version", "0.2");
                        put("phases", new HashMap<String, Object>() {{ 
                            put("install", new HashMap<String, List<String>>() {{
                                put("commands", Arrays.asList("cd lambda", "npm install",
                                        "npm install typescript"));
                            }});
                            put("build", new HashMap<String, List<String>>() {{
                                put("commands", Arrays.asList("npx tsc index.ts"));
                            }});
                        }});
                        put("artifacts", new HashMap<String, Object>() {{
                            put("base-directory", "lambda");
                            put("files", Arrays.asList("index.js", "node_modules/**/*"));
                        }});
                    }}))
                    .environment(BuildEnvironment.builder().buildImage(
                            LinuxBuildImage.STANDARD_2_0).build())
                    .build();

        Artifact sourceOutput = new Artifact();
        Artifact cdkBuildOutput = new Artifact("CdkBuildOutput");
        Artifact lambdaBuildOutput = new Artifact("LambdaBuildOutput");

        Pipeline.Builder.create(this, "Pipeline")
                .stages(Arrays.asList(
                    StageProps.builder()
                        .stageName("Source")
                        .actions(Arrays.asList(
                            CodeCommitSourceAction.Builder.create()
                                .actionName("Source")
                                .repository(code)
                                .output(sourceOutput)
                                .build()))
                        .build(),
                    StageProps.builder()
                        .stageName("Build")
                        .actions(Arrays.asList(
                            CodeBuildAction.Builder.create()
                                .actionName("Lambda_Build")
                                .project(lambdaBuild)
                                .input(sourceOutput)
                                .outputs(Arrays.asList(lambdaBuildOutput)).build(),
                            CodeBuildAction.Builder.create()
                                .actionName("CDK_Build")
                                .project(cdkBuild)
                                .input(sourceOutput)
                                .outputs(Arrays.asList(cdkBuildOutput))
                                .build()))
                        .build(),
                    StageProps.builder()
                        .stageName("Deploy")
                        .actions(Arrays.asList(
                             CloudFormationCreateUpdateStackAction.Builder.create()
                                         .actionName("Lambda_CFN_Deploy")
                                         .templatePath(cdkBuildOutput.atPath("LambdaStack.template.json"))
                                         .adminPermissions(true)
                                         .parameterOverrides(lambdaCode.assign(lambdaBuildOutput.getS3Location()))
                                         .extraInputs(Arrays.asList(lambdaBuildOutput))
                                         .stackName("LambdaDeploymentStack")
                                         .build()))
                        .build()))
                .build();
    }
}
```

------
#### [ C\# ]

File: `src/Pipeline/PipelineStack.cs`

```
using Amazon.CDK;
using Amazon.CDK.AWS.CodeBuild;
using Amazon.CDK.AWS.CodeCommit;
using Amazon.CDK.AWS.CodePipeline;
using Amazon.CDK.AWS.CodePipeline.Actions;
using Amazon.CDK.AWS.Lambda;
using System.Collections.Generic;

namespace Pipeline
{
    public class PipelineStackProps : StackProps
    {
        public CfnParametersCode LambdaCode { get; set; }
        public string RepoName { get; set; }
    }

    public class PipelineStack : Stack
    {
        public PipelineStack(Construct scope, string id, PipelineStackProps props = null) :
            base(scope, id, props)
        {
            var code = Repository.FromRepositoryName(this, "ImportedRepo", props.RepoName);

            var cdkBuild = new PipelineProject(this, "CDKBuild", new PipelineProjectProps
            {
                BuildSpec = BuildSpec.FromObject(new Dictionary<string, object>
                {
                    ["version"] = "0.2",
                    ["phases"] = new Dictionary<string, object>
                    {
                        ["install"] = new Dictionary<string, object>
                        {
                            ["commands"] = "npm install aws-cdk"
                        },
                        ["build"] = new Dictionary<string, object>
                        {
                            ["commands"] = "npx cdk synth -o dist"
                        }
                    },
                    ["artifacts"] = new Dictionary<string, object>
                    {
                        ["base-directory"] = "dist",
                        ["files"] = new string[]
                        {
                            "LambdaStack.template.json"
                        }
                    }
                }),
                Environment = new BuildEnvironment
                {
                    BuildImage = WindowsBuildImage.WINDOWS_BASE_2_0
                }
            });

            var lambdaBuild = new PipelineProject(this, "LambdaBuild", new PipelineProjectProps
            {
                BuildSpec = BuildSpec.FromObject(new Dictionary<string, object>
                {
                    ["version"] = "0.2",
                    ["phases"] = new Dictionary<string, object>
                    {
                        ["install"] = new Dictionary<string, object>
                        {
                            ["commands"] = new string[]
                            {
                                "cd lambda",
                                "npm install",
                                "npm install typescript"
                            }
                        },
                        ["build"] = new Dictionary<string, string>
                        {
                            ["commands"] = "npx tsc index.ts"
                        }
                    },
                    ["artifacts"] = new Dictionary<string, object>
                    {
                        ["base-directory"] = "lambda",
                        ["files"] = new string[]
                        {
                            "index.js",
                            "node_modules/**/*"
                        }
                    }
                }),
                Environment = new BuildEnvironment
                {
                    BuildImage = LinuxBuildImage.STANDARD_2_0
                }
            });

            var sourceOutput = new Artifact_();
            var cdkBuildOutput = new Artifact_("CdkBuildOutput");
            var lambdaBuildOutput = new Artifact_("LambdaBuildOutput");

            new Amazon.CDK.AWS.CodePipeline.Pipeline(this, "Pipeline", new PipelineProps
            {
                Stages = new[]
                {
                    new Amazon.CDK.AWS.CodePipeline.StageProps
                    {
                        StageName = "Source",
                        Actions = new []
                        {
                            new CodeCommitSourceAction(new CodeCommitSourceActionProps
                            {
                                ActionName = "Source",
                                Repository = code,
                                Output = sourceOutput
                            })
                        }
                    },
                    new Amazon.CDK.AWS.CodePipeline.StageProps
                    {
                        StageName = "Build",
                        Actions = new []
                        {
                            new CodeBuildAction(new CodeBuildActionProps
                            {
                                ActionName = "Lambda_Build",
                                Project = lambdaBuild,
                                Input = sourceOutput,
                                Outputs = new [] { lambdaBuildOutput }
                            }),
                            new CodeBuildAction(new CodeBuildActionProps
                            {
                                ActionName = "CDK_Build",
                                Project = cdkBuild,
                                Input = sourceOutput,
                                Outputs = new [] { cdkBuildOutput }
                            })
                        }
                    },
                    new Amazon.CDK.AWS.CodePipeline.StageProps
                    {
                        StageName = "Deploy",
                        Actions = new []
                        {
                            new CloudFormationCreateUpdateStackAction(new CloudFormationCreateUpdateStackActionProps {
                                ActionName = "Lambda_CFN_Deploy",
                                TemplatePath = cdkBuildOutput.AtPath("LambdaStack.template.json"),
                                StackName = "LambdaDeploymentStack",
                                AdminPermissions = true,
                                ParameterOverrides = props.LambdaCode.Assign(lambdaBuildOutput.S3Location),
                                ExtraInputs = new [] { lambdaBuildOutput }
                            })
                        }
                    }
                }
            });
        }
    }
}
```

------

## Main program<a name="codepipeline_example_main"></a>

Finally, we have our main AWS CDK application file\.

**Note**  
If you didn't name your new CodeCommit repository `pipeline`, here's where you'd change it\. Just edit the value of `CODECOMMIT_REPO_NAME`\.

This code is straightforward: it first instantiates the `LambdaStack` class as `LambdaStack`, which is what the AWS CDK build in the pipeline expects\. Then it instantiates the `PipelineStack` class, passing the Lambda code from the `LambdaStack` object\.

------
#### [ TypeScript ]

File: `bin/pipeline.ts`

```
#!/usr/bin/env node

const CODECOMMIT_REPO_NAME = "pipeline";

import { App } from '@aws-cdk/core';
import { LambdaStack } from '../lib/lambda-stack';
import { PipelineStack } from '../lib/pipeline-stack';

const app = new App();

const lambdaStack = new LambdaStack(app, 'LambdaStack');
new PipelineStack(app, 'PipelineDeployingLambdaStack', {
  lambdaCode: lambdaStack.lambdaCode,
  repoName: CODECOMMIT_REPO_NAME
});

app.synth();
```

------
#### [ JavaScript ]

File: `bin/pipeline.js`

```
#!/usr/bin/env node

const CODECOMMIT_REPO_NAME = "pipeline";

const { App } = require('@aws-cdk/core');
const { LambdaStack } = require('../lib/lambda-stack');
const { PipelineStack } = require('../lib/pipeline-stack');

const app = new App();

const lambdaStack = new LambdaStack(app, 'LambdaStack');
new PipelineStack(app, 'PipelineDeployingLambdaStack', {
  lambdaCode: lambdaStack.lambdaCode,
  repoName: CODECOMMIT_REPO_NAME
});

app.synth();
```

------
#### [ Python ]

File: `app.py`

```
#!/usr/bin/env python3

CODECOMMIT_REPO_NAME = "pipeline"

from aws_cdk import core

from pipeline.pipeline_stack import PipelineStack
from pipeline.lambda_stack import LambdaStack

app = core.App()

lambda_stack = LambdaStack(app, "LambdaStack")

PipelineStack(app, "PipelineDeployingLambdaStack",
    lambda_code=lambda_stack.lambda_code,
    repo_name=CODECOMMIT_REPO_NAME)

app.synth()
```

------
#### [ Java ]

File: `src/main/java/com/myorg/PipelineApp.java`

```
package com.myorg;

import software.amazon.awscdk.core.*;

public class PipelineApp {
    static final String CODECOMMIT_REPO_NAME = "pipeline";

    public static void main(final String[] argv) {
        App app = new App();

        LambdaStack lambdaStack = new LambdaStack(app, "LambdaStack");
        new PipelineStack(app, "PipelineStack", 
                lambdaStack.getLambdaCode(), CODECOMMIT_REPO_NAME);

        app.synth();
    }
}
```

------
#### [ C\# ]

File: `src/Pipeline/Program.cs`

```
using Amazon.CDK;

namespace Pipeline
{
    class Program
    {
        const string CODECOMMIT_REPO_NAME = "pipeline";

        static void Main(string[] args)
        {
            var app = new App();

            var lambdaStack = new LambdaStack(app, "LambdaStack");
            new PipelineStack(app, "PipelineDeployingLambdaStack", new PipelineStackProps
            {
                LambdaCode = lambdaStack.lambdaCode,
                RepoName = CODECOMMIT_REPO_NAME
            });

            app.Synth();
        }
    }
}
```

------

Now check this code in to Git and push it to AWS CodeCommit\.

```
git add --all
git commit -m "add CDK app"
git push
```

## Deploying the pipeline<a name="codepipeline_example_create"></a>

Now we can deploy the pipeline\.

```
cdk deploy PipelineDeployingLambdaStack
```

The name, `PipelineDeployingLambdaStack`, is the name we used when we instantiated `PipelineStack`\.

**Tip**  
Rather than typing that whole name out, this is a good place to use a wildcard\! Put quotes around the name pattern to prevent the shell from tyring to expand it\.  

```
cdk deploy "Pipe*"
```

You'll be asked to approve your stack's security changes\. Type **y** to accept them and continue with deployment\.

Don't deploy `LambdaStack`\. This stack is deployed by the pipeline, and it won't deploy without values provided by the pipeline\.

After the deployment finishes, you should have a three\-stage pipeline that looks something like the following\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/pipeline.jpg)

Try making a change to your Lambda function code and push it to the repository\. The pipeline should pick up your change, build it, and deploy it automatically, without any other action from you\.

## Cleaning up<a name="codepipeline_example_destroy"></a>

To avoid unexpected AWS charges, destroy your AWS CDK stacks after you're done with this exercise\.

Delete the `LambdaStack` first using the AWS CloudFormation console\. The IAM role needed to delete `LambdaStack` is provided by `PipelineDeployingLambdaStack`, so if you delete it first, you no longer have permission to destroy `LambdaStack`\.

Then you may delete the `PipelineDeployingLambdaStack`\. 

```
cdk destroy LambdaStack 
cdk destroy PipelineDeployingLambdaStack
```

Finally, delete your AWS CodeCommit repository from the AWS Console\.