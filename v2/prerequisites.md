# AWS CDK prerequisites<a name="prerequisites"></a>

Complete all prerequisites before getting started with the AWS Cloud Development Kit \(AWS CDK\)\.

## Set up your AWS account<a name="prerequisites-account"></a>

If you or your organization are new to AWS, you must set up your AWS account\. This includes signing up for an AWS account, securing your root user, determining your method of managing users, and creating an administrative user\. To manage users, you can use AWS Identity and Access Management \(IAM\) or AWS IAM Identity Center\. We recommend that you use IAM Identity Center\. For more information, see the following:
+ [What is IAM?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) in the *IAM User Guide*\.
+ [What is IAM Identity Center?](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) in the *AWS IAM Identity Center User Guide*\.

After setting up an AWS account, you should have an administrative user and the ability to create and manage additional users using IAM or IAM Identity Center\.

Before moving forward, we recommend that you take time to learn the recommended best practices in AWS Identity and Access Management\. For more information, see [Security best practices and use cases in AWS Identity and Access Management](https://docs.aws.amazon.com/IAM/latest/UserGuide/IAMBestPracticesAndUseCases.html) in the *IAM User Guide*\.

## Install and configure the AWS CLI<a name="prerequisites-cli"></a>

When you develop AWS CDK applications on your local machine, you will use the AWS Cloud Development Kit \(AWS CDK\) Command Line Interface \(CLI\) to interact with AWS, such as deploying applications to provision your AWS resources\. To interact with AWS outside of the AWS Management Console, you must configure security credentials on your local machine\. To do this, we recommend that you install and use the AWS Command Line Interface \(AWS CLI\)\.

For instructions on installing the AWS CLI, see [Install or update to the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) in the *AWS Command Line Interface User Guide*\.

How you configure security credentials will depend on how you or your organization manages users\. For instructions, see [Authentication and access credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-authentication.html) in the *AWS Command Line Interface User Guide*\.

After installing and configuring the AWS CLI, you should have the following:
+ The AWS CLI installed on your local machine\.
+ Credentials configured in a `config` on your local machine using the AWS CLI\.

## Install Node\.js and programming language prerequisites<a name="prerequisites-node"></a>

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

**Third\-party language deprecation**  
Each language version is only supported until it is EOL \(End Of Life\) and is subject to change with prior notice\.

## Next steps<a name="prerequisites-next"></a>

To get started with the AWS CDK, see [Getting started with the AWS CDK](getting_started.md)\.