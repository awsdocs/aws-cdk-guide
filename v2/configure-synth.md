# Configure and customize CDK stack synthesis<a name="configure-synth"></a>

Before you can deploy an AWS Cloud Development Kit \(AWS CDK\) stack, it must first be synthesized\. *Stack synthesis* is the process of producing an AWS CloudFormation template and deployment artifacts from a CDK stack\. These templates and artifacts are what gets deployed to provision your resources on AWS\. For more information on how deployments work, see [How AWS CDK deployments work](deploy.md#deploy-how)\.

In the AWS CDK, bootstrapping and synthesis must work together for a deployment to be successful:
+ Bootstrapping is a one\-time process of setting up an AWS environment for AWS CDK deployments\. It configures specific AWS resources in your environment that are used by the CDK for deployments\. These are commonly referred to as *bootstrap resources*\. For instructions on bootstrapping, see [Bootstrap your environment for use with the AWS CDK](bootstrapping-env.md)\.
+ CloudFormation templates produced during synthesis include information on which bootstrap resources to use\. Synthesis behavior is determined by the synthesizer that you configure for each CDK stack\. For a deployment to be successful, the synthesizer must produce CloudFormation templates that reference the correct bootstrap resources to use\.

The CDK comes with a default synthesizer and bootstrapping configuration that are designed to work together\. If you customize one, you must apply relevant customizations to the other\.

## Configure CDK stack synthesis<a name="bootstrapping-synthesizers"></a>

You configure CDK stack synthesis using the `synthesizer` property of your `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` instance\. This property specifies how your CDK stacks will be synthesized\. You provide an instance of a class that implements `[IStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.IStackSynthesizer.html)` or `[IReusableStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.IReusableStackSynthesizer.html)`\. Its methods will be invoked every time an asset is added to the stack or when the stack is synthesized\. The following is a basic example of using this property within your stack:

------
#### [ TypeScript ]

```
new MyStack(this, 'MyStack', {
  // stack properties
  synthesizer: new DefaultStackSynthesizer({
    // synthesizer properties
  }),
});
```

------
#### [ JavaScript ]

```
new MyStack(this, 'MyStack', {
  // stack properties
  synthesizer: new DefaultStackSynthesizer({
    // synthesizer properties
  }),
});
```

------
#### [ Python ]

```
MyStack(self, "MyStack",
    # stack properties
    synthesizer=DefaultStackSynthesizer(
        # synthesizer properties
))
```

------
#### [ Java ]



```
new MyStack(app, "MyStack", StackProps.builder()
  // stack properties
  .synthesizer(DefaultStackSynthesizer.Builder.create()
    // synthesizer properties
    .build())
  .build();
```

------
#### [ C\# ]

```
new MyStack(app, "MyStack", new StackProps
// stack properties
{
    Synthesizer = new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
    {
        // synthesizer properties
    })
});
```

------
#### [ Go ]

```
func main() {
  app := awscdk.NewApp(nil)

  NewMyStack(app, "MyStack", &MyStackProps{
    StackProps: awscdk.StackProps{
      Synthesizer: awscdk.NewDefaultStackSynthesizer(&awscdk.DefaultStackSynthesizerProps{
        // synthesizer properties
      }),
    },
  })

  app.Synth(nil)
}
```

------

You can also configure a synthesizer for all CDK stacks in your CDK app using the `defaultStackSynthesizer` property of your `App` instance:

------
#### [ TypeScript ]



```
import { App, Stack, DefaultStackSynthesizer } from 'aws-cdk-lib';

const app = new App({
  // Configure for all stacks in this app
  defaultStackSynthesizer: new DefaultStackSynthesizer({
    /* ... */
  }),
});
```

------
#### [ JavaScript ]



```
const { App, Stack, DefaultStackSynthesizer } = require('aws-cdk-lib');

const app = new App({
  // Configure for all stacks in this app
  defaultStackSynthesizer: new DefaultStackSynthesizer({
    /* ... */
  }),
});
```

------
#### [ Python ]



```
from aws_cdk import App, Stack, DefaultStackSynthesizer

app = App(
    default_stack_synthesizer=DefaultStackSynthesizer(
        # Configure for all stacks in this app
        # ...
    )
)
```

------
#### [ Java ]



```
import software.amazon.awscdk.App;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.DefaultStackSynthesizer;

public class Main {
    public static void main(final String[] args) {
        App app = new App(AppProps.builder()
            // Configure for all stacks in this app
            .defaultStackSynthesizer(DefaultStackSynthesizer.Builder.create().build())
            .build()
        );
    }
}
```

------
#### [ C\# ]



```
using Amazon.CDK;
using Amazon.CDK.Synthesizers;

namespace MyNamespace
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            var app = new App(new AppProps
            {
                // Configure for all stacks in this app
                DefaultStackSynthesizer = new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
                {
                    // ...
                })
            });
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

func main() {
    defer jsii.Close()
    
    app := awscdk.NewApp(&awscdk.AppProps{
        // Configure for all stacks in this app
        DefaultStackSynthesizer: awscdk.NewDefaultStackSynthesizer(&awscdk.DefaultStackSynthesizerProps{
            // ...
        }),
    })
}
```

------

By default, the AWS CDK uses `[DefaultStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizer.html)`\. If you don’t configure a synthesizer, this synthesizer will be used\.

If you don't modify bootstrapping, such as making changes to the bootstrap stack or template, you don’t have to modify stack synthesis\. You don’t even have to provide a synthesizer\. The CDK will use the default `DefaultStackSynthesizer` class to configure CDK stack synthesis to properly interact with your bootstrap stack\.

## Synthesize a CDK stack<a name="configure-synth-stack"></a>

To synthesize a CDK stack, use the AWS CDK Command Line Interface \(AWS CDK CLI\) `cdk synth` command\. For more information about this command, including options that you can use with this command, see [cdk synthesize](ref-cli-cmd-synth.md)\.

If your CDK app contains a single stack, or to synthesize all stacks, you don't have to provide the CDK stack name as an argument\. By default, the CDK CLI will synthesize your CDK stacks into AWS CloudFormation templates\. A `json` formatted template for each stack is saved to the `cdk.out` directory\. If your app contains a single stack, a `yaml` formatted template is printed to `stdout`\. The following is an example:

```
$ cdk synth
Resources:
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Analytics: v2:deflate64:H4sIAAAAAAAA/unique-identifier
    Metadata:
      aws:cdk:path: CdkAppStack/CDKMetadata/Default
    Condition: CDKMetadataAvailable
    ...
```

If your CDK app contains multiple stacks, you can provide the logical ID of a stack to synthesize a single stack\. The following is an example:

```
$ cdk synth MyStackName
```

If you don’t synthesize a stack and run `cdk deploy`, the CDK CLI will automatically synthesize your stack before deployment\.

## Customize CDK stack synthesis<a name="bootstrapping-custom-synth"></a>

The AWS CDK includes the following built\-in synthesizers that you can use to customize synthesis behavior:
+ `[DefaultStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizer.html)` – If you don't specify a synthesizer, this one is used automatically\. It supports cross\-account deployments and deployments using the [CDK Pipelines](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.pipelines-readme.html) construct\. Its bootstrap contract requires an existing Amazon S3 bucket with a known name, an existing Amazon ECR repository with a known name, and five existing IAM roles with known names\. The default bootstrapping template meets these requirements\. 
+ `[CliCredentialsStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CliCredentialsStackSynthesizer.html)` – This synthesizer's bootstrap contract requires an existing Amazon S3 bucket and existing Amazon ECR repository\. It does not require any IAM roles\. To perform deployments, this synthesizer relies on the permissions of the CDK CLI user and is recommend for organizations that want to restrict IAM deployment credentials\. This synthesizer doesn't support cross\-account deployments or CDK Pipelines\.
+ `[LegacyStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.LegacyStackSynthesizer.html)` – This synthesizer emulates CDK v1 synthesis behavior\. Its bootstrap contract requires an existing Amazon S3 bucket of an arbitrary name and expects the locations of assets to be passed in as CloudFormation stack parameters\. If you use this synthesizer, you must use the CDK CLI to perform deployment\.

If none of these built\-in synthesizers are appropriate for your use case, you can write your own synthesizer as a class that implements `IStackSynthesizer` or look at [synthesizers](https://constructs.dev/search?q=synthesizer&cdk=aws-cdk) from the Construct Hub\.

### Customize the `DefaultStackSynthesizer`<a name="bootstrapping-custom-synth-default"></a>

The `DefaultStackSynthesizer` is the default synthesizer for the AWS CDK\. It is designed to allow cross\-account deployments of CDK applications, as well as deploying CDK apps from a CI/CD system that does not have explicit support for the AWS CDK, but supports regular CloudFormation deployments, such as AWS CodePipeline\. This synthesizer is the best option for most use cases\.

#### `DefaultStackSynthesizer` bootstrap contract<a name="bootstrapping-custom-synth-default-contract"></a>

`DefaultStackSynthesizer` requires the following bootstrap contract\. These are the resources that must be created during bootstrapping:


| Bootstrap resource | Description | Default expected resource name | Purpose | 
| --- | --- | --- | --- | 
|  Amazon S3 bucket  |  Staging bucket  |  cdk\-hnb659fds\-assets\-*ACCOUNT*\-*REGION*  |  Stores file assets\.  | 
|  Amazon ECR repository  |  Staging repository  |  cdk\-hnb659fds\-container\-assets\-*ACCOUNT*\-*REGION*  |  Stores and manages Docker image assets\.  | 
|  IAM role  |  Deploy role  |  cdk\-hnb659fds\-deploy\-role\-*ACCOUNT*\-*REGION*  |  Assumed by the CDK CLI and potentially CodePipeline to assume other roles and start the AWS CloudFormation deployment\. The trust policy of this role controls who can deploy with the AWS CDK in this AWS environment\.  | 
|  IAM role  |  AWS CloudFormation execution role  |  cdk\-hnb659fds\-cfn\-exec\-role\-*ACCOUNT*\-*REGION*  |  This role is used by AWS CloudFormation to perform the deployment\. The policies of this role control what operations the CDK deployment can perform\.  | 
|  IAM role  |  Lookup role  |  cdk\-hnb659fds\-lookup\-role\-*ACCOUNT*\-*REGION*  |  This role is used when the CDK CLI needs to perform environmental context lookups\. The trust policy of this role controls who can look up information in the environment\.  | 
|  IAM role  |  File publishing role  |  cdk\-hnb659fds\-file\-publishing\-role\-*ACCOUNT*\-*REGION*  |  This role is used to upload assets to the Amazon S3 staging bucket\. It is assumed from the deploy role\.  | 
|  IAM role  |  Image publishing role  |  cdk\-hnb659fds\-image\-publishing\-role\-*ACCOUNT*\-*REGION*  |  This role is used to upload Docker images to the Amazon ECR staging repository\. It is assumed from the deploy role\.  | 
|  SSM parameter  |  Bootstrap version parameter  |  /cdk\-bootstrap/hnb659fds/*version*  |  The version of the bootstrap template\. It is used by the bootstrap template and the CDK CLI to validate requirements\.  | 

One way to customize CDK stack synthesis, is by modifying the `[DefaultStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizer.html)`\. You can customize this synthesizer for a single CDK stack using the `synthesizer` property of your `Stack` instance\. You can also modify `DefaultStackSynthesizer` for all stacks in your CDK app using the `defaultStackSynthesizer` property of your `App` instance\.

#### Change the qualifier<a name="bootstrapping-custom-synth-qualifiers"></a>

The *qualifier* is added to the name of resources created during bootstrapping\. By default, this value is `hnb659fds`\. When you modify the qualifier during bootstrapping, you need to customize CDK stack synthesis to use the same qualifier\.

To change the qualifier, configure the `qualifier` property of `DefaultStackSynthesizer` or configure the qualifier as a context key in your CDK project’s `cdk.json` file\.

The following is an example of configuring the `qualifier` property of the `DefaultStackSynthesizer`:

------
#### [ TypeScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    qualifier: 'MYQUALIFIER',
  }),
});
```

------
#### [ JavaScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    qualifier: 'MYQUALIFIER',
  }),
})
```

------
#### [ Python ]

```
MyStack(self, "MyStack",
    synthesizer=DefaultStackSynthesizer(
        qualifier="MYQUALIFIER"
))
```

------
#### [ Java ]

```
new MyStack(app, "MyStack", StackProps.builder()
  .synthesizer(DefaultStackSynthesizer.Builder.create()
    .qualifier("MYQUALIFIER")
    .build())
  .build();
```

------
#### [ C\# ]

```
new MyStack(app, "MyStack", new StackProps
{
    Synthesizer = new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
    {
        Qualifier = "MYQUALIFIER"
    })
});
```

------

The following is an example of configuring the qualifier as a context key in `cdk.json`:

```
{
  "app": "...",
  "context": {
    "@aws-cdk/core:bootstrapQualifier": "MYQUALIFIER"
  }
}
```

#### Change resource names<a name="bootstrapping-custom-synth-names"></a>

All of the other `DefaultStackSynthesizer` properties relate to the names of the resources in the bootstrap template\. You only need to provide any of these properties if you modified the bootstrap template and changed the resource names or naming scheme\.

All properties accept the special placeholders `${Qualifier}`, `${AWS::Partition}`, `${AWS::AccountId}`, and `${AWS::Region}`\. These placeholders are replaced with the values of the `qualifier` parameter and the AWS partition, account ID, and AWS Region values for the stack's environment, respectively\.

The following example shows the most commonly used properties for `DefaultStackSynthesizer` along with their default values, as if you were instantiating the synthesizer\. For a complete list, see [DefaultStackSynthesizerProps](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizerProps.html#properties):

------
#### [ TypeScript ]

```
new DefaultStackSynthesizer({
  // Name of the S3 bucket for file assets
  fileAssetsBucketName: 'cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}',
  bucketPrefix: '',

  // Name of the ECR repository for Docker image assets
  imageAssetsRepositoryName: 'cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}',
  dockerTagPrefix: '',

  // ARN of the role assumed by the CLI and Pipeline to deploy here
  deployRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}',
  deployRoleExternalId: '',

  // ARN of the role used for file asset publishing (assumed from the CLI role)
  fileAssetPublishingRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}',
  fileAssetPublishingExternalId: '',

  // ARN of the role used for Docker asset publishing (assumed from the CLI role)
  imageAssetPublishingRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}',
  imageAssetPublishingExternalId: '',

  // ARN of the role passed to CloudFormation to execute the deployments
  cloudFormationExecutionRole: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}',

  // ARN of the role used to look up context information in an environment
  lookupRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}',
  lookupRoleExternalId: '',

  // Name of the SSM parameter which describes the bootstrap stack version number
  bootstrapStackVersionSsmParameter: '/cdk-bootstrap/${Qualifier}/version',

  // Add a rule to every template which verifies the required bootstrap stack version
  generateBootstrapVersionRule: true,

})
```

------
#### [ JavaScript ]

```
new DefaultStackSynthesizer({
  // Name of the S3 bucket for file assets
  fileAssetsBucketName: 'cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}',
  bucketPrefix: '',

  // Name of the ECR repository for Docker image assets
  imageAssetsRepositoryName: 'cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}',
  dockerTagPrefix: '',

  // ARN of the role assumed by the CLI and Pipeline to deploy here
  deployRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}',
  deployRoleExternalId: '',

  // ARN of the role used for file asset publishing (assumed from the CLI role)
  fileAssetPublishingRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}',
  fileAssetPublishingExternalId: '',

  // ARN of the role used for Docker asset publishing (assumed from the CLI role)
  imageAssetPublishingRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}',
  imageAssetPublishingExternalId: '',

  // ARN of the role passed to CloudFormation to execute the deployments
  cloudFormationExecutionRole: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}',

  // ARN of the role used to look up context information in an environment
  lookupRoleArn: 'arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}',
  lookupRoleExternalId: '',

  // Name of the SSM parameter which describes the bootstrap stack version number
  bootstrapStackVersionSsmParameter: '/cdk-bootstrap/${Qualifier}/version',

  // Add a rule to every template which verifies the required bootstrap stack version
  generateBootstrapVersionRule: true,
})
```

------
#### [ Python ]

```
DefaultStackSynthesizer(
  # Name of the S3 bucket for file assets
  file_assets_bucket_name="cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}",
  bucket_prefix="",

  # Name of the ECR repository for Docker image assets
  image_assets_repository_name="cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}",
  docker_tag_prefix="",

  # ARN of the role assumed by the CLI and Pipeline to deploy here
  deploy_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}",
  deploy_role_external_id="",

  # ARN of the role used for file asset publishing (assumed from the CLI role)
  file_asset_publishing_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}",
  file_asset_publishing_external_id="",

  # ARN of the role used for Docker asset publishing (assumed from the CLI role)
  image_asset_publishing_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}",
  image_asset_publishing_external_id="",

  # ARN of the role passed to CloudFormation to execute the deployments
  cloud_formation_execution_role="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}",

  # ARN of the role used to look up context information in an environment
  lookup_role_arn="arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}",
  lookup_role_external_id="",

  # Name of the SSM parameter which describes the bootstrap stack version number
  bootstrap_stack_version_ssm_parameter="/cdk-bootstrap/${Qualifier}/version",

  # Add a rule to every template which verifies the required bootstrap stack version
  generate_bootstrap_version_rule=True,
)
```

------
#### [ Java ]

```
DefaultStackSynthesizer.Builder.create()
  // Name of the S3 bucket for file assets
  .fileAssetsBucketName("cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}")
  .bucketPrefix('')

  // Name of the ECR repository for Docker image assets
  .imageAssetsRepositoryName("cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}")
  .dockerTagPrefix('')

  // ARN of the role assumed by the CLI and Pipeline to deploy here
  .deployRoleArn("arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}")
  .deployRoleExternalId("")

  // ARN of the role used for file asset publishing (assumed from the CLI role)
  .fileAssetPublishingRoleArn("arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}")
  .fileAssetPublishingExternalId("")

  // ARN of the role used for Docker asset publishing (assumed from the CLI role)
  .imageAssetPublishingRoleArn("arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}")
  .imageAssetPublishingExternalId("")

  // ARN of the role passed to CloudFormation to execute the deployments
  .cloudFormationExecutionRole("arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}")

  .lookupRoleArn("arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}")
  .lookupRoleExternalId("")

  // Name of the SSM parameter which describes the bootstrap stack version number
  .bootstrapStackVersionSsmParameter("/cdk-bootstrap/${Qualifier}/version")

  // Add a rule to every template which verifies the required bootstrap stack version
  .generateBootstrapVersionRule(true)
.build()
```

------
#### [ C\# ]

```
new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
{
    // Name of the S3 bucket for file assets
    FileAssetsBucketName = "cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}",
    BucketPrefix = "",

    // Name of the ECR repository for Docker image assets
    ImageAssetsRepositoryName = "cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}",
    DockerTagPrefix = "",

    // ARN of the role assumed by the CLI and Pipeline to deploy here
    DeployRoleArn = "arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-deploy-role-${AWS::AccountId}-${AWS::Region}",
    DeployRoleExternalId = "",

    // ARN of the role used for file asset publishing (assumed from the CLI role)
    FileAssetPublishingRoleArn = "arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-file-publishing-role-${AWS::AccountId}-${AWS::Region}",
    FileAssetPublishingExternalId = "",

    // ARN of the role used for Docker asset publishing (assumed from the CLI role)
    ImageAssetPublishingRoleArn = "arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-image-publishing-role-${AWS::AccountId}-${AWS::Region}",
    ImageAssetPublishingExternalId = "",

    // ARN of the role passed to CloudFormation to execute the deployments
    CloudFormationExecutionRole = "arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-cfn-exec-role-${AWS::AccountId}-${AWS::Region}",

    LookupRoleArn = "arn:${AWS::Partition}:iam::${AWS::AccountId}:role/cdk-${Qualifier}-lookup-role-${AWS::AccountId}-${AWS::Region}",
    LookupRoleExternalId = "",

    // Name of the SSM parameter which describes the bootstrap stack version number
    BootstrapStackVersionSsmParameter = "/cdk-bootstrap/${Qualifier}/version",

    // Add a rule to every template which verifies the required bootstrap stack version
    GenerateBootstrapVersionRule = true,
})
```

------

### Use `CliCredentialsStackSynthesizer`<a name="bootstrapping-custom-synth-cli"></a>

To modify the security credentials used to provide permissions during CDK deployments, you can customize synthesis by using `[CliCredentialsStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CliCredentialsStackSynthesizer.html)`\. This synthesizer works with the default AWS resources that are created during bootstrapping to store assets, such as the Amazon S3 bucket and Amazon ECR repository\. Instead of using the default IAM roles created by the CDK during bootstrapping, it uses the security credentials of the actor initiating deployment\. Therefore, the security credentials of the actor must have valid permissions to perform all deployment actions\. The following diagram illustrates the deployment process when using this synthesizer:

![\[Flowchart of the default AWS CDK deployment process.\]](http://docs.aws.amazon.com/cdk/v2/guide/images/CliCredentialsStackSynthesizer-deploy-process_cdk_flowchart.svg)

When using `CliCredentialsStackSynthesizer`:
+ By default, CloudFormation performs API calls in your account using the permissions of the actor\. Therefore, the current identity must have permission to make necessary changes to the AWS resources in the CloudFormation stack, along with the permissions to perform necessary CloudFormation operations, such as `CreateStack` or `UpdateStack`\. Deployment capabilities will be limited to the permissions of the actor\.
+ Asset publishing and CloudFormation deployments will be done using the current IAM identity\. This identity must have sufficient permissions to both read from and write to the asset bucket and repository\.
+ Lookups are performed using the current IAM identity, and lookups are subject to its policies\.

When using this synthesizer, you can use a separate CloudFormation execution role by specifying it using the `\-\-role\-arn` option with any CDK CLI command\.

#### `CliCredentialsStackSynthesizer` bootstrap contract<a name="bootstrapping-custom-synth-cli-contract"></a>

`CliCredentialsStackSynthesizer` requires the following bootstrap contract\. These are the resources that must be created during bootstrapping:


| Bootstrap resource | Description | Default expected resource name | Purpose | 
| --- | --- | --- | --- | 
|  Amazon S3 bucket  |  Staging bucket  |  cdk\-hnb659fds\-assets\-*ACCOUNT*\-*REGION*  |  Stores file assets\.  | 
|  Amazon ECR repository  |  Staging repository  |  cdk\-hnb659fds\-container\-assets\-*ACCOUNT*\-*REGION*  |  Stores and manages Docker image assets\.  | 

The string `hnb659fds` in the resource name is called the *qualifier*\. Its default value has no special significance\. You can have multiple copies of the bootstrap resources in a single environment as long as they have a different qualifier\. Having multiple copies can be useful for keeping assets of different applications in the same environment separated\.

You can deploy the default bootstrap template to satisfy `CliCredentialsStackSynthesizer`'s bootstrap contract\. The default bootstrap template will create IAM roles, but this synthesizer will not use them\. You can also customize the bootstrap template to remove the IAM roles\.

#### Modify `CliCredentialsStackSynthesizer`<a name="bootstrapping-custom-synth-cli-modify"></a>

If you change the qualifier or any of the default bootstrap resource names during bootstrapping, you have to modify the synthesizer to use the same names\. You can modify the synthesizer for a single stack or for all stacks in your app\. The following is an example:

------
#### [ TypeScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new CliCredentialsStackSynthesizer({
    qualifier: 'MYQUALIFIER',
  }),
});
```

------
#### [ JavaScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new CliCredentialsStackSynthesizer({
    qualifier: 'MYQUALIFIER',
  }),
})
```

------
#### [ Python ]

```
MyStack(self, "MyStack",
    synthesizer=CliCredentialsStackSynthesizer(
        qualifier="MYQUALIFIER"
))
```

------
#### [ Java ]

```
new MyStack(app, "MyStack", StackProps.builder()
  .synthesizer(CliCredentialsStackSynthesizer.Builder.create()
    .qualifier("MYQUALIFIER")
    .build())
  .build();
```

------
#### [ C\# ]

```
new MyStack(app, "MyStack", new StackProps
{
    Synthesizer = new CliCredentialsStackSynthesizer(new CliCredentialsStackSynthesizerProps
    {
        Qualifier = "MYQUALIFIER"
    })
});
```

------

The following example shows the most commonly used properties for `CliCredentialsStackSynthesizer` along with their default values\. For a complete list, see [CliCredentialsStackSynthesizerProps](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CliCredentialsStackSynthesizerProps.html):

------
#### [ TypeScript ]

```
new CliCredentialsStackSynthesizer({
  // Value for '${Qualifier}' in the resource names
  qualifier: 'hnb659fds',
  
  // Name of the S3 bucket for file assets
  fileAssetsBucketName: 'cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}',
  bucketPrefix: '',

  // Name of the ECR repository for Docker image assets
  imageAssetsRepositoryName: 'cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}',
  dockerTagPrefix: '',
})
```

------
#### [ JavaScript ]

```
new CliCredentialsStackSynthesizer({
  // Value for '${Qualifier}' in the resource names
  qualifier: 'hnb659fds',
  
  // Name of the S3 bucket for file assets
  fileAssetsBucketName: 'cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}',
  bucketPrefix: '',

  // Name of the ECR repository for Docker image assets
  imageAssetsRepositoryName: 'cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}',
  dockerTagPrefix: '',
})
```

------
#### [ Python ]

```
CliCredentialsStackSynthesizer(
  # Value for '${Qualifier}' in the resource names
  qualifier="hnb659fds",
  
  # Name of the S3 bucket for file assets
  file_assets_bucket_name="cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}",
  bucket_prefix="",

  # Name of the ECR repository for Docker image assets
  image_assets_repository_name="cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}",
  docker_tag_prefix="",
)
```

------
#### [ Java ]

```
CliCredentialsStackSynthesizer.Builder.create()
  // Value for '${Qualifier}' in the resource names
  .qualifier("hnb659fds")
  
  // Name of the S3 bucket for file assets
  .fileAssetsBucketName("cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}")
  .bucketPrefix('')

  // Name of the ECR repository for Docker image assets
  .imageAssetsRepositoryName("cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}")
  .dockerTagPrefix('')
.build()
```

------
#### [ C\# ]

```
new CliCredentialsStackSynthesizer(new CliCredentialsStackSynthesizerProps
{

    // Value for '${Qualifier}' in the resource names
    Qualifier = "hnb659fds",
  
    // Name of the S3 bucket for file assets
    FileAssetsBucketName = "cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}",
    BucketPrefix = "",

    // Name of the ECR repository for Docker image assets
    ImageAssetsRepositoryName = "cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}",
    DockerTagPrefix = "",
})
```

------

### Use `LegacyStackSynthesizer`<a name="bootstrapping-custom-synth-legacy"></a>

The `LegacyStackSynthesizer` emulates the behavior of CDK v1 deployments\. The security credentials of the actor performing deployment will be used to establish permissions\. File assets will be uploaded to a bucket that must be created using a AWS CloudFormation stack named `CDKToolkit`\. The CDK CLI will create an unmanaged Amazon ECR repository named `aws-cdk/assets` to store Docker image assets\. You will be responsible to clean up and manage this repository\. Stacks synthesized using the `LegacyStackSynthesizer` can only be deployed using the CDK CLI\.

You can use the `LegacyStackSynthesizer` if you are migrating from CDK v1 to CDK v2, and are unable to re\-bootstrap your environments\. For new projects, we recommend that you don't use `LegacyStackSynthesizer`\.

#### `LegacyStackSynthesizer` bootstrap contract<a name="bootstrapping-custom-synth-legacy-contract"></a>

`LegacyStackSynthesizer` requires the following bootstrap contract\. These are the resources that must be created during bootstrapping:


| Bootstrap resource | Description | Default expected resource name | Purpose | 
| --- | --- | --- | --- | 
|  Amazon S3 bucket  |  Staging bucket  |  cdk\-hnb659fds\-assets\-*ACCOUNT*\-*REGION*  |  Stores file assets\.  | 
|  CloudFormation output  |  Bucket name output  |  Stack – `CDKToolkit` Output name – `BucketName`  |  A CloudFormation output describing the name of the staging bucket  | 

The `LegacyStackSynthesizer` does not assume the existence of an Amazon S3 bucket with a fixed name\. Instead, the synthesized CloudFormation template will contain three CloudFormation parameters for each file asset\. These parameters will store the Amazon S3 bucket name, Amazon S3 object key, and artifact hash for each file asset\.

Docker image assets will be published to an Amazon ECR repository named `aws-cdk/assets`\. This name can be changed per asset\. The repositories will be created if they do not exist\.

A CloudFormation stack must exist with the default name `CDKToolkit`\. This stack must have a CloudFormation export named `BucketName` that refers to the staging bucket\.

The default bootstrap template satisfies the `LegacyStackSynthesizer` bootstrap contract\. However, only the Amazon S3 bucket will be used\. You can customize the bootstrap template to remove the Amazon ECR, IAM, and SSM bootstrap resources\.

#### `LegacyStackSynthesizer` deployment process<a name="bootstrapping-custom-synth-legacy-deploy"></a>

When you use this synthesizer, the following process is performed during deployment:
+ The CDK CLI looks for a CloudFormation stack named `CDKToolkit` in your environment\. From this stack, the CDK CLI reads the CloudFormation output named `BucketName`\. You can use the `--toolkit-stack-name` option with `cdk deploy` to specify a different stack name\.
+ The security credentials of the actor initiating deployment will be used to establish permissions for deployment\. Therefore, the actor must have sufficient permissions to perform all deployment actions\. This includes reading and writing to the Amazon S3 staging bucket, creating and writing to the Amazon ECR repository, starting and monitoring AWS CloudFormation deployments, and performing any API calls necessary for deployment\.
+ If necessary, and if permissions are valid, file assets will be published to the Amazon S3 staging bucket\.
+ If necessary, and if permissions are valid, Docker image assets are published to the repository named by the `repositoryName` property of the asset\. The default value is `'aws-cdk/assets'` if you don't provide a repository name\.
+ If permissions are valid, the AWS CloudFormation deployment is performed\. The locations of the Amazon S3 staging bucket and keys are passed as CloudFormation parameters\.