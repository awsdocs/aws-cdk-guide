# Configure and manage CDK CLI security credentials for IAM Identity Center users<a name="configure-access-sso"></a>

Users that are created and managed with AWS IAM Identity Center can generate short\-term credentials for the AWS CDK Command Line Interface \(AWS CDK CLI\)\.

For an introduction to security credentials with the CDK CLI, including prerequisites, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\.

**Topics**
+ [Configure the AWS CLI to use IAM Identity Center](#configure-access-sso-cli)
+ [Use the AWS CLI to obtain short\-term credentials](#configure-access-sso-obtain)
+ [Manually configure short\-term credentials](#configure-access-sso-manual)
+ [Use short\-term credentials with the CDK CLI](#configure-access-sso-cdk)
+ [Example: Use the AWS CLI to configure short\-term security credentials as an IAM Identity Center user](configure-access-sso-example-cli.md)

## Configure the AWS CLI to use IAM Identity Center<a name="configure-access-sso-cli"></a>

You can configure the AWS CLI to authenticate with IAM Identity Center\. This is the recommended approach of configuring security credentials for IAM Identity Center users\. The IAM Identity Center information needed to generate short\-term credentials are stored in the `config` file on your local machine\. You will use the AWS CLI `aws configure sso` to configure this file\. For more information, see [Configure the AWS CLI to use AWS IAM Identity Center](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html) in the *AWS Command Line Interface User Guide*\.

After completing those instructions, your `config` file should have at least one named profile and `sso-session`\. You can use this information to request and refresh short\-term credentials for your profile\.

## Use the AWS CLI to obtain short\-term credentials<a name="configure-access-sso-obtain"></a>

You can use the AWS CLI `aws sso login` command to obtain short\-term credentials for your profile\. You can also use the `aws sso login` command to switch profiles\. For instructions on obtaining short\-term credentials and switching profiles, see [Use an IAM Identity Center named profile](https://docs.aws.amazon.com/cli/latest/userguide/sso-using-profile.html) in the *AWS Command Line Interface User Guide*\.

## Manually configure short\-term credentials<a name="configure-access-sso-manual"></a>

As an alternative to using the AWS CLI, you can obtain your short\-term credentials from the AWS access portal and manually update your `credentials` and `config` files to configure security credentials\. For instructions, see [Authenticate with short\-term credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-short-term.html) in the *AWS Command Line Interface User Guide*\.

## Use short\-term credentials with the CDK CLI<a name="configure-access-sso-cdk"></a>

The CDK CLI uses the short\-term credentials that you generate with the AWS CLI or configure manually in the `credentials` and `config` files\. Once you’ve configured short\-term credentials, you can use the CDK CLI to interact with AWS\.

When you use the CDK CLI to interact with your CDK stacks on AWS, use the `\-\-profile` option with any CDK CLI command to specify the named profile that you used to generate short\-term credentials\. If you've configured the `default` profile as your IAM Identity Center profile, you don’t have to specify a profile\.