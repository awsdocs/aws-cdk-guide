# Deploy AWS CDK applications<a name="deploy"></a>

An AWS Cloud Development Kit \(AWS CDK\) deployment is the process of provisioning your infrastructure on AWS\.

## How AWS CDK deployments work<a name="deploy-how"></a>

The AWS CDK utilizes the AWS CloudFormation service to perform deployments\. Before you deploy, you synthesize your CDK stacks\. This creates a CloudFormation template and deployment artifacts for each CDK stack in your app\. Deployments are initiated from a local development machine or from a *continuous integration and continuous delivery \(CI/CD\)* environment\. During deployment, assets are uploaded to the bootstrapped resources and the CloudFormation template is submitted to CloudFormation to provision your AWS resources\.

For a deployment to be successful, the following is required:
+ The AWS CDK Command Line Interface \(AWS CDK CLI\) must be provided with valid permissions\.
+ The AWS environment must be bootstrapped\.
+ The AWS CDK must know the bootstrapped resources to upload assets into\.

## Prerequisites for CDK deployments<a name="deploy-prerequisites"></a>

Before you can deploy an AWS CDK application, you must complete the following:
+ Configure security credentials for the CDK CLI\.
+ Bootstrap your AWS environment\.
+ Configure an AWS environment for each of your CDK stacks\.
+ Develop your CDK app\.

### Configure security credentials<a name="deploy-prerequisites-creds"></a>

To use the CDK CLI to interact with AWS, you must configure security credentials on your local machine\. For instructions, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\.

### Bootstrap your AWS environment<a name="deploy-prerequisites-bootstrap"></a>

A deployment is always associated with one or more AWS [environments](environments.md)\. Before you can deploy, the environment must first be [bootstrapped](bootstrapping.md)\. Bootstrapping provisions resources in your environment that the CDK uses to perform and manage deployments\. These resources include an Amazon Simple Storage Service \(Amazon S3\) bucket and Amazon Elastic Container Registry \(Amazon ECR\) repository to store and manage [assets](assets.md)\. These resources also include AWS Identity and Access Management \(IAM\) roles that are used to provide permissions during development and deployment\.

We recommend that you use the AWS CDK Command Line Interface \(AWS CDK CLI\) `cdk bootstrap` command to bootstrap your environment\. You can customize bootstrapping or manually create these resources in your environment if necessary\. For instructions, see [Bootstrap your environment for use with the AWS CDK](bootstrapping-env.md)\.

### Configure AWS environments<a name="deploy-prerequisites-env"></a>

Each CDK stack must be associated with an environment to determine where the stack is deployed to\. For instructions, see [Configure environments to use with the AWS CDK](configure-env.md)\.

### Develop your CDK app<a name="deploy-prerequisites-develop"></a>

Within a CDK [project](projects.md), you create and develop your CDK app\. Within your app, you create one or more CDK [stacks](stacks.md)\. Within your stacks, you import and use [constructs](constructs.md) from the AWS Construct Library to define your infrastructure\. Before you can deploy, your CDK app must contain at least one stack\.

## CDK app synthesis<a name="deploy-how-synth"></a>

To perform synthesis, we recommend that you use the CDK CLI `cdk synth` command\. The `cdk deploy` command will also perform synthesis before initiating deployment\. However, by using `cdk synth`, you can validate your CDK app and catch errors before initiating deployment\.

Synthesis behavior is determined by the [stack synthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html#stack-synthesizers) that you configure for your CDK stack\. If you don’t configure a synthesizer, `[DefaultStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizer.html)` will be used\. You can also configure and customize synthesis to meet your needs\. For instructions, see [Configure and customize CDK stack synthesis](configure-synth.md)\.

For your synthesized CloudFormation template to deploy successfully into your environment, it must be compatible with how your environment was bootstrapped\. For example, your CloudFormation template must specify the correct Amazon S3 bucket to deploy assets into\. If you use the default method of bootstrapping your environment, the default stack synthesizer will work\. If you customize CDK behavior, such as customizing bootstrapping or synthesis, CDK deployment behavior may vary\.

### The app lifecycle<a name="deploy-how-synth-app"></a>

When you perform synthesis, your CDK app is run through the following phases, known as the *app lifecycle*:

**Construction \(or Initialization\)**  
Your code instantiates all of the defined constructs and then links them together\. In this stage, all of the constructs \(app, stacks, and their child constructs\) are instantiated and the constructor chain is run\. Most of your app code is run in this stage\.

**Preparation**  
All constructs that have implemented the `prepare` method participate in a final round of modifications, to set up their final state\. The preparation phase happens automatically\. As a user, you don't see any feedback from this phase\. It's rare to need to use the "prepare" hook, and generally not recommended\. Be very careful when mutating the construct tree during this phase, because the order of operations could impact behavior\.  
During this phase, once the construct tree has been built, any [aspects](aspects.md) that you have configured are applied as well\.

**Validation**  
All constructs that have implemented the `validate` method can validate themselves to ensure that they're in a state that will correctly deploy\. You will get notified of any validation failures that happen during this phase\. Generally, we recommend performing validation as soon as possible \(usually as soon as you get some input\) and throwing exceptions as early as possible\. Performing validation early improves reliability as stack traces will be more accurate, and ensures that your code can continue to execute safely\.

**Synthesis**  
 This is the final stage of running your CDK app\. It's triggered by a call to `app.synth()`, and it traverses the construct tree and invokes the `synthesize` method on all constructs\. Constructs that implement `synthesize` can participate in synthesis and produce deployment artifacts to the resulting cloud assembly\. These artifacts include CloudFormation templates, AWS Lambda application bundles, file and Docker image assets, and other deployment artifacts\. In most cases, you won't need to implement the `synthesize` method\.

### Running your app<a name="deploy-how-synth-run"></a>

The CDK CLI needs to know how to run your CDK app\. If you created the project from a template using the `cdk init` command, your app's `cdk.json` file includes an `app` key\. This key specifies the necessary command for the language that the app is written in\. If your language requires compilation, the command line performs this step before running the app automatically\.

------
#### [ TypeScript ]

```
{
  "app": "npx ts-node --prefer-ts-exts bin/my-app.ts"
}
```

------
#### [ JavaScript ]

```
{
  "app": "node bin/my-app.js"
}
```

------
#### [ Python ]

```
{
    "app": "python app.py"
}
```

------
#### [ Java ]

```
{
  "app": "mvn -e -q compile exec:java"
}
```

------
#### [ C\# ]

```
{
  "app": "dotnet run -p src/MyApp/MyApp.csproj"
}
```

------
#### [ Go ]

```
{
  "app": "go mod download && go run my-app.go"
}
```

------

If you didn't create your project using the CDK CLI, or if you want to override the command line given in `cdk.json`, you can provide the `\-\-app` option when running the `cdk` command\.

```
$ cdk --app 'executable' cdk-command ...
```

The *executable* part of the command indicates the command that should be run to execute your CDK application\. Use quotation marks as shown, since such commands contain spaces\. The *cdk\-command* is a subcommand like `synth` or `deploy` that tells the CDK CLI what you want to do with your app\. Follow this with any additional options needed for that subcommand\.

The CDK CLI can also interact directly with an already\-synthesized cloud assembly\. To do that, pass the directory in which the cloud assembly is stored in `--app`\. The following example lists the stacks defined in the cloud assembly stored under `./my-cloud-assembly`\.

```
$ cdk --app ./my-cloud-assembly ls
```

### Cloud assemblies<a name="deploy-how-synth-assemblies"></a>

The call to `app.synth()` is what tells the AWS CDK to synthesize a cloud assembly from an app\. Typically you don't interact directly with cloud assemblies\. They are files that include everything needed to deploy your app to a cloud environment\. For example, it includes an AWS CloudFormation template for each stack in your app\. It also includes a copy of any file assets or Docker images that you reference in your app\.

See the [cloud assembly specification](https://github.com/aws/aws-cdk/blob/master/design/cloud-assembly.md) for details on how cloud assemblies are formatted\.

To interact with the cloud assembly that your AWS CDK app creates, you typically use the AWS CDK CLI\. However, any tool that can read the cloud assembly format can be used to deploy your app\.

## Deploy your application<a name="deploy-how-deploy"></a>

To deploy your application, we recommend that you use the CDK CLI `cdk deploy` command to initiate deployments or to configure automated deployments\.

When you run `cdk deploy`, the CDK CLI initiates `cdk synth` to prepare for deployment\. The following diagram illustrates the app lifecycle in the context of a deployment:

![\[Flowchart of the AWS CDK app lifecycle.\]](http://docs.aws.amazon.com/cdk/v2/guide/images/app-lifecycle_cdk-flowchart.svg)

During deployment, the CDK CLI takes the cloud assembly produced by synthesis and deploys it to an AWS environment\. Assets are uploaded to Amazon S3 and Amazon ECR and the CloudFormation template is submitted to AWS CloudFormation for deployment\.

By the time the AWS CloudFormation deployment phase starts, your CDK app has already finished running and exited\. This has the following implications:
+ The CDK app can't respond to events that happen during deployment, such as a resource being created or the whole deployment finishing\. To run code during the deployment phase, you must inject it into the AWS CloudFormation template as a [custom resource](cfn_layer.md#develop-customize-custom)\. For more information about adding a custom resource to your app, see the [AWS CloudFormation module](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cloudformation-readme.html), or the [custom\-resource](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/custom-resource/) example\. You can also configure the [Triggers](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.triggers-readme.html) module to run code during deployments\.
+ The CDK app might have to work with values that can't be known at the time it runs\. For example, if the AWS CDK app defines an Amazon S3 bucket with an automatically generated name, and you retrieve the `bucket.bucketName` \(Python: `bucket_name`\) attribute, that value is not the name of the deployed bucket\. Instead, you get a `Token` value\. To determine whether a particular value is available, call `cdk.isUnresolved(value)` \(Python: `is_unresolved`\)\. See [Tokens and the AWS CDK](tokens.md) for details\.

### Deployment permissions<a name="deploy-how-deploy-permissions"></a>

Before deployment can be performed, permissions must be established\. The following diagram illustrates the permissions that are used during a default deployment, when using the default bootstrapping process and stack synthesizer:

![\[Flowchart of the default AWS CDK deployment process.\]](http://docs.aws.amazon.com/cdk/v2/guide/images/default-deploy-process_cdk_flowchart.svg)

**Actor initiates deployment**  
Deployments are initiated by an *actor*, using the CDK CLI\. An actor can either be a person, or a service such as AWS CodePipeline\.  
If necessary, the CDK CLI runs `cdk synth` when you run `cdk deploy`\. During synthesis, the AWS identity assumes the `LookupRole` to perform context lookups in the AWS environment\.

**Permissions are established**  
First, the actor’s security credentials are used to authenticate to AWS and obtain the first IAM identity in the process\. For human actors, how security credentials are configured and obtained depends on how you or your organization manages users\. For more information, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\. For service actors, such as CodePipeline, an IAM execution role is assumed and used\.  
Next, the IAM roles created in your AWS environment during bootstrapping are used to establish permissions to perform the actions needed for deployment\. For more information about these roles and what they grant permissions for, see [IAM roles created during bootstrapping](bootstrapping-env.md#bootstrapping-env-roles)\. This process includes the following:  
+ The AWS identity assumes the `DeploymentActionRole` role and passes the `CloudFormationExecutionRole` role to CloudFormation, ensuring that CloudFormation assumes the role when it performs any actions in your AWS environment\. `DeploymentActionRole` grants permission to perform deployments into your environment and `CloudFormationExecutionRole` determines what actions CloudFormation can perform\.
+ The AWS identity assumes the `FilePublishingRole`, which determines the actions that can be performed on the Amazon S3 bucket created during bootstrapping\.
+ The AWS identity assumes the `ImagePublishingRole`, which determines the actions that can be performed on the Amazon ECR repository created during bootstrapping\.
+ If necessary, the AWS identity assumes the `LookupRole` to perform context lookups in the AWS environment\. This action may also be performed during template synthesis\.

**Deployment is performed**  
During deployment, the CDK CLI reads the bootstrap version parameter to confirm the bootstrap version number\. AWS CloudFormation also reads this parameter at deployment time to confirm\. If permissions across the deployment workflow are valid, deployment is performed\. Assets are uploaded to the bootstrapped resources and the CloudFormation template produced at synthesis is deployed using the CloudFormation service as a CloudFormation stack to provision your resources\.