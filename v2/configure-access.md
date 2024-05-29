# Configure security credentials for the AWS CDK CLI<a name="configure-access"></a>

When you use the AWS Cloud Development Kit \(AWS CDK\) to develop applications in your local environment, you will primarily use the AWS CDK Command Line Interface \(AWS CDK CLI\) to interact with AWS to deploy and manage your CDK stacks\. To use the CDK CLI to interact with AWS, you must configure security credentials on your local machine\. This lets AWS know who you are and what permissions you have\.

To learn more about security credentials, see [AWS security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) in the *IAM User Guide*\.

**Topics**
+ [Prerequisites](#configure-access-prerequisites)
+ [How to configure security credentials](#configure-access-how)
+ [Additional information](#configure-access-info)
+ [Configure and manage CDK CLI security credentials for IAM Identity Center users](configure-access-sso.md)

## Prerequisites<a name="configure-access-prerequisites"></a>

If you are new to AWS or the AWS CDK, complete prerequisites in this section\.

### Create an AWS account and administrative user<a name="configure-access-prerequisites-account"></a>

Before you determine your method of authenticating with security credentials, you or your organization must have an AWS account and administrative user\. To learn more, see [Getting set up with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-set-up.html) in the *IAM User Guide*\.

You can manage IAM using different methods, such as through the AWS console, the AWS Command Line Interface \(AWS CLI\), or through application interfaces \(APIs\) in the associated SDKs\. When using IAM with the CDK CLI, you will primarily use the AWS CLI to configure and manage security credentials\. To learn more, see [AWS Command Line Interface \(CLI\) and Software Development Kits \(SDKs\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/management_methods.html#management-method-cli-sdk) in the *IAM User Guide*\.

### Determine your method of creating and managing users<a name="configure-access-prerequisites-method"></a>

Once you’ve created your AWS account and administrative user, you’ll want to determine your method of managing users, adhering to the concept of *least\-privilege permissions* for your users\. This is an IAM [best practice](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) recommendation\. To learn about the different methods of managing users, see [Overview of AWS identity management: Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_identity-management.html) in the *IAM User Guide*\.

If your organization has a method of managing users, follow their guidance\. Otherwise, we recommend using AWS IAM Identity Center to create and manage users\. With IAM Identity Center, you can manage AWS accounts, users, and permissions from a centrally managed service\. You can also provide your users with short\-term credentials for authentication with AWS\. For an introduction, see [What is IAM Identity Center?](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) in the *AWS IAM Identity Center User Guide*\.

Once you begin creating users, they will need to configure security credentials on their local machine\. Users can use the AWS CLI to accomplish this\.

### Install the AWS CLI<a name="configure-access-prerequisites-cli"></a>

As a user, you use the AWS CLI to create and manage configuration and credential files on your local machine\. These files are used to store, manage, and generate security credentials that you can use with the CDK CLI\.

To install the AWS CLI, see [Install or update to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) in the *AWS Command Line Interface User Guide*\.

To learn more about these files, see [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) in the *AWS Command Line Interface User Guide*\.

### Install the AWS CDK CLI<a name="configure-access-prerequisites-cdk"></a>

Use the Node Package Manager to install the CDK CLI\. In most cases, we recommend installing the latest version globally by running the following:

```
$ npm install -g aws-cdk
```

## How to configure security credentials<a name="configure-access-how"></a>

How you configure security credentials depends on how you or your organization manages users\. You will primarily use the AWS CLI to configure and manage security credentials for the CDK CLI\. However, you can use alternative methods such as manually updating your `config` and `credentials` files, or setting environment variables\.

For guidance on configuring security credentials using the AWS CLI, along with information on configuration and credential precedence when using different methods, see [Authentication and access credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html) in the *AWS Command Line Interface User Guide*\. If you prefer to quickly configure basic settings, including security credentials, see [Set up the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) in the *AWS Command Line Interface User Guide*\.

Once you’ve configured security credentials on your local machine, you can use the CDK CLI to interact with AWS\. For more information on using the CDK CLI with your method of managing users, see the following:
+ [Configure and manage CDK CLI security credentials for IAM Identity Center users](configure-access-sso.md)\.

## Additional information<a name="configure-access-info"></a>

To learn about the different ways that you can sign in to AWS, depending on the type of user you are, see [What is AWS Sign\-In?](https://docs.aws.amazon.com/signin/latest/userguide/what-is-sign-in.html) in the *AWS Sign\-In User Guide*\.

For reference information when using AWS SDKs and tools, including the AWS CLI, see the [AWS SDKs and Tools Reference Guide](https://docs.aws.amazon.com/sdkref/latest/guide/overview.html)\.