# AWS CDK bootstrapping<a name="bootstrapping"></a>

*Bootstrapping* is the process of preparing your AWS environment for usage with the AWS Cloud Development Kit \(AWS CDK\)\. Before you deploy a CDK stack into an AWS environment, the environment must first be bootstrapped\.

## What is bootstrapping?<a name="bootstrapping-what"></a>

Bootstrapping prepares your AWS environment by provisioning specific AWS resources in your environment that are used by the AWS CDK\. These resources are commonly referred to as your *bootstrap resources*\. They include the following:
+ **Amazon Simple Storage Service \(Amazon S3\) bucket** – Used to store your CDK project files, such as AWS Lambda function code and assets\.
+ **Amazon Elastic Container Registry \(Amazon ECR\) repository** – Used primarily to store Docker images\.
+ **AWS Identity and Access Management \(IAM\) roles** – Configured to grant permissions needed by the AWS CDK to perform deployments\. For more information about the IAM roles created during bootstrapping, see [IAM roles created during bootstrapping](bootstrapping-env.md#bootstrapping-env-roles)\.

## How does bootstrapping work?<a name="bootstrapping-how"></a>

Resources and their configuration that are used by the CDK are defined in an AWS CloudFormation template\. This template is created and managed by the CDK team\. For the latest version of this template, see `[bootstrap\-template\.yaml](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk/lib/api/bootstrap/bootstrap-template.yaml)` in the *aws\-cdk GitHub repository*\.

To bootstrap an environment, you use the AWS CDK Command Line Interface \(AWS CDK CLI\) `cdk bootstrap` command\. The CDK CLI retrieves the template and deploys it to AWS CloudFormation as a stack, known as the *bootstrap stack*\. By default, the stack name is `CDKToolkit`\. By deploying this template, CloudFormation provisions the resources in your environment\. After deployment, the bootstrap stack will appear in the AWS CloudFormation console of your environment\.

You can also customize bootstrapping by modifying the template or by using CDK CLI options with the `cdk bootstrap` command\.

AWS environments are independent\. Each environment that you want to use with the AWS CDK must first be bootstrapped\.

## Learn more<a name="bootstrapping-learn"></a>

For instructions on bootstrapping your environment, see [Bootstrap your environment for use with the AWS CDK](bootstrapping-env.md)\.