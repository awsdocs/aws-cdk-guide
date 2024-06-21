# Getting started with the AWS CDK<a name="getting_started"></a>

Get started with the AWS Cloud Development Kit \(AWS CDK\) by creating an AWS account, configuring the AWS CDK Command Line Interface \(AWS CDK CLI\) and creating your first CDK app\.

## Prerequisites<a name="getting_started_prerequisites"></a>

Before getting started with the AWS CDK, we recommend that you have a basic understanding of what the AWS CDK is\. For more information, see [What is the AWS CDK?](home.md) and [Learn AWS CDK core concepts](core_concepts.md)\.

## Step 1: Create an AWS account and administrative user<a name="getting_started_account"></a>

If you or your organization are new to AWS, you must sign up for an AWS account and create an administrative user\. For instructions, see [Getting set up with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-set-up.html) in the *IAM User Guide*\.

You can manage IAM using different methods, such as through the AWS console, the AWS Command Line Interface \(AWS CLI\), or through application interfaces \(APIs\) in the associated SDKs\. When using IAM with the AWS CDK CLI, you will primarily use the AWS CLI to configure and manage security credentials\. To learn more, see [AWS Command Line Interface \(CLI\) and Software Development Kits \(SDKs\)](https://docs.aws.amazon.com/IAM/latest/UserGuide/management_methods.html#management-method-cli-sdk) in the *IAM User Guide*\.

Once you’ve created an administrative user, you can start using the AWS CDK by installing the CDK CLI\. However, we recommend that you first determine your method of managing users, adhering to the IAM best practice of [applying least\-privilege permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)\. Rather than using your administrative user for everything, you create IAM users and grant only the permissions required to perform a task\.

## Step 2: Determine your method of managing users<a name="getting_started_users"></a>

Once you’ve created your AWS account and administrative user, you’ll want to determine your method of managing users\. To learn about the different methods of managing users, see [Overview of AWS identity management: Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction_identity-management.html) in the *IAM User Guide*\.

If your organization has a method of managing users, follow their guidance\. Otherwise, we recommend using AWS IAM Identity Center to create and manage users\. With IAM Identity Center, you can manage AWS accounts, users, and permissions from a centrally managed service\. You can also use an identity provider to authenticate users and provide temporary credentials for use with the CDK CLI\. This is an [IAM security best practice](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#bp-users-federation-idp)\. For an introduction to IAM Identity Center, see [What is IAM Identity Center?](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) in the *AWS IAM Identity Center User Guide*\.

As a user, you need to configure security credentials in your local development environment for the CDK CLI\. We recommend that you use install and use the AWS Command Line Interface \(AWS CLI\) to accomplish this\.

## Step 3: Install the AWS CLI<a name="getting_started_cli"></a>

As a user, you use the AWS CLI to create and manage configuration and credential files on your local machine\. These files are used to store, manage, and generate security credentials for use with the CDK CLI\.

To install the AWS CLI, see [Install or update to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) in the *AWS Command Line Interface User Guide*\.

To learn more about these files, see [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) in the *AWS Command Line Interface User Guide*\.

After installing the AWS CLI, you will configure security credentials in a later step\.

## Step 4: Install Node\.js and programming language prerequisites<a name="getting_started_node"></a>

All AWS CDK developers, regardless of the supported programming language that you will use, require [https://nodejs.org/en/download/](https://nodejs.org/en/download/) 14\.15\.0 or later\. All supported programming languages use the same backend, which runs on Node\.js\. We recommend a version in [active long\-term support](https://nodejs.org/en/about/releases/)\. If your organization has a different recommendation, follow their guidance\.

**Important**  
Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK due to compatibility issues with its dependencies\.

Other programming language prerequisites depend on the language that you will use to develop AWS CDK applications:

------
#### [ TypeScript ]
+ TypeScript 3\.8 or later \(`npm -g install typescript`\)

------
#### [ JavaScript ]

No additional requirements

------
#### [ Python ]
+ Python 3\.7 or later including `pip` and `virtualenv`

------
#### [ Java ]
+ Java Development Kit \(JDK\) 8 \(a\.k\.a\. 1\.8\) or later
+ Apache Maven 3\.5 or later

Java IDE recommended \(we use Eclipse in some examples in this guide\)\. IDE must be able to import Maven projects\. Check to make sure that your project is set to use Java 1\.8\. Set the JAVA\_HOME environment variable to the path where you have installed the JDK\.

------
#### [ C\# ]

\.NET Core 3\.1 or later, or \.NET 6\.0 or later\.

Visual Studio 2019 \(any edition\) or Visual Studio Code recommended\.

------
#### [ Go ]

Go 1\.1\.8 or later\.

------

For more detailed information, see the *Prerequisites* section for your language:
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)
+ [Working with the AWS CDK in Go](work-with-cdk-go.md)

**Third\-party language deprecation**  
Each language version is only supported until it is EOL \(End Of Life\) and is subject to change with prior notice\.

## Step 5: Install the AWS CDK CLI<a name="getting_started_install"></a>

Use the Node Package Manager to install the CDK CLI\. We recommend that you install it globally using the following command:

```
$ npm install -g aws-cdk
```

To install a specific version of the CDK CLI, use the following command structure:

```
$ npm install -g aws-cdk@X.YY.Z
```

If you want to use multiple versions of the AWS CDK, consider installing a matching version of the CDK CLI in individual CDK projects\. To do this, remove the `-g` option from the `npm install` command\. Then, use `npx aws-cdk` to invoke the CDK CLI\. This will run a local version if it exists\. Otherwise, the globally installed version will be used\.

**Note**  
If you get a permission error, and have administrator access on your system, try `sudo npm install -g aws-cdk`\.

Run the following command to verify a successful installation\. The AWS CDK CLI should output the version number:

```
$ cdk --version
```

If you receive an error message, try uninstalling the AWS CDK CLI by running the following:

```
$ npm uninstall -g aws-cdk
```

Then, repeat steps to reinstall the AWS CDK CLI\.

## Step 6: Configure security credentials for the CDK CLI<a name="getting_started_creds"></a>

When you develop with the AWS CDK in your local environment, you will primarily use the CDK CLI to interact with AWS\. These interactions include deploying CDK stacks, performing stack diffs, importing resources into the CDK, and more\.

To perform these actions, you must configure security credentials for the CDK CLI on your local machine\. This lets AWS know who you are and what permissions you have\. For guidance, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\.

## Step 7: Bootstrap your AWS environment<a name="getting_started_bootstrap"></a>

CDK stacks are deployed into AWS [environments](environments.md)\. Before you can deploy a CDK stack into an environment, the environment must first be [bootstrapped](bootstrapping.md)\.

To bootstrap your environment, use the CDK CLI `cdk bootstrap` command\. For instructions, see [How to bootstrap your environment](bootstrapping-env.md#bootstrapping-howto)\.

## Step 8: \(Optional\) Install additional AWS CDK tools<a name="getting_started_tools"></a>

The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open source plug\-in for Visual Studio Code that helps you create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications\. It includes the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. For instructions, see the following:
+ [Installing the AWS Toolkit for Visual Studio Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html)\.
+ [AWS CDK for VS Code](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Step 9: Create your first CDK app<a name="getting_started_app"></a>

You're now ready to get started with using the AWS CDK by creating your first CDK app\. For instructions, see [Tutorial: Create your first AWS CDK app](hello_world.md)\.