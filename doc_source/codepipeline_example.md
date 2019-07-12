# Creating a Code Pipeline Using the AWS CDK<a name="codepipeline_example"></a>

This example creates a code pipeline using the AWS CDK\.

The AWS CDK enables you to easily create applications running in the AWS Cloud\. But creating the application is just the start of the journey\. You also want to make changes to it, test those changes, and finally deploy them to your stack\. The AWS CDK enables this workflow by using the **Code\*** suite of AWS tools: AWS CodeCommit, AWS CodeBuild, AWS CodeDeploy, and AWS CodePipeline\. Together, they allow you to build what’s called a [deployment pipeline](https://aws.amazon.com/getting-started/tutorials/continuous-deployment-pipeline/) for your application\.

The following example shows how to deploy an AWS Lambda function in a pipeline\. In this example, your AWS CDK code and your Lambda code are in the same repository\. The Lambda code is in the `Lambda` directory, while your CDK code is in the `bin` and `lib` directories that the `cdk init` command sets up for your CDK code\.

## Lambda Stack<a name="codepipeline_example_lambda"></a>

The first step is to create the file `lambda-stack.ts` in which we define the class `LambdaStack`\. This class defines the AWS CloudFormation stack for the Lambda function\. This is the stack that is deployed in our pipeline\.

This class includes the `lambdaCode` property, which is an instance of the `CfnParametersCode` class\. This property represents the code that is supplied later by the pipeline\. Because the pipeline needs access to the object, we expose it as a public, read\-only property on our class\.

The example also uses the CodeDeploy support for blue\-green deployments to Lambda, and the deployment increases the traffic to the new version in 10\-percent increments every minute\. As blue\-green deployment can only operate on aliases, not on the function directly, we create an alias for our function, named `Prod`\.

The alias uses a Lambda version, which is named after the date when the code executed\. This ensures that every invocation of the AWS CDK code publishes a new version of the function\.

If the Lambda function needs any other resources when executing, such as an Amazon S3 bucket, Amazon DynamoDB table, or Amazon API Gateway, declare those resources here\.

```
// file: lib/lambda-stack.ts
    
    import codedeploy = require('@aws-cdk/aws-codedeploy');
    import lambda = require('@aws-cdk/aws-lambda');
    import { App, Stack, StackProps } from '@aws-cdk/core';
    
    export class LambdaStack extends Stack {
      public readonly lambdaCode: lambda.CfnParametersCode;
    
      constructor(app: App, id: string, props?: StackProps) {
        super(app, id, props);
    
        this.lambdaCode = lambda.Code.cfnParameters();
    
        const func = new lambda.Function(this, 'Lambda', {
          code: this.lambdaCode,
          handler: 'index.handler',
          runtime: lambda.Runtime.NODEJS_8_10,
        });
        
        const version = func.addVersion(new Date().toISOString());
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

## Pipeline Stack<a name="codepipeline_example_stack"></a>

The second class, `PipelineStack`, is the stack that contains our pipeline\.

First it needs a reference to the Lambda code it’s deploying\. For that, we define a new props interface for it, `PipelineStackProps`\. This extends the standard `StackProps`, and use it in its constructor signature\. This is how clients of this class pass the Lambda code that the class needs\.

Then comes the Git repository used to store the source code\. In the example, it’s hosted by CodeCommit\. The `Repository.fromRepositoryName` method is a standard AWS CDK idiom for referencing a resource, such as a CodeCommit repository, that lives outside the AWS CDK code\. Replace *NameOfYourCodeCommitRepository* with the name of your repository\.

The example has two CodeBuild projects\. The first project obtains the AWS CloudFormation template from the AWS CDK code\. To do that, it calls the standard install and build targets for Node\.js, and then calls the cdk synth command\. This produces AWS CloudFormation templates in the target directory `dist`\. Finally, it uses the `dist/LambdaStack.template.json` file as its output\.

The second project does a similar thing, except for the Lambda code\. Because of that, it starts by changing the current directory to `lambda`, which is where we said the Lambda code lives in the repository\. It then invokes the same install and build Node\.js targets as before\. The output is the contents of the node\_modules directory, plus the `index.js` file\. Because `index.handler` is the entry point to the Lambda code, `index.js` must exist, and must export a `handler` function\. This function is called by the Lambda runtime to handle requests\. If your Lambda code uses more files than just `index.js`, add them here\.

Finally, we create our pipeline\. It has a source Action targeting the CodeCommit repository, two build Actions using the previously defined projects, and finally a deploy Action that uses AWS CloudFormation\. It takes the template generated by the AWS CDK build Project \(stored in the `LambdaStack.template.json` file, same as the build specified\), and then uses the Lambda code that was passed in its props to reference the output of the build of our Lambda function\. The deployed Lambda function uses the output of that build as its code\. We have to make sure that the Lambda build output is an input to the AWS CloudFormation action though, and that’s why we pass it in the `extraInputs` property\.

We also change the name of the stack that will be deployed, from `LambdaStack` to `LambdaDeploymentStack`\. The name change isn't required\. We could have left it the same\.

```
// lib/pipeline-stack.ts

import codebuild = require('@aws-cdk/aws-codebuild');
import codecommit = require('@aws-cdk/aws-codecommit');
import codepipeline = require('@aws-cdk/aws-codepipeline');
import codepipeline_actions = require('@aws-cdk/aws-codepipeline-actions');
import lambda = require('@aws-cdk/aws-lambda');
import s3 = require('@aws-cdk/aws-s3');
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
        buildImage: codebuild.LinuxBuildImage.UBUNTU_14_04_NODEJS_8_11_0,
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
        buildImage: codebuild.LinuxBuildImage.UBUNTU_14_04_NODEJS_8_11_0,
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

## Main Program<a name="codepipeline_example_main"></a>

Finally, we have our main AWS CDK entry point file, `pipeline.ts`, in the `bin` directory\. It’s simple: it first instantiates the `LambdaStack` class as `LambdaStack`, which is what the AWS CDK build in the pipeline expects\. Then it instantiates the `PipelineStack` class, passing the required Lambda code from the `LambdaStack` object\.

```
#!/usr/bin/env node

// bin/pipeline.ts

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

## Creating the Pipeline<a name="codepipeline_example_create"></a>

The final steps are building the code and deploying the pipeline\. Use the following command to build the TypeScript code\.

```
npm run build
```

Use the following command to deploy the pipeline stack\.

```
npm run cdk deploy PipelineDeployingLambdaStack
```

The name, **PipelineDeployingLambdaStack**, is the name we used when we instantiated `PipelineStack` in `pipeline.ts`\.

After the deployment finishes, you should have a three\-stage pipeline that looks something like the following\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/pipeline.jpg)

Try making a change, such as to your `LambdaStack` AWS CDK code or to your Lambda function code, and push it to the repository\. The pipeline should pick up your change, build it, and deploy it automatically, without any human intervention\.