# AWS CDK concepts<a name="core_concepts"></a>

Learn core concepts behind the AWS Cloud Development Kit \(AWS CDK\)\.

An AWS CDK [app](apps.md) is an application written in TypeScript, JavaScript, Python, Java, C\# or Go that uses the AWS CDK to define AWS infrastructure\. An app defines one or more [stacks](stacks.md)\. Stacks \(equivalent to AWS CloudFormation stacks\) contain [constructs](constructs.md)\. Each construct defines one or more AWS resources, such as Amazon S3 buckets, Lambda functions, or Amazon DynamoDB tables\.

Constructs \(and also stacks and apps\) are represented as classes \(types\) in your programming language of choice\. You instantiate constructs within a stack to declare them to AWS, and connect them to each other using well\-defined interfaces\.

The AWS CDK includes the CDK Toolkit \(also called the CLI\), a command line tool for working with your AWS CDK apps and stacks\. Among other functions, the Toolkit provides the ability to do the following:
+ Convert one or more AWS CDK stacks to AWS CloudFormation templates and related assets \(a process called *synthesis*\)\.
+ Deploy your stacks to an AWS [environment](environments.md)\.

**Topics**
+ [Supported programming languages](languages.md)
+ [Constructs](constructs.md)
+ [Apps](apps.md)
+ [Stacks](stacks.md)
+ [Environments](environments.md)
+ [Bootstrapping](bootstrapping.md)
+ [Resources](resources.md)
+ [Identifiers](identifiers.md)
+ [Tokens](tokens.md)
+ [Parameters](parameters.md)
+ [Tagging](tagging.md)
+ [Assets](assets.md)
+ [Permissions](permissions.md)
+ [Runtime context](context.md)
+ [Feature flags](featureflags.md)
+ [Aspects](aspects.md)