# Configure security credentials for the AWS CDK CLI<a name="configure-access"></a>

When you use the AWS Cloud Development Kit \(AWS CDK\) to develop applications in your local environment, you will primarily use the AWS CDK Command Line Interface \(AWS CDK CLI\) to interact with AWS\. For example, you can use the CDK CLI to deploy your application or to delete your resources from your AWS environment\.

To use the CDK CLI to interact with AWS, you must configure security credentials on your local machine\. This lets AWS know who you are and what permissions you have\.

To learn more about security credentials, see [AWS security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) in the *IAM User Guide*\.

## Prerequisites<a name="configure-access-prerequisites"></a>

Configuring security credentials is part of the *getting started* process\. Complete all prerequisites and previous steps at [Getting started with the AWS CDK](getting_started.md)\.

## How to configure security credentials<a name="configure-access-how"></a>

How you configure security credentials depends on how you or your organization manages users\. Whether you use AWS Identity and Access Management \(IAM\) or AWS IAM Identity Center, we recommend that you use the AWS Command Line Interface \(AWS CLI\) to configure and manage security credentials for the CDK CLI\. This includes using AWS CLI commands like `aws configure` to configure security credentials on your local machine\. However, you can use alternative methods such as manually updating your `config` and `credentials` files, or setting environment variables\.

For guidance on configuring security credentials using the AWS CLI, along with information on configuration and credential precedence when using different methods, see [Authentication and access credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html) in the *AWS Command Line Interface User Guide*\. The CDK CLI adheres to the same configuration and credential precedence of the AWS CLI\. The `--profile` command line option takes precedence over environment variables\. If you have both the `AWS_PROFILE` and `CDK_DEFAULT_PROFILE` environment variables configured, the `AWS_PROFILE` environment variable takes precedence\.

If you configure multiple profiles, you can use the CDK CLI `\-\-profile` option with any command to specify the profile from your `credentials` and `config` files to use for authentication\. If you don't provide `--profile`, the `default` profile will be used\.

If you prefer to quickly configure basic settings, including security credentials, see [Set up the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) in the *AWS Command Line Interface User Guide*\.

Once you’ve configured security credentials on your local machine, you can use the CDK CLI to interact with AWS\.

## Configure and manage security credentials for IAM Identity Center users<a name="configure-access-sso"></a>

IAM Identity Center users can authenticate with IAM Identity Center or manually by using short\-term credentials\.

### Authenticate with IAM Identity Center to generate short\-term credentials<a name="configure-access-sso-auto"></a>

You can configure the AWS CLI to authenticate with IAM Identity Center\. This is the recommended approach of configuring security credentials for IAM Identity Center users\. IAM Identity Center users can use the AWS CLI `aws configure sso` wizard to configure an IAM Identity Center profile and `sso-session`, which gets stored in the `config` file on your local machine\. For instructions, see [Configure the AWS CLI to use AWS IAM Identity Center](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html) in the *AWS Command Line Interface User Guide*\.

Next, you can use the AWS CLI `aws sso login` command to request refreshed credentials\. You can also use this command to switch profiles\. For instructions, see [Use an IAM Identity Center named profile](https://docs.aws.amazon.com/cli/latest/userguide/sso-using-profile.html) in the *AWS Command Line Interface User Guide*\.

Once authenticated, you can use the CDK CLI to interact with AWS for the duration of your session\. For an example, see [Example: Authenticate with IAM Identity Center automatic token refresh for use with the AWS CDK CLI](configure-access-sso-example-cli.md)\.

### Manually configure short\-term credentials<a name="configure-access-sso-manual"></a>

As an alternative to using the AWS CLI and authenticating with IAM Identity Center, IAM Identity Center users can obtain short\-term credentials from the AWS Management Console and manually configure the `credentials` and `config` files on their local machine\. Once configured, you can use the CDK CLI to interact with AWS until your credentials expire\. For instructions, see [Authenticate with short\-term credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html) in the *AWS Command Line Interface User Guide*\.

## Configure and manage security credentials for IAM users<a name="configure-access-iam"></a>

IAM users can use an IAM role or IAM user credentials with the CDK CLI\.

### Use an IAM role to configure short\-term credentials<a name="configure-access-iam-role"></a>

IAM users can assume IAM roles to gain additional \(or different\) permissions\. For IAM users, this is the recommended approach since it provides short\-term credentials\.

First, the IAM role and user’s permission to assume the role must be configured\. This is typically performed by an administrator using the AWS Management Console or AWS CLI\. Then, the IAM user can use the AWS CLI to assume the role and configure short\-term credentials on their local machine\. For instructions, see [Use an IAM role in the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-role.html) in the *AWS Command Line Interface User Guide*\.

### Use IAM user credentials<a name="configure-access-iam-user"></a>

**Warning**  
To avoid security risks, we don’t recommend using IAM user credentials since they provide long\-term access\. If you must use long\-term credentials, we recommend that you [update access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#rotate-credentials) as an IAM security best practice\.

IAM users can obtain access keys from the AWS Management Console\. You can then use the AWS CLI to configure long\-term credentials on your local machine\. For instructions, see [Authenticate with IAM user credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html) in the *AWS Command Line Interface User Guide*\.

## Additional information<a name="configure-access-info"></a>

To learn about the different ways that you can sign in to AWS, depending on the type of user you are, see [What is AWS Sign\-In?](https://docs.aws.amazon.com/signin/latest/userguide/what-is-sign-in.html) in the *AWS Sign\-In User Guide*\.

For reference information when using AWS SDKs and tools, including the AWS CLI, see the [AWS SDKs and Tools Reference Guide](https://docs.aws.amazon.com/sdkref/latest/guide/overview.html)\.