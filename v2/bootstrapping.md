# Bootstrapping<a name="bootstrapping"></a>

*Bootstrapping* is the process of preparing an [environment](environments.md) for deployment\. Bootstrapping is a one\-time action that you must perform for every environment that you deploy resources into\.

**Topics**
+ [Bootstrapping environments](#bootstrapping-env)
+ [How to bootstrap](#bootstrapping-howto)
+ [Customizing bootstrapping](#bootstrapping-customizing)
+ [Bootstrapping template differences](#bootstrapping-template)
+ [Stack synthesizers](#bootstrapping-synthesizers)
+ [Customizing synthesis](#bootstrapping-custom-synth)
+ [The bootstrapping template contract](#bootstrapping-contract)
+ [Security Hub Findings](#bootstrapping-securityhub)

## Bootstrapping environments<a name="bootstrapping-env"></a>

**Important**  
You may incur AWS charges for data stored in the bootstrapped resources\.

Bootstrapping provisions resources in your environment such as an Amazon Simple Storage Service \(Amazon S3\) bucket for storing files and AWS Identity and Access Management \(IAM\) roles that grant permissions needed to perform deployments\. These resources get provisioned in an AWS CloudFormation stack, called the *bootstrap stack*\. It is usually named `CDKToolkit`\. Like any AWS CloudFormation stack, it will appear in the AWS CloudFormation console of your environment once it has been deployed\.

**Note**  
CDK v2 uses a modern bootstrap template\. The legacy template from CDK v1 is not supported in v2\.

Environments are independent\. If you want to deploy to multiple environments, each environment must be bootstrapped separately\.

If you attempt to deploy a CDK app into an environment that hasn't been bootstrapped, you will receive an error message reminding you to bootstrap the environment\.

### Bootstrapping with CDK Pipelines<a name="bootstrapping-env-pipelines"></a>

If you are using CDK Pipelines to deploy into another account's environment, and you receive a message like the following:

```
Policy contains a statement with one or more invalid principals
```

This error message means that the appropriate IAM roles do not exist in the other environment\. The most likely cause is that the environment has not been bootstrapped\. Bootstrap the environment and try again\.

**Note**  
If the environment is bootstrapped, do not delete and recreate the environment's bootstrap stack\. Deleting the bootstrap stack will delete the AWS resources that were originally provisioned in the environment to support CDK deployments\. This will cause the pipeline to stop working\. Instead, try to update the bootstrap stack to a new version by running the CDK CLI `cdk bootstrap` command again\.

## How to bootstrap<a name="bootstrapping-howto"></a>

When you bootstrap an environment, an AWS CloudFormation template is deployed to the specific environment\. This template provisions resources in your account to prepare your environment for deployment\.

The bootstrapping template accepts parameters that customize some aspects of the bootstrapped resources\. For more information, see [Customizing bootstrapping](#bootstrapping-customizing)\.

You can bootstrap in any of the following ways:
+ Use the AWS CDK CLI cdk bootstrap command\. This is the simplest method and works well if you have only a few environments to bootstrap\.
+ Deploy the template provided by the AWS CDK CLI using another AWS CloudFormation deployment tool\. This lets you use AWS CloudFormation StackSets or AWS Control Tower and also the AWS CloudFormation console or the AWS CLI\. You can make small modifications to the template before deployment\. This approach is more flexible and is suitable for large\-scale deployments\.

It's not an error to bootstrap an environment more than once\. If an environment you bootstrap has already been bootstrapped, its bootstrap stack will be upgraded if necessary\. Otherwise, nothing will happen\.

### Bootstrapping with the AWS CDK CLI<a name="bootstrapping-howto-cli"></a>

Use the `cdk bootstrap` command to bootstrap one or more AWS environments\.

The following example bootstraps two environments:

```
$ cdk bootstrap aws://ACCOUNT-NUMBER-1/REGION-1 aws://ACCOUNT-NUMBER-2/REGION-2 ...
```

The following examples show multiple ways of bootstrapping environments\. As shown in the second example, the `aws://` prefix is optional when specifying an environment\.

```
$ cdk bootstrap aws://123456789012/us-east-1
$ cdk bootstrap 123456789012/us-east-1 123456789012/us-west-1
```

When you run cdk bootstrap, the CDK CLI always synthesizes the CDK app in the current directory\. If you do not specify at least one environment, the CDK CLI will bootstrap all environments referenced in the app\.

For environment\-agnostic stacks, the CDK CLI will attempt to determine an environment from default sources\. This could be an environment specified using the \-\-profile option, from environment variables, or default AWS CLI sources\. If found, the environment is then bootstrapped\.

For example, the following command synthesizes the current AWS CDK app using the `prod` AWS profile, then bootstraps its environments\.

```
$ cdk bootstrap --profile prod
```

### Bootstrapping from the AWS CloudFormation template<a name="bootstrapping-howto-cfn"></a>

You can bootstrap an environment by obtaining and deploying the bootstrap AWS CloudFormation template\.

To get a copy of this template in the file `bootstrap-template.yaml`, run the following command:

------
#### [ macOS/Linux ]

```
$ cdk bootstrap --show-template > bootstrap-template.yaml
```

------
#### [ Windows ]

On Windows, PowerShell must be used to preserve the encoding of the template\.

```
powershell "cdk bootstrap --show-template | Out-File -encoding utf8 bootstrap-template.yaml"
```

------

The template is also available in the [AWS CDK GitHub repository](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk/lib/api/bootstrap/bootstrap-template.yaml)\.

Deploy this template using the CDK CLI or your preferred deployment mechanism for AWS CloudFormation templates\. To deploy using the CDK CLI, run cdk bootstrap \-\-template *TEMPLATE\_FILENAME*\. You can also deploy it using the AWS CLI by running the command below, or [deploy to one or more accounts at once using AWS CloudFormation Stack Sets](https://aws.amazon.com/blogs/mt/bootstrapping-multiple-aws-accounts-for-aws-cdk-using-cloudformation-stacksets/)\. 

------
#### [ macOS/Linux ]

```
aws cloudformation create-stack \
  --stack-name CDKToolkit \
  --template-body file://path/to/bootstrap-template.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-west-1
```

------
#### [ Windows ]

```
aws cloudformation create-stack ^
  --stack-name CDKToolkit ^
  --template-body file://path/to/bootstrap-template.yaml ^
  --capabilities CAPABILITY_NAMED_IAM ^
  --region us-west-1
```

------

## Customizing bootstrapping<a name="bootstrapping-customizing"></a>

There are two ways to customize the bootstrapping of resources in your environment:
+ Use command line parameters with the `cdk bootstrap` command\. This lets you modify a few aspects of the template\.
+ Modify the default bootstrap template and deploy it yourself\. This gives you more complete control over the bootstrap resources\.

The following command line options, when used with CDK CLI cdk bootstrap, provide commonly used adjustments to the bootstrapping template:
+  \-\-bootstrap\-bucket\-name overrides the name of the Amazon S3 bucket\. May require changes to your CDK app \(see [Stack synthesizers](#bootstrapping-synthesizers)\)\.
+ \-\-bootstrap\-kms\-key\-id overrides the AWS KMS key used to encrypt the S3 bucket\.
+ \-\-cloudformation\-execution\-policies specifies the ARNs of managed policies that should be attached to the deployment role assumed by AWS CloudFormation during deployment of your stacks\. By default, stacks are deployed with full administrator permissions using the `AdministratorAccess` policy\.

  The policy ARNs must be passed as a single string argument, with the individual ARNs separated by commas\. For example:

  ```
  --cloudformation-execution-policies "arn:aws:iam::aws:policy/AWSLambda_FullAccess,arn:aws:iam::aws:policy/AWSCodeDeployFullAccess".
  ```
**Important**  
To avoid deployment failures, be sure the policies that you specify are sufficient for any deployments you will perform in the environment being bootstrapped\.
+ \-\-qualifier is a string that is added to the names of all resources in the bootstrap stack\. A qualifier lets you avoid resource name clashes when you provision multiple bootstrap stacks in the same environment\. The default is `hnb659fds` \(this value has no significance\)\.

  Changing the qualifier also requires that your CDK app pass the changed value to the stack synthesizer\. For more information, see [Stack synthesizers](#bootstrapping-synthesizers)\. 
+ \-\-tags adds one or more AWS CloudFormation tags to the bootstrap stack\.
+ \-\-trust lists the AWS accounts that may deploy into the environment being bootstrapped\.

  Use this flag when bootstrapping an environment that a CDK Pipeline in another environment will deploy into\. The account doing the bootstrapping is always trusted\.
+ \-\-trust\-for\-lookup lists the AWS accounts that may look up context information from the environment being bootstrapped\.

  Use this flag to give accounts permission to synthesize stacks that will be deployed into the environment, without actually giving them permission to deploy those stacks directly\.
+ \-\-termination\-protection prevents the bootstrap stack from being deleted\. For more information, see [Protecting a stack from being deleted](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-protect-stacks.html) in the *AWS CloudFormation User Guide*\.

**Important**  
The modern bootstrap template effectively grants the permissions implied by the `--cloudformation-execution-policies` to any AWS account in the `--trust` list\. By default, this extends permissions to read and write to any resource in the bootstrapped account\. Make sure to [configure the bootstrapping stack](#bootstrapping-customizing) with policies and trusted accounts that you are comfortable with\.

### Customizing the template<a name="bootstrapping-customizing-extended"></a>

When you need more customization than the CDK CLI can provide, you can modify the bootstrap template to suit your needs\. First, you obtain the template using the \-\-show\-template option\. The following is an example:

```
$ cdk bootstrap --show-template
```

Any modifications you make must adhere to the [bootstrapping template contract](#bootstrapping-contract)\. To ensure that your customizations are not accidentally overwritten later by someone running cdk bootstrap using the default template, change the default value of the `BootstrapVariant` template parameter\. The CDK CLI will only allow overwriting the bootstrap stack with templates that have the same `BootstrapVariant` and a equal or higher version than the template that is currently deployed\. 

You can then deploy your modified template as described in [Bootstrapping from the AWS CloudFormation template](#bootstrapping-howto-cfn), or using cdk bootstrap \-\-template\.

```
$ cdk bootstrap --template bootstrap-template.yaml
```

## Bootstrapping template differences<a name="bootstrapping-template"></a>

As previously mentioned, AWS CDK v1 supported two bootstrapping templates, legacy and modern\. CDK v2 supports only the modern template\. For reference, here are the high\-level differences between these two templates\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/cdk/v2/guide/bootstrapping.html)

\* *We will add additional resources to the bootstrap template as needed\.*

An environment that was bootstrapped using the legacy template must be upgraded to use the modern template for CDK v2 by re\-bootstrapping\. Re\-deploy all AWS CDK applications in the environment at least once before deleting the legacy bucket\.

## Stack synthesizers<a name="bootstrapping-synthesizers"></a>

Your AWS CDK app needs to know about the bootstrapping resources available to it in order to successfully synthesize a stack that can be deployed\. The *stack synthesizer* is an AWS CDK class that controls how the stack's template is synthesized\. This includes how it uses bootstrapping resources \(for example, how it refers to assets stored in the bootstrap bucket\)\.

The AWS CDK's built\-in stack synthesizers is called `DefaultStackSynthesizer`\. It includes capabilities for cross\-account deployments and [CDK Pipelines](cdk_pipeline.md) deployments\.

You can pass a stack synthesizer to a stack when you instantiate it using the `synthesizer` property\.

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

If you don't provide the `synthesizer` property, `DefaultStackSynthesizer` is used\.

## Customizing synthesis<a name="bootstrapping-custom-synth"></a>

Depending on the changes you made to the bootstrap template, you may also need to customize synthesis\. The `DefaultStackSynthesizer` can be customized using the properties described as follows\.

If none of these properties provide the customizations you require, you can write your synthesizer as a class that implements `IStackSynthesizer` \(perhaps deriving from `DefaultStackSynthesizer`\)\.

### Changing the qualifier<a name="bootstrapping-custom-synth-qualifiers"></a>

The *qualifier* is added to the name of bootstrap resources to distinguish the resources in separate bootstrap stacks\. To deploy two different versions of the bootstrap stack in the same environment \(AWS account and Region\), the stacks must have different qualifiers\.

This feature is intended for name isolation between automated tests of the CDK itself\. Unless you can very precisely scope down the IAM permissions given to the AWS CloudFormation execution role, there are no permission isolation benefits to having two different bootstrap stacks in a single account\. Therefore, there's usually no need to change this value\.

To change the qualifier, configure the `DefaultStackSynthesizer` either by instantiating the synthesizer with the property:

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

Or by configuring the qualifier as a context key in `cdk.json`\.

```
{
  "app": "...",
  "context": {
    "@aws-cdk/core:bootstrapQualifier": "MYQUALIFIER"
  }
}
```

### Changing the resource names<a name="bootstrapping-custom-synth-names"></a>

All the other `DefaultStackSynthesizer` properties relate to the names of the resources in the bootstrapping template\. You only need to provide any of these properties if you modified the bootstrap template and changed the resource names or naming scheme\.

All properties accept the special placeholders `${Qualifier}`, `${AWS::Partition}`, `${AWS::AccountId}`, and `${AWS::Region}`\. These placeholders are replaced with the values of the `qualifier` parameter and the AWS partition, account ID, and Region values for the stack's environment, respectively\.

The following example shows the most commonly used properties for `DefaultStackSynthesizer` along with their default values, as if you were instantiating the synthesizer\. For a complete list, see [DefaultStackSynthesizerProps](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.DefaultStackSynthesizerProps.html#properties)\.

------
#### [ TypeScript ]

```
new DefaultStackSynthesizer({
  // Name of the S3 bucket for file assets
  fileAssetsBucketName: 'cdk-${Qualifier}-assets-${AWS::AccountId}-${AWS::Region}',
  bucketPrefix: '',

  // Name of the ECR repository for Docker image assets
  imageAssetsRepositoryName: 'cdk-${Qualifier}-container-assets-${AWS::AccountId}-${AWS::Region}',

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

## The bootstrapping template contract<a name="bootstrapping-contract"></a>

The requirements of the bootstrapping stack depend on the stack synthesizer in use\. If you write your own stack synthesizer, you have complete control of the bootstrap resources that your synthesizer requires and how the synthesizer finds them\.

This section describes the expectations that the `DefaultStackSynthesizer` has of the bootstrapping template\.

### Versioning<a name="bootstrapping-contract-versioning"></a>

The template should contain a resource to create an SSM parameter with a well\-known name and an output to reflect the template's version\.

```
Resources:
  CdkBootstrapVersion:
    Type: AWS::SSM::Parameter
    Properties:
      Type: String
      Name:
        Fn::Sub: '/cdk-bootstrap/${Qualifier}/version'
      Value: 4
Outputs:
  BootstrapVersion:
    Value:
      Fn::GetAtt: [CdkBootstrapVersion, Value]
```

### Roles<a name="bootstrapping-contract-roles"></a>

The `DefaultStackSynthesizer` requires five IAM roles for five different purposes\. If you are not using the default roles, you must tell the synthesizer the ARNs for the roles you want to use\.

The roles are as follows:
+ The *deployment role* is assumed by the AWS CDK Toolkit and by AWS CodePipeline to deploy into an environment\. Its `AssumeRolePolicy` controls who can deploy into the environment\. In the template, you can see the permissions that this role needs\.
+ The *lookup role* is assumed by the AWS CDK Toolkit to perform context lookups in an environment\. Its `AssumeRolePolicy` controls who can deploy into the environment\. The permissions this role needs can be seen in the template\.
+ The *file publishing role* and the *image publishing role* are assumed by the AWS CDK Toolkit and by AWS CodeBuild projects to publish assets into an environment\. They're used to write to the S3 bucket and the ECR repository, respectively\. These roles require write access to these resources\.
+ *The AWS CloudFormation execution role* is passed to AWS CloudFormation to perform the actual deployment\. Its permissions are the permissions that the deployment will execute under\. The permissions are passed to the stack as a parameter that lists managed policy ARNs\.

### Outputs<a name="bootstrapping-contract-outputs"></a>

The AWS CDK Toolkit requires that the following CloudFormation outputs exist on the bootstrap stack\.
+ `BucketName`: the name of the file asset bucket
+ `BucketDomainName`: the file asset bucket in domain name format
+ `BootstrapVersion`: the current version of the bootstrap stack

### Template history<a name="bootstrap-template-history"></a>

The bootstrap template is versioned and evolves over time with the AWS CDK itself\. If you provide your own bootstrap template, keep it up to date with the canonical default template\. You want to make sure that your template continues to work with all CDK features\.

**Note**  
Earlier versions of the bootstrap template created an AWS KMS key in each bootstrapped environment by default\. To avoid charges for the KMS key, re\-bootstrap these environments using `--no-bootstrap-customer-key`\. The current default is no KMS key, which helps avoid these charges\. 

This section contains a list of the changes made in each version\.


| Template version | AWS CDK version | Changes | 
| --- | --- | --- | 
| 1 | 1\.40\.0 | Initial version of template with Bucket, Key, Repository, and Roles\. | 
| 2 | 1\.45\.0 | Split asset publishing role into separate file and image publishing roles\. | 
| 3 | 1\.46\.0 | Add FileAssetKeyArn export to be able to add decrypt permissions to asset consumers\. | 
| 4 | 1\.61\.0 | AWS KMS permissions are now implicit via Amazon S3 and no longer require FileAsetKeyArn\. Add CdkBootstrapVersion SSM parameter so the bootstrap stack version can be verified without knowing the stack name\. | 
| 5 | 1\.87\.0 | Deployment role can read SSM parameter\. | 
| 6 | 1\.108\.0 | Add lookup role separate from deployment role\. | 
| 6 | 1\.109\.0 | Attach aws\-cdk:bootstrap\-role tag to deployment, file publishing, and image publishing roles\.  | 
| 7 | 1\.110\.0 | Deployment role can no longer read Buckets in the target account directly\. \(However, this role is effectively an administrator, and could always use its AWS CloudFormation permissions to make the bucket readable anyway\)\. | 
| 8 | 1\.114\.0 | The lookup role has full read\-only permissions to the target environment, and has a aws\-cdk:bootstrap\-role tag as well\. | 
| 9 | 2\.1\.0 | Fixes Amazon S3 asset uploads from being rejected by commonly referenced encryption SCP\. | 
| 10 | 2\.4\.0 | Amazon ECR ScanOnPush is now enabled by default\. | 
| 11 | 2\.18\.0 | Adds policy allowing Lambda to pull from Amazon ECR repos so it survives re\-bootstrapping\. | 
| 12 | 2\.20\.0 | Adds support for experimental cdk import\. | 
| 13 | 2\.25\.0 | Makes container images in bootstrap\-created Amazon ECR repositories immutable\. | 
| 14 | 2\.34\.0 | Turns off Amazon ECR image scanning at the repository level by default to allow bootstrapping Regions that do not support image scanning\. | 
| 15 | 2\.60\.0 | KMS keys cannot be tagged\. | 
| 16 | 2\.69\.0 | Addresses Security Hub finding [KMS\.2](https://docs.aws.amazon.com/securityhub/latest/userguide/kms-controls.html#kms-2)\. | 
| 17 | 2\.72\.0 | Addresses Security Hub finding [ECR\.3](https://docs.aws.amazon.com/securityhub/latest/userguide/ecr-controls.html#ecr-3)\. | 
| 18 | 2\.80\.0 | Reverted changes made for version 16 as they don't work in all partitions and are are not recommended\. | 
| 19 | 2\.106\.1 | Reverted changes made to version 18 where AccessControl property was removed from the template\. \([\#27964](https://github.com/aws/aws-cdk/issues/27964)\) | 
| 20 | 2\.119\.0 | Add ssm:GetParameters action to the AWS CloudFormation deploy IAM role\. For more information, see [\#28336](https://github.com/aws/aws-cdk/pull/28336/files#diff-4fdac38426c4747aa17d515b01af4994d3d2f12c34f7b6655f24328259beb7bf)\. | 

## Security Hub Findings<a name="bootstrapping-securityhub"></a>

 If you are using AWS Security Hub, you may see findings reported on some of the resources created by the AWS CDK Bootstrapping process\. Security Hub findings help you find resource configurations you should double\-check for accuracy and safety\. We have reviewed these specific resource configurations with AWS Security and are confident they do not constitute a security problem\. 

### \[KMS\.2\] IAM principals should not have IAM inline policies that allow decryption actions on all KMS keys<a name="bootstrapping-securityhub-kms2"></a>

 The Deploy Role \(default name `cdk-hnb659fds-deploy-role-ACCOUNT-REGION`\) has permissions to read encrypted data stored in Amazon S3\. The policy does not give permission to any data by itself: only data read from Amazon S3 can be decrypted, and only from buckets that explicitly allow the Deploy Role to read from them via their Bucket Policy, and keys that explicitly allow the Deploy Role to decrypt using them using their Key Policy\. This statement is used to allow AWS CDK Pipelines to perform cross\-account deployments\. 

 ** Why does Security Hub flag this? ** The policy contains a `Resource: *` combined with a `Condition` clause; Security Hub is flagging the `*`\. The `*` is necessary because at the time the account is bootstrapped, the AWS KMS key created by AWS CDK Pipelines for the CodePipeline Artifact Bucket does not exist yet so we can't reference its ARN\. In addition, Security Hub does not include the `Condition` clause in the policy statement in its reasoning\. 

**What if I want to fix this finding?** As long as the resource policies on your AWS KMS keys are not unnecessarily permissive, the current Role policy does not allow the Deploy Role to access any more data than it should\. If you still want to get rid of the finding, you can do so by customizing the bootstrap stack \(using the process outlined above\) in one of these 2 ways:
+ If you are not using AWS CDK Pipelines for cross\-account deployments: remove the statement with `Sid: PipelineCrossAccountArtifactsBucket` from the deploy role; or
+ If you are using AWS CDK Pipelines for cross\-account deployments: after deploying your AWS CDK Pipeline, look up the AWS KMS Key ARN of the Artifact Bucket and replace the `Resource: *` of the `Sid: PipelineCrossAccountArtifactsBucket` statement with the actual Key ARN\.