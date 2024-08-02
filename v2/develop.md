# Develop AWS CDK applications<a name="develop"></a>

Manage your infrastructure on AWS by developing applications using the AWS Cloud Development Kit \(AWS CDK\)\.

## Prerequisites<a name="develop-prerequisites"></a>

Before you can start developing applications, complete all set up steps in [Getting started with the AWS CDK](getting_started.md)\.

## Developing AWS CDK applications overview<a name="develop-overview"></a>

At a high\-level, developing CDK applications involves the following steps:

1. **Create a CDK project** – A CDK [project](projects.md) consists of the files and folders that store and organize your CDK code\.

1. **Create a CDK app** – Within a CDK project, you use the `App` construct to define a CDK [application](apps.md)\.

1. **Create a CDK stack** – Within the scope of your CDK app, you define one or more CDK [stacks](stacks.md)\.

1. **Define your infrastructure** – Within the scope of a CDK stack, you use [constructs](constructs.md) from the AWS Construct Library to define the AWS resources and properties that will become your infrastructure\. Using a general\-purpose programming [language](languages.md) of your choice, you can use logic, such as conditional statements and loops, to define your infrastructure based on certain conditions\.

## Get started with developing CDK applications<a name="develop-gs"></a>

To get started, you can use the AWS CDK Command Line Interface \(AWS CDK CLI\) `cdk init` command\. Provide the `--language` option to specify the programming language to use\. This command creates a starting CDK project and imports the main AWS Construct Library and core modules\.

## Import and use the AWS CDK Library<a name="develop-library"></a>

After you create a CDK project, import and use constructs from the AWS CDK Library to begin defining your infrastructure\. For instructions, see [Work with the AWS CDK library](work-with.md)\.

## Next steps<a name="develop-next"></a>

When ready to deploy your application, use the CDK CLI `cdk deploy` command\. For instructions, see [Deploy AWS CDK applications](deploy.md)\.