# Customize AWS CDK bootstrapping<a name="bootstrapping-customizing"></a>

You can customize AWS Cloud Development Kit \(AWS CDK\) bootstrapping by using the AWS CDK Command Line Interface \(AWS CDK CLI\) or by modifying and deploying the AWS CloudFormation bootstrap template\.

For an introduction to bootstrapping, see [Bootstrapping](bootstrapping.md)\.

**Topics**
+ [Use the CDK CLI to customize bootstrapping](#bootstrapping-customizing-cli)
+ [Modify the default bootstrap template](#bootstrapping-customizing-template)
+ [Follow the bootstrap template contract](#bootstrapping-contract)

## Use the CDK CLI to customize bootstrapping<a name="bootstrapping-customizing-cli"></a>

The following are a few examples of how you can customize bootstrapping by using the CDK CLI\. For a list of all `cdk bootstrap` options, see [cdk bootstrap](ref-cli-cmd-bootstrap.md)\.

**Override the name of the Amazon S3 bucket**  <a name="bootstrapping-customizing-cli-s3-name"></a>
Use the `--bootstrap-bucket-name` option to override the default Amazon S3 bucket name\. This may require that you modify template synthesis\. For more information, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.

**Modify server\-side encryption keys for the Amazon S3 bucket**  <a name="bootstrapping-customizing-keys"></a>
By default, the Amazon S3 bucket in the bootstrap stack is configure to use AWS managed keys for server\-side encryption\. To use an existing customer managed key, use the `--bootstrap-kms-key-id` option and provide a value for the AWS Key Management Service \(AWS KMS\) key to use\. If you want more control over the encryption key, provide `--bootstrap-customer-key` to use a customer managed key\.

**Attach managed policies to the deployment role assumed by AWS CloudFormation**  <a name="bootstrapping-customizing-cli-deploy-role"></a>
By default, stacks are deployed with full administrator permissions using the `AdministratorAccess` policy\. To use your own managed policies, use the `--cloudformation-execution-policies` option and provide the ARNs of the managed policies to attach to the deployment role\.  
To provide multiple policies, pass them a single string, separated by commas:  

```
$ cdk bootstrap --cloudformation-execution-policies "arn:aws:iam::aws:policy/AWSLambda_FullAccess,arn:aws:iam::aws:policy/AWSCodeDeployFullAccess"
```
To avoid deployment failures, be sure that the policies you specify are sufficient for any deployments that you will perform into the environment being bootstrapped\.

**Change the qualifier that is added to the names of resources in your bootstrap stack**  <a name="bootstrapping-customizing-cli-qualifier"></a>
By default, the `hnb659fds` qualifier is added to the physical ID of resources in your bootstrap stack\. To change this value, use the `--qualifier` option\.  
This modification is useful when provisioning multiple bootstrap stacks in the same environment to avoid name clashes\.  
Changing the qualifier is intended for name isolation between automated tests of the CDK itself\. Unless you can very precisely scope down the IAM permissions given to the CloudFormation execution role, there are no permission isolation benefits to having two different bootstrap stacks in a single account\. Therefore, there’s usually no need to change this value\.  
When you change the qualifier, your CDK app must pass the changed value to the stack synthesizer\. For more information, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.

**Add tags to the bootstrap stack**  <a name="bootstrapping-customizing-cli-tags"></a>
Use the `--tags` option in the format of `KEY=VALUE` to add CloudFormation tags to your bootstrap stack\.

**Specify additional AWS accounts that can deploy into the environment being bootstrapped**  <a name="bootstrapping-customizing-cli-accounts-deploy"></a>
Use the `--trust` option to provide additional AWS accounts that are allowed to deploy into the environment being bootstrapped\. By default, the account performing the bootstrapping will always be trusted\.  
This option is useful when you are bootstrapping an environment that a CDK Pipeline from another environment will deploy into\.  
When you use this option, you must also provide `--cloudformation-execution-policies`\.  
To add trusted accounts to an existing bootstrap stack, you must specify all of the accounts to trust, including those that you may have previously provided\. If you only provide new accounts to trust, the previously trusted accounts will be removed\.  
The following is an example that trusts two accounts:  

```
$ cdk bootstrap aws://123456789012/us-west-2 --trust 234567890123 --trust 987654321098 --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess
 ⏳  Bootstrapping environment aws://123456789012/us-west-2...
Trusted accounts for deployment: 234567890123, 987654321098
Trusted accounts for lookup: (none)
Execution policies: arn:aws:iam::aws:policy/AdministratorAccess
CDKToolkit: creating CloudFormation changeset...
 ✅  Environment aws://123456789012/us-west-2 bootstrapped.
```

**Specify additional AWS accounts that can look up information in the environment being bootstrapped**  <a name="bootstrapping-customizing-cli-accounts-lookup"></a>
Use the `--trust-for-lookup` option to specify AWS accounts that are allowed to look up context information from the environment being bootstrapped\. This option is useful to give accounts permission to synthesize stacks that will be deployed into the environment, without actually giving them permission to deploy those stacks directly\.

**Enable termination protection for the bootstrap stack**  <a name="bootstrapping-customizing-cli-protection"></a>
If a bootstrap stack is deleted, the AWS resources that were originally provisioned in the environment will also be deleted\. After your environment is bootstrapped, we recommend that you don't delete and recreate the environment’s bootstrap stack, unless you are intentionally doing so\. Instead, try to update the bootstrap stack to a new version by running the `cdk bootstrap` command again\.   
Use the `--termination-protection` option to manage termination protection settings for the bootstrap stack\. By enabling termination protection, you prevent the bootstrap stack and its resources from being accidentally deleted\. This is especially important if you use CDK Pipelines since there is no general recovery option if you accidentally delete the bootstrap stack\.  
After enabling termination protection, you can use the AWS CLI or AWS CloudFormation console to verify\.  

**To enable termination protection**

1. Run the following command to enable termination protection on a new or existing bootstrap stack:

   ```
   $ cdk bootstrap --termination-protection
   ```

1. Use the AWS CLI or CloudFormation console to verify\. The following is an example, using the AWS CLI\. If you modified your bootstrap stack name, replace `CDKToolkit` with your stack name:

   ```
   $ aws cloudformation describe-stacks --stack-name CDKToolkit --query "Stacks[0].EnableTerminationProtection"
   true
   ```

## Modify the default bootstrap template<a name="bootstrapping-customizing-template"></a>

When you need more customization than the CDK CLI can provide, you can modify the bootstrap template as needed\. Then, deploy the template to bootstrap your environment\.

**To modify and deploy the default bootstrap template**

1. Obtain the default bootstrap template using the `--show-template` option\. By default, the CDK CLI will output the template in your terminal window\. You can modify the CDK CLI command to save the template to your local machine\. The following is an example:

   ```
   $ cdk bootstrap --show-template > my-bootstrap-template.yaml
   ```

1. Modify the bootstrap template as needed\. Any changes that you make should adhere to the bootstrapping template contract\. For more information on the bootstrapping template contract, see [Follow the bootstrap template contract](#bootstrapping-contract)\.

   To ensure that your customizations are not accidentally overwritten later by someone running `cdk bootstrap` using the default template, change the default value of the `BootstrapVariant` template parameter\. The CDK CLI will only allow overwriting the bootstrap stack with templates that have the same `BootstrapVariant` and an equal or higher version than the template that is currently deployed\.

1. Deploy your modified template using your preferred AWS CloudFormation deployment method\. The following is an example that uses the CDK CLI:

   ```
   $ cdk bootstrap --template my-bootstrap-template.yaml
   ```

## Follow the bootstrap template contract<a name="bootstrapping-contract"></a>

When you customize bootstrapping, you may need to customize stack synthesis behavior\. This ensures that your synthesized CloudFormation template remains compatible with your bootstrap stack\. For more information, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.

The simplest method to customize stack synthesis is by modifying the `DefaultStackSynthesizer` class in your `Stack` instance\. If you require customization beyond what this class can offer, you can write your own synthesizer as a class that implements `[IStackSynthesizer](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.IStackSynthesizer.html)` \(perhaps deriving from `DefaultStackSynthesizer`\)\.

When you customize bootstrapping, follow the bootstrap template contract to remain compatible with `DefaultStackSynthesizer`\. If you modify bootstrapping beyond the bootstrap template contract, you will need to write your own synthesizer\.

### Versioning<a name="bootstrapping-contract-versioning"></a>

The bootstrap template should contain a resource to create an Amazon EC2 Systems Manager \(SSM\) parameter with a well\-known name and an output to reflect the template's version:

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

The `DefaultStackSynthesizer` requires five IAM roles for five different purposes\. If you are not using the default roles, you must specify your IAM role ARNs within your `DefaultStackSynthesizer` object\. The roles are as follows:
+ The *deployment role* is assumed by the CDK CLI and by AWS CodePipeline to deploy into an environment\. Its `AssumeRolePolicy` controls who can deploy into the environment\. In the template, you can see the permissions that this role needs\.
+ The *lookup role* is assumed by the CDK CLI to perform context lookups in an environment\. Its `AssumeRolePolicy` controls who can deploy into the environment\. The permissions this role needs can be seen in the template\.
+ The *file publishing role* and the *image publishing role* are assumed by the CDK CLI and by AWS CodeBuild projects to publish assets into an environment\. They're used to write to the Amazon S3 bucket and the Amazon ECR repository, respectively\. These roles require write access to these resources\.
+ *The AWS CloudFormation execution role* is passed to AWS CloudFormation to perform the actual deployment\. Its permissions are the permissions that the deployment will execute under\. The permissions are passed to the stack as a parameter that lists managed policy ARNs\.

### Outputs<a name="bootstrapping-contract-outputs"></a>

The CDK CLI requires that the following CloudFormation outputs exist on the bootstrap stack:
+ `BucketName` – The name of the file asset bucket\.
+ `BucketDomainName` – The file asset bucket in domain name format\.
+ `BootstrapVersion` – The current version of the bootstrap stack\.

### Template history<a name="bootstrapping-contract-history"></a>

The bootstrap template is versioned and evolves over time with the AWS CDK itself\. If you provide your own bootstrap template, keep it up to date with the canonical default template\. You want to make sure that your template continues to work with all CDK features\. For more information, see [Bootstrap template version history](bootstrapping-env.md#bootstrap-template-history)\.