# Getting started with the AWS CDK<a name="getting_started"></a>

Get started with the AWS Cloud Development Kit \(AWS CDK\) by installing and configuring the AWS CDK Command Line Interface \(AWS CDK CLI\)\. Then, use the CDK CLI to create your first CDK app, bootstrap your AWS environment, and deploy your application\.

## Prerequisites<a name="getting_started_prerequisites"></a>

Before getting started with the AWS CDK, complete all prerequisites\. These prerequisites are required for those that are new to AWS or new to programming\. For instructions, see [AWS CDK prerequisites](prerequisites.md)\.

We recommend that you have a basic understanding of what the AWS CDK is\. For more information, see [What is the AWS CDK?](home.md) and [Learn AWS CDK core concepts](core_concepts.md)\.

## Install the AWS CDK CLI<a name="getting_started_install"></a>

Use the Node Package Manager to install the CDK CLI\. We recommend that you install it globally using the following command:

```
$ npm install -g aws-cdk
```

To install a specific version of the CDK CLI, use the following command structure:

```
$ npm install -g aws-cdk@X.YY.Z
```

If you want to use multiple versions of the AWS CDK, consider installing a matching version of the CDK CLI in individual CDK projects\. To do this, remove the `-g` option from the `npm install` command\. Then, use `npx aws-cdk` to invoke the CDK CLI\. This will run a local version if it exists\. Otherwise, the globally installed version will be used\.

### Troubleshoot a CDK CLI installation<a name="getting_started_install_troubleshoot"></a>

If you get a permission error, and have administrator access on your system, run the following:

```
$ sudo npm install -g aws-cdk
```

If you receive an error message, try uninstalling the CDK CLI by running the following:

```
$ npm uninstall -g aws-cdk
```

Then, repeat steps to reinstall the CDK CLI\.

### Verify a successful CDK CLI installation<a name="getting_started_install_verify"></a>

Run the following command to verify a successful installation\. The AWS CDK CLI should output the version number:

```
$ cdk --version
```

## Configure the AWS CDK CLI<a name="getting_started_configure"></a>

After installing the CDK CLI, you can start using it to develop applications on your local machine\. To interact with AWS, such as deploying applications, you must have security credentials configured on your local machine with permissions to perform any actions that you initiate\.

To configure security credentials on your local machine, you use the AWS CLI\. How you configure security credentials depends on how you manage users\. For instructions, see [Authentication and access credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html) in the *AWS Command Line Interface User Guide*\.

The CDK CLI will automatically use the security credentials that you configure with the AWS CLI\. For example, if you are an IAM Identity Center user, you can use the `aws configure sso` command to configure security credentials\. If you are an IAM user, you can use the `aws configure` command\. The AWS CLI will guide you through configuring security credentials on your local machine and save the necessary information in your `config` and `credentials` files\. Then, when you use the CDK CLI, such as deploying an application with `cdk deploy`, the CDK CLI will use your configured security credentials\.

Just like the AWS CLI, the CDK CLI will use your `default` profile by default\. You can specify a profile using the CDK CLI `\-\-profile` option\. For more information on using security credentials with the CDK CLI, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\.

## \(Optional\) Install additional AWS CDK tools<a name="getting_started_tools"></a>

The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open source plug\-in for Visual Studio Code that helps you create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications\. It includes the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. For instructions, see the following:
+ [Installing the AWS Toolkit for Visual Studio Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html)\.
+ [AWS CDK for VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Create your first CDK app<a name="getting_started_app"></a>

You're now ready to get started with using the AWS CDK by creating your first CDK app\. For instructions, see [Tutorial: Create your first AWS CDK app](hello_world.md)\.