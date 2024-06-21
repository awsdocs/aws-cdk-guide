# Configure security credentials for the AWS CDK CLI<a name="configure-access"></a>

When you use the AWS Cloud Development Kit \(AWS CDK\) to develop applications in your local environment, you will primarily use the AWS CDK Command Line Interface \(AWS CDK CLI\) to interact with AWS to deploy and manage your CDK stacks\. To use the CDK CLI to interact with AWS, you must configure security credentials on your local machine\. This lets AWS know who you are and what permissions you have\.

To learn more about security credentials, see [AWS security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) in the *IAM User Guide*\.

## Prerequisites<a name="configure-access-prerequisites"></a>

Configuring security credentials is part of the *getting started* process\. Complete all prerequisites and previous steps at [Getting started with the AWS CDK](getting_started.md)\.

## How to configure security credentials<a name="configure-access-how"></a>

How you configure security credentials depends on how you or your organization manages users\. We recommend that you use the AWS CLI to configure and manage security credentials for the CDK CLI\. This includes using AWS CLI commands like `aws configure` to configure security credentials on your local machine\. However, you can use alternative methods such as manually updating your `config` and `credentials` files, or setting environment variables\.

For guidance on configuring security credentials using the AWS CLI, along with information on configuration and credential precedence when using different methods, see [Authentication and access credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html) in the *AWS Command Line Interface User Guide*\. The CDK CLI adheres to the same configuration and credential precedence of the AWS CLI\. The `--profile` command line option takes precedence over environment variables\. If you have both the `AWS_PROFILE` and `CDK_DEFAULT_PROFILE` environment variables configured, the `AWS_PROFILE` environment variable takes precedence\.

If you prefer to quickly configure basic settings, including security credentials, see [Set up the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) in the *AWS Command Line Interface User Guide*\.

Once you’ve configured security credentials on your local machine, you can use the CDK CLI to interact with AWS\. For more information on using the CDK CLI with your method of managing users, see the following:
+ [Configure and manage CDK CLI security credentials for IAM Identity Center users](configure-access-sso.md)\.

## Additional information<a name="configure-access-info"></a>

To learn about the different ways that you can sign in to AWS, depending on the type of user you are, see [What is AWS Sign\-In?](https://docs.aws.amazon.com/signin/latest/userguide/what-is-sign-in.html) in the *AWS Sign\-In User Guide*\.

For reference information when using AWS SDKs and tools, including the AWS CLI, see the [AWS SDKs and Tools Reference Guide](https://docs.aws.amazon.com/sdkref/latest/guide/overview.html)\.