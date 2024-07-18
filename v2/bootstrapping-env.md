# Bootstrap your environment for use with the AWS CDK<a name="bootstrapping-env"></a>

Bootstrap your AWS environment to prepare it for AWS Cloud Development Kit \(AWS CDK\) stack deployments\.
+ For an introduction to environments, see [Environments](environments.md)\.
+ For an introduction to bootstrapping, see [Bootstrapping](bootstrapping.md)\.

**Topics**
+ [How to bootstrap your environment](#bootstrapping-howto)
+ [When to bootstrap your environment](#bootstrapping-env-when)
+ [Customize bootstrapping](#bootstrapping-env-customize)
+ [Bootstrapping with CDK Pipelines](#bootstrapping-env-pipelines)
+ [Bootstrap template version history](#bootstrap-template-history)
+ [Upgrade from legacy to modern bootstrap template](#bootstrapping-template)
+ [Address Security Hub Findings](#bootstrapping-securityhub)
+ [Considerations](#bootstrapping-env-considerations)
+ [Customize AWS CDK bootstrapping](bootstrapping-customizing.md)

## How to bootstrap your environment<a name="bootstrapping-howto"></a>

You can use the AWS CDK Command Line Interface \(AWS CDK CLI\) or your preferred AWS CloudFormation deployment tool to bootstrap your environment\.

### Use the CDK CLI<a name="bootstrapping-howto-cli"></a>

You can use the CDK CLI `cdk bootstrap` command to bootstrap your environment\. This is the method that we recommend if you don't require significant modifications to bootstrapping\.

**Bootstrap from any working directory**  
To bootstrap from any working directory, provide the environment to bootstrap as a command line argument\. The following is an example:  

```
$ cdk bootstrap aws://123456789012/us-east-1
```
If you don't have your AWS account number, you can get it from the AWS Management Console\. You can also use the following AWS CLI command to display your default account information, including your account number:  

```
$ aws sts get-caller-identity
```
If you have named profiles in your AWS `config` and `credentials` files, use the `--profile` option to retrieve account information for a specific profile\. The following is an example:  

```
$ aws sts get-caller-identity --profile prod
```
To display the default Region, use the `aws configure get` command:  

```
$ aws configure get region
$ aws configure get region --profile prod
```
When providing an argument, the `aws://` prefix is optional\. The following is valid:  

```
$ cdk bootstrap 123456789012/us-east-1
```
To bootstrap multiple environments at the same time, provide multiple arguments:  

```
$ cdk bootstrap aws://123456789012/us-east-1 aws://123456789012/us-east-2
```

**Bootstrap from the parent directory of a CDK project**  
You can run `cdk bootstrap` from the parent directory of a CDK project containing a `cdk.json` file\. If you don’t provide an environment as an argument, the CDK CLI will obtain environment information from default sources, such as your `config` and `credentials` files or any environment information specified for your CDK stack\.  
When you bootstrap from the parent directory of a CDK project, environments provided from command line arguments take precedence over other sources\.  
To bootstrap an environment that is specified in your `config` and `credentials` files, use the `--profile` option:  

```
$ cdk bootstrap --profile prod
```

For more information on the `cdk bootstrap` command and supported options, see [cdk bootstrap](ref-cli-cmd-bootstrap.md)\.

### Use any AWS CloudFormation tool<a name="bootstrapping-howto-cfn"></a>

You can copy the [bootstrap template](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk/lib/api/bootstrap/bootstrap-template.yaml) from the *aws\-cdk GitHub repository* or obtain the template with the `cdk bootstrap --show-template` command\. Then, use any AWS CloudFormation tool to deploy the template into your environment\.

With this method, you can use AWS CloudFormation StackSets or AWS Control Tower\. You can also use the AWS CloudFormation console or the AWS Command Line Interface \(AWS CLI\)\. You can make modifications to your template before you deploy it\. This method may be more flexible and suitable for large\-scale deployments\.

The following is an example of using the `--show-template` option to retrieve and save the bootstrap template to your local machine:

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

To deploy this template using the CDK CLI, you can run the following:

```
$ cdk bootstrap --template bootstrap-template.yaml
```

The following is an example of using the AWS CLI to deploy the template:

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

For information on using CloudFormation StackSets to bootstrap multiple environments, see [Bootstrapping multiple AWS accounts for AWS CDK using CloudFormation StackSets](https://aws.amazon.com/blogs/mt/bootstrapping-multiple-aws-accounts-for-aws-cdk-using-cloudformation-stacksets/) in the *AWS Cloud Operations & Migrations Blog*\.

## When to bootstrap your environment<a name="bootstrapping-env-when"></a>

You must bootstrap each environment before you deploy into the environment\. If you attempt to deploy a CDK stack into an environment that hasn’t been bootstrapped, you will see an error like the following:

```
$ cdk deploy

✨  Synthesis time: 2.02s

 ❌ Deployment failed: Error: BootstrapExampleStack: SSM parameter /cdk-bootstrap/hnb659fds/version not found. Has the environment been bootstrapped? Please run 'cdk bootstrap' (see https://docs.aws.amazon.com/cdk/latest/guide/bootstrapping.html)
```

It’s okay to bootstrap an environment more than once\. If an environment has already been bootstrapped, the bootstrap stack will be upgraded if necessary\. Otherwise, nothing will happen\.

### Update your bootstrap stack<a name="bootstrapping-env-when-update"></a>

Periodically, the CDK team will update the bootstrap template to a new version\. When this happens, we recommend that you update your bootstrap stack\. If you haven’t customized the bootstrapping process, you can update your bootstrap stack by following the same steps that you took to originally bootstrap your environment\. For more information, see [Bootstrap template version history](#bootstrap-template-history)\.

## Customize bootstrapping<a name="bootstrapping-env-customize"></a>

If the default bootstrap template doesn’t suit your needs, you can customize the bootstrapping of resources into your environment in the following ways:
+ Use command line options with the `cdk bootstrap` command – This method is best for making small, specific changes that are supported through command line options\.
+ Modify the default bootstrap template and deploy it – This method is best for making complex changes or if you want complete control over the configuration of resources provisioned during bootstrapping\.

For more information on customizing bootstrapping, see [Customize AWS CDK bootstrapping](bootstrapping-customizing.md)\.

## Bootstrapping with CDK Pipelines<a name="bootstrapping-env-pipelines"></a>

If you are using CDK Pipelines to deploy into another account's environment, and you receive a message like the following:

```
Policy contains a statement with one or more invalid principals
```

This error message means that the appropriate IAM roles do not exist in the other environment\. The most likely cause is that the environment has not been bootstrapped\. Bootstrap the environment and try again\.

### Protecting your bootstrap stack from deletion<a name="bootstrapping-env-pipelines-protect"></a>

If a bootstrap stack is deleted, the AWS resources that were originally provisioned in the environment to support CDK deployments will also be deleted\. This will cause the pipeline to stop working\. If this happens, there is no general solution for recovery\.

After your environment is bootstrapped, do not delete and recreate the environment’s bootstrap stack\. Instead, try to update the bootstrap stack to a new version by running the `cdk bootstrap` command again\.

To protect against accidental deletion of your bootstrap stack, we recommend that you provide the `--termination-protection` option with the `cdk bootstrap` command to enable termination protection\. You can enable termination protection on new or existing bootstrap stacks\. For instructions on enabling termination protection, see [Enable termination protection for the bootstrap stack](bootstrapping-customizing.md#bootstrapping-customizing-cli-protection)\.

## Bootstrap template version history<a name="bootstrap-template-history"></a>

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
| 21 | 2\.149\.0 | Add condition to the file publishing role\. | 

## Upgrade from legacy to modern bootstrap template<a name="bootstrapping-template"></a>

The AWS CDK v1 supported two bootstrapping templates, legacy and modern\. CDK v2 supports only the modern template\. For reference, here are the high\-level differences between these two templates\.

[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/cdk/v2/guide/bootstrapping-env.html)

\* *We will add additional resources to the bootstrap template as needed\.*

An environment that was bootstrapped using the legacy template must be upgraded to use the modern template for CDK v2 by re\-bootstrapping\. Re\-deploy all AWS CDK applications in the environment at least once before deleting the legacy bucket\.

## Address Security Hub Findings<a name="bootstrapping-securityhub"></a>

 If you are using AWS Security Hub, you may see findings reported on some of the resources created by the AWS CDK bootstrapping process\. Security Hub findings help you find resource configurations you should double\-check for accuracy and safety\. We have reviewed these specific resource configurations with AWS Security and are confident they do not constitute a security problem\. 

### \[KMS\.2\] IAM principals should not have IAM inline policies that allow decryption actions on all KMS keys<a name="bootstrapping-securityhub-kms2"></a>

The *deploy role* \(`DeploymentActionRole`\) grants permission to read encrypted data, which is necessary for cross\-account deployments with CDK Pipelines\. Policies in this role do not grant permission to all data\. It only grants permission to read encrypted data from Amazon S3 and AWS KMS, and only when those resources allow it through their bucket or key policy\.

The following is a snippet of these two statements in the *deploy role* from the bootstrap template:

```
DeploymentActionRole:
    Type: AWS::IAM::Role
    Properties:
      ...
      Policies:
        - PolicyDocument:
            Statement:
              ...
              - Sid: PipelineCrossAccountArtifactsBucket
                Effect: Allow
                Action:
                  - s3:GetObject*
                  - s3:GetBucket*
                  - s3:List*
                  - s3:Abort*
                  - s3:DeleteObject*
                  - s3:PutObject*
                Resource: "*"
                Condition:
                  StringNotEquals:
                    s3:ResourceAccount:
                      Ref: AWS::AccountId
              - Sid: PipelineCrossAccountArtifactsKey
                Effect: Allow
                Action:
                  - kms:Decrypt
                  - kms:DescribeKey
                  - kms:Encrypt
                  - kms:ReEncrypt*
                  - kms:GenerateDataKey*
                Resource: "*"
                Condition:
                  StringEquals:
                    kms:ViaService:
                      Fn::Sub: s3.${AWS::Region}.amazonaws.com
              ...
```

#### Why does Security Hub flag this?<a name="bootstrapping-securityhub-kms2-why"></a>

The policies contain a `Resource: *` combined with a `Condition` clause\. Security Hub flags the `*` wildcard\. This wildcard is used because at the time the account is bootstrapped, the AWS KMS key created by CDK Pipelines for the CodePipeline artifact bucket does not exist yet, and therefore, can’t be referenced on the bootstrap template by ARN\. In addition, Security Hub does not consider the `Condition` clause when raising this flag\. This `Condition` restricts `Resource: *` to requests made from the same AWS account of the AWS KMS key\. These requests must come from Amazon S3 in the same AWS Region as the AWS KMS key\. 

#### Do I need to fix this finding?<a name="bootstrapping-securityhub-kms2-decide"></a>

As long as you have not modified the AWS KMS key on your bootstrap template to be overly permissive, the *deploy role* does not allow more access than it needs\. Therefore, it is not necessary to fix this finding\.

#### What if I want to fix this finding?<a name="bootstrapping-securityhub-kms2-fix"></a>

How you fix this finding depends on whether or not you will be using CDK Pipelines for cross\-account deployments\.

**To fix the Security Hub finding and use CDK Pipelines for cross\-account deployments**

1. If you have not done so, deploy the CDK bootstrap stack using the `cdk bootstrap` command\.

1. If you have not done so, create and deploy your CDK Pipeline\. For instructions, see [Continuous integration and delivery \(CI/CD\) using CDK Pipelines](cdk_pipeline.md)\.

1. Obtain the AWS KMS key ARN of the CodePipeline artifact bucket\. This resource is created during pipeline creation\.

1. Obtain a copy of the CDK bootstrap template to modify it\. The following is an example, using the AWS CDK CLI:

   ```
   $ cdk bootstrap --show-template > bootstrap-template.yaml
   ```

1. Modify the template by replacing `Resource: *` of the `PipelineCrossAccountArtifactsKey` statement with your ARN value\.

1. Deploy the template to update your bootstrap stack\. The following is an example, using the CDK CLI:

   ```
   $ cdk bootstrap aws://account-id/region --template bootstrap-template.yaml
   ```

**To fix the Security Hub finding if you’re not using CDK Pipelines for cross\-account deployments**

1. Obtain a copy of the CDK bootstrap template to modify it\. The following is an example, using the CDK CLI:

   ```
   $ cdk bootstrap --show-template > bootstrap-template.yaml
   ```

1. Delete the `PipelineCrossAccountArtifactsBucket` and `PipelineCrossAccountArtifactsKey` statements from the template\.

1. Deploy the template to update your bootstrap stack\. The following is an example, using the CDK CLI:

   ```
   $ cdk bootstrap aws://account-id/region --template bootstrap-template.yaml
   ```

## Considerations<a name="bootstrapping-env-considerations"></a>

Since bootstrapping provisions resources in your environment, you may incur AWS charges when those resources are used with the AWS CDK\.