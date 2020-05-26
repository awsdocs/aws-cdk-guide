# Creating a code pipeline using the AWS CDK<a name="codepipeline_example"></a>

This example creates a code pipeline using the AWS CDK\.

The AWS CDK enables you to easily create applications running in the AWS Cloud\. But creating the application is just the start of the journey\. You also want to make changes to it, test those changes, and finally deploy them to your stack\. The AWS CDK enables this workflow by using the **Code\*** suite of AWS tools: AWS CodeCommit, AWS CodeBuild, AWS CodeDeploy, and AWS CodePipeline\. Together, they allow you to build what's called a [deployment pipeline](https://aws.amazon.com/getting-started/tutorials/continuous-deployment-pipeline/) for your application\.

The following example shows how to deploy an AWS Lambda function in a pipeline\. Two stacks are created: one to deploy your Lambda code, and one to define a pipeline to deploy the first stack whenever your Lambda code changes\. Your Lambda code is intended to be in a AWS CodeCommit repository, although you can work through this example without any Lambda code \(the pipeline will fail, but the stack that defines it will deploy\)\.

The Lambda code must be in a directory named `Lambda` in the named AWS CodeCommit repository\. The AWS CDK code does not need to be in a repository\.

**Note**  
The Lambda function is assumed to be written in TypeScript regardless of the language you're using for your AWS CDK app\. To use this example to deploy a Lambda function written in anotehr language, you'll need to modify the pipeline\. This is outside the scope of this example\.

To set up a project like this from scratch, follow these instructions\.

------
#### [ TypeScript ]

```
mkdir pipeline
cd pipeline
cdk init --language typescript
npm install @aws-cdk/aws-codedeploy @aws-cdk/aws-lambda @aws-cdk/aws-codebuild @aws-cdk/aws-codepipeline
npm install @aws-cdk/aws-codecommit @aws-cdk/aws-codepipeline-actions @aws-cdk/aws-s3
```

------
#### [ JavaScript ]

```
mkdir pipeline
cd pipeline
cdk init ‐-language javascript
npm install @aws-cdk/aws-codedeploy @aws-cdk/aws-lambda @aws-cdk/aws-codebuild @aws-cdk/aws-codepipeline
npm install @aws-cdk/aws-codecommit @aws-cdk/aws-codepipeline-actions @aws-cdk/aws-s3
```

------
#### [ Python ]

```
mkdir pipeline
cd pipeline
cdk init --language python
source .env/bin/activate
pip install -r requirements.txt
pip install aws_cdk.aws_codedeploy aws_cdk.aws_lambda aws_cdk.aws_codebuild aws_cdk.aws_codepipeline
pip install aws_cdk.aws_codecommit aws_cdk.aws_codepipeline_actions aws_cdk.aws_s3
```

------
#### [ Java ]

```
mkdir pipeline
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

------
#### [ C\# ]

```
mkdir pipeline
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

## Lambda stack<a name="codepipeline_example_lambda"></a>

The first step is to define the AWS CloudFormation stack that will create the Lambda function\. This is the stack that we'll deploy in our pipeline\.

We'll create a new file to hold this stack\.

This class includes the `lambdaCode` \(Python: `lambda_code`\) property, which is an instance of the `CfnParametersCode` class\. This property represents the code that is supplied later by the pipeline\. Because the pipeline needs access to the object, we expose it as a public property of our class\.

The example also uses the CodeDeploy support for blue\-green deployments to Lambda, and the deployment increases the traffic to the new version in 10\-percent increments every minute\. As blue\-green deployment can only operate on aliases, not on the function directly, we create an alias for our function, named `Prod`\.

The alias uses a Lambda version obtained using the function's `currentVersion` property\. This ensures that every invocation of the AWS CDK code publishes a new version of the function\.

If the Lambda function needs any other resources when executing, such as an Amazon S3 bucket, Amazon DynamoDB table, or Amazon API Gateway, declare those resources here\.

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
      handler: 'index.handler',
      runtime: lambda.Runtime.NODEJS_10_X,
    });
      
    const version = func.latestVersion;
    const alias = new lambda.Alias(this, 'LambdaAlias', {
      aliasName: 'Prod',
      version,
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
      handler: 'index.handler',
      runtime: lambda.Runtime.NODEJS_10_X
    });
      
    const version = func.latestVersion;
    const alias = new lambda.Alias(this, 'LambdaAlias', {
      aliasName: 'Prod',
      version
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
                            handler="index.handler",
                            runtime=lambda_.Runtime.NODEJS_10_X,
    )
      
    version = func.latest_version
    alias = lambda_.Alias(self, "LambdaAlias",
                          alias_name="Prod", version=version)
      
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

import software.amazon.awscdk.services.codedeploy.LambdaDeploymentConfig;
import software.amazon.awscdk.services.codedeploy.LambdaDeploymentGroup;

import software.amazon.awscdk.services.lambda.Alias;
import software.amazon.awscdk.services.lambda.CfnParametersCode;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.lambda.Version;

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
                .handler("index.handler")
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
using System;
using Amazon.CDK;
using Amazon.CDK.AWS.CodeDeploy;
using Amazon.CDK.AWS.Lambda;

namespace Pipeline
{
    public class LambdaStack : Stack
    {
        public readonly CfnParametersCode lambdaCode;

        public LambdaStack(App app, string id, StackProps props = null) : base(app, id, props)
        {
            lambdaCode = Code.FromCfnParameters();

            var func = new Function(this, "Lambda", new FunctionProps
            {
                Code = lambdaCode,
                Handler = "index.handler",
                Runtime = Runtime.NODEJS_10_X
            });

            var version = func.LatestVersion;
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

## Pipeline stack<a name="codepipeline_example_stack"></a>

The second class, `PipelineStack`, is the stack that contains our pipeline\.

First it needs a reference to the Lambda code it's deploying\. For that, we define a new props interface for it, `PipelineStackProps`\. \(This isn't necessary in Python, where properties are instead passed as keyword arguments\.\) This extends the standard `StackProps` and is how clients of this class \(including ourselves\) pass the Lambda code that the class needs\.

Then comes the Git repository used to store the source code\. In the example, it's hosted by CodeCommit\. The `Repository.fromRepositoryName` method \(Python: `from_repository_name`\) is a standard AWS CDK idiom for referencing a resource, such as a CodeCommit repository, that lives outside the AWS CDK code\. Replace *NameOfYourCodeCommitRepository* with the name of your repository\.

The example has two CodeBuild projects\. The first project obtains the AWS CloudFormation template from the AWS CDK code\. To do that, it calls the standard install and build targets for Node\.js, and then calls the cdk synth command\. This produces AWS CloudFormation templates in the target directory `dist`\. Finally, it uses the `dist/LambdaStack.template.json` file as its output\.

The second project does a similar thing, except for the Lambda code\. Because of that, it starts by changing the current directory to `lambda`, which is where we said the Lambda code lives in the repository\. It then invokes the same install and build Node\.js targets as before\. The output is the contents of the node\_modules directory, plus the `index.js` file\. Because `index.handler` is the entry point to the Lambda code, `index.js` must exist, and must export a `handler` function\. This function is called by the Lambda runtime to handle requests\. If your Lambda code uses more files than just `index.js`, add them here\.

Finally, we create our pipeline\. It has a source Action targeting the CodeCommit repository, two build Actions using the previously defined projects, and finally a deploy Action that uses AWS CloudFormation\. It takes the template generated by the AWS CDK build Project \(stored in the `LambdaStack.template.json` file, same as the build specified\), and then uses the Lambda code that was passed in its props to reference the output of the build of our Lambda function\. The deployed Lambda function uses the output of that build as its code\. We have to make sure that the Lambda build output is an input to the AWS CloudFormation action though, and that's why we pass it in the `extraInputs` property \(Python: `extra_inputs`\)\.

We also change the name of the stack that will be deployed, from `LambdaStack` to `LambdaDeploymentStack`\. The name change isn't required\. We could have left it the same\.

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
}

export class PipelineStack extends Stack {
  constructor(app: App, id: string, props: PipelineStackProps) {
    super(app, id, props);

    const code = codecommit.Repository.fromRepositoryName(this, 'ImportedRepo',
      'NameOfYourCodeCommitRepository');

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
      'NameOfYourCodeCommitRepository');

    const cdkBuild = new codebuild.PipelineProject(this, 'CdkBuild', {
      buildSpec: codebuild.BuildSpec.fromObject({
        version: '0.2',
        phases: {
          install: {
            commands: 'npm install'
          },
          build: {
            commands: [
              'npm run build',
              'npm run cdk synth -- -o dist'
            ]
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
              'npm install'
            ]
          },
          build: {
            commands: 'npm run build'
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

    def __init__(self, scope: core.Construct, id: str, *,
                 lambda_code: lambda_.CfnParametersCode = None, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        code = codecommit.Repository.from_repository_name(self, "ImportedRepo",
                  "NameOfYourCodeCommitRepository");

        cdk_build = codebuild.PipelineProject(self, "CdkBuild",
                        build_spec=codebuild.BuildSpec.from_object(dict(
                            version="0.2",
                            phases=dict(
                                install=dict(
                                    commands="npm install"),
                                build=dict(commands=[
                                    "npm run build",
                                    "npm run cdk synth -- -o dist"])),
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
                                        "npm install"]),
                                build=dict(
                                    commands="npm run build")),
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

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;

import software.amazon.awscdk.services.codebuild.BuildEnvironment;
import software.amazon.awscdk.services.codebuild.BuildSpec;
import software.amazon.awscdk.services.codebuild.LinuxBuildImage;
import software.amazon.awscdk.services.codebuild.PipelineProject;

import software.amazon.awscdk.services.codecommit.Repository;
import software.amazon.awscdk.services.codecommit.IRepository;

import software.amazon.awscdk.services.codepipeline.Artifact;
import software.amazon.awscdk.services.codepipeline.StageProps;
import software.amazon.awscdk.services.codepipeline.Pipeline;

import software.amazon.awscdk.services.codepipeline.actions.CloudFormationCreateUpdateStackAction;
import software.amazon.awscdk.services.codepipeline.actions.CodeBuildAction;
import software.amazon.awscdk.services.codepipeline.actions.CodeCommitSourceAction;

import software.amazon.awscdk.services.lambda.CfnParametersCode;

public class PipelineStack extends Stack {
    // alternate constructor for calls without props. lambdaCode is required. 
    public PipelineStack(final App scope, final String id, final CfnParametersCode lambdaCode) {
        this(scope, id, null, lambdaCode);
    }
    
    @SuppressWarnings("serial")
    public PipelineStack(final App scope, final String id, final StackProps props, final CfnParametersCode lambdaCode) {
        super(scope, id, props);

        IRepository code = Repository.fromRepositoryName(this, "ImportedRepo", 
                "NameOfYourCodeCommitRepository");

        PipelineProject cdkBuild = PipelineProject.Builder.create(this, "CDKBuild") 
                    .buildSpec(BuildSpec.fromObject(new HashMap<String, Object>() {{
                        put("version", "0.2");
                        put("phases", new HashMap<String, Object>() {{
                            put("install", new HashMap<String, String>() {{
                                put("commands", "npm install");
                            }});
                            put("build", new HashMap<String, Object>() {{
                                put("commands", Arrays.asList("npm run build", 
                                                              "npm run cdk synth -- o dist"));
                            }});
                        }});
                        put("artifacts", new HashMap<String, String>() {{
                            put("base-directory", "dist");
                        }});
                        put("files", Arrays.asList("LambdaStack.template.json"));
                    }}))
                    .environment(BuildEnvironment.builder().buildImage(
                            LinuxBuildImage.STANDARD_2_0).build())
                    .build();

        PipelineProject lambdaBuild = PipelineProject.Builder.create(this, "LambdaBuild") 
                    .buildSpec(BuildSpec.fromObject(new HashMap<String, Object>() {{
                        put("version", "0.2");
                        put("phases", new HashMap<String, Object>() {{ 
                            put("install", new HashMap<String, List<String>>() {{
                                put("commands", Arrays.asList("cd lambda", "npm install"));
                            }});
                            put("build", new HashMap<String, List<String>>() {{
                                put("commands", Arrays.asList("npm run build"));
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
    }

    public class PipelineStack : Stack
    {
        public PipelineStack(App app, string id, PipelineStackProps props = null)
        {
            var code = Repository.FromRepositoryName(this, "ImportedRepo",
                "NameOfYourCodeCommitRepository");

            var cdkBuild = new PipelineProject(this, "CDKBuild", new PipelineProjectProps
            {
                BuildSpec = BuildSpec.FromObject(new Dictionary<string, object>
                {
                    ["version"] = "0.2",
                    ["phases"] = new Dictionary<string, object>
                    {
                        ["install"] = new Dictionary<string, object>
                        {
                            ["commands"] = "npm install"
                        },
                        ["build"] = new Dictionary<string, object>
                        {
                            ["commands"] = new string[] {
                                "npm run build",
                                "npm run cdk synth -- o dist"
                            }
                        }
                    },
                    ["artifacts"] = new Dictionary<string, object>
                    {
                        ["base-directory"] = "dist"
                    },
                    ["files"] = new string[]
                    {
                        "LambdaStack.template.json"
                    }
                }),
                Environment = new BuildEnvironment
                {
                    BuildImage = LinuxBuildImage.STANDARD_2_0
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
                                "npm install"
                            }
                        },
                        ["build"] = new Dictionary<string, string>
                        {
                            ["commands"] = "npm run build"
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
                    new StageProps
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
                    new StageProps
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
                    new StageProps
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

Finally, we have our main AWS CDK entry point file, which contains our app\.

This code is straightforward: it first instantiates the `LambdaStack` class as `LambdaStack`, which is what the AWS CDK build in the pipeline expects\. Then it instantiates the `PipelineStack` class, passing the required Lambda code from the `LambdaStack` object\.

------
#### [ TypeScript ]

File: `bin/pipeline.ts`

```
#!/usr/bin/env node

import { App } from '@aws-cdk/core';
import { LambdaStack } from '../lib/lambda-stack';
import { PipelineStack } from '../lib/pipeline-stack';

const app = new App();

const lambdaStack = new LambdaStack(app, 'LambdaStack');
new PipelineStack(app, 'PipelineDeployingLambdaStack', {
  lambdaCode: lambdaStack.lambdaCode,
});

app.synth();
```

------
#### [ JavaScript ]

File: `bin/pipeline.js`

```
#!/usr/bin/env node

const { App } = require('@aws-cdk/core');
const { LambdaStack } = require('../lib/lambda-stack');
const { PipelineStack } = require('../lib/pipeline-stack');

const app = new App();

const lambdaStack = new LambdaStack(app, 'LambdaStack');
new PipelineStack(app, 'PipelineDeployingLambdaStack', {
  lambdaCode: lambdaStack.lambdaCode
});

app.synth();
```

------
#### [ Python ]

File: `app.py`

```
#!/usr/bin/env python3

from aws_cdk import core

from pipeline.pipeline_stack import PipelineStack
from pipeline.lambda_stack import LambdaStack

app = core.App()

lambda_stack = LambdaStack(app, "LambdaStack")

PipelineStack(app, "PipelineDeployingLambdaStack",
    lambda_code=lambda_stack.lambda_code)

app.synth()
```

------
#### [ Java ]

File: `src/main/java/com/myorg/PipelineApp.java`

```
package com.myorg;

import software.amazon.awscdk.core.App;

public class PipelineApp {
    public static void main(final String argv[]) {
        App app = new App();

        LambdaStack lambdaStack = new LambdaStack(app, "LambdaStack");
        new PipelineStack(app, "PipelineStack", lambdaStack.getLambdaCode());

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
        static void Main(string[] args)
        {
            var app = new App();

            var lambdaStack = new LambdaStack(app, "LambdaStack");
            new PipelineStack(app, "PipelineDeployingLambdaStack", new PipelineStackProps
            {
                LambdaCode = lambdaStack.lambdaCode
            });

            app.Synth();
        }
    }
}
```

------

## Deploying the pipeline<a name="codepipeline_example_create"></a>

The final steps are building the code and deploying the pipeline\.

------
#### [ TypeScript ]

```
npm run build
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
mvn compile
```

**Note**  
Instead of issuing `mvn compile`, you can instead press Control\-B in Eclipse\.

------
#### [ C\# ]

```
dotnet build src
```

**Note**  
Instead of issuing `dotnet build`, you can instead press F6 in Visual Studio\.

------

```
cdk deploy PipelineDeployingLambdaStack
```

The name, **PipelineDeployingLambdaStack**, is the name we used when we instantiated `PipelineStack`\.

**Note**  
Don't deploy *LambdaStack*\. This stack is meant to be deployed by the pipeline\.

After the deployment finishes, you should have a three\-stage pipeline that looks something like the following\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/pipeline.jpg)

Try making a change to your Lambda function code and push it to the repository\. The pipeline should pick up your change, build it, and deploy it automatically, without any human intervention\.

## Cleaning up<a name="codepipeline_example_destroy"></a>

To avoid unexpected AWS charges, destroy your AWS CDK stacks after you're done with this exercise\.

```
cdk destroy '*'
```