# Getting started with the AWS CDK<a name="getting_started"></a>

This topic introduces you to important AWS CDK concepts and describes how to install and configure the AWS CDK\. When you're done, you'll be ready to create [your first AWS CDK app](hello_world.md)\.

## Your background<a name="getting_started_background"></a>

The AWS Cloud Development Kit \(AWS CDK\) lets you define your cloud infrastructure as code in one of five supported programming languages\. It is intended for moderately to highly experienced AWS users\.

Ideally, you already have experience with popular AWS services, particularly [AWS Identity and Access Management](https://aws.amazon.com/iam/) \(IAM\)\. You might already have AWS credentials on your workstation for use with an AWS SDK or the AWS CLI and experience working with AWS resources programmatically\.

Familiarity with [AWS CloudFormation](https://aws.amazon.com/cloudformation/) is also useful, as the output of an AWS CDK program is a AWS CloudFormation template\.

Finally, you should be proficient in the programming language you intend to use with the AWS CDK\. 

## Key concepts<a name="getting_started_concepts"></a>

The AWS CDK is designed around a handful of important concepts\. We will introduce a few of these here briefly\. Follow the links to learn more, or see the Concepts topics in this guide's Table of Contents\.

An AWS CDK [app](apps.md) is an application written in TypeScript, JavaScript, Python, Java, or C\# that uses the AWS CDK to define AWS infrastructure\. An app defines one or more [stacks](stacks.md)\. Stacks \(equivalent to AWS CloudFormation stacks\) contain [constructs](constructs.md), each of which defines one or more concrete AWS resources, such as Amazon S3 buckets, Lambda functions, Amazon DynamoDB tables, and so on\.

Constructs \(as well as stacks and apps\) are represented as types in your programming language of choice\. You instantiate constructs within a stack to declare them to AWS, and connect them to each other using well\-defined interfaces\.

The AWS CDK includes the AWS CDK Toolkit \(also called the CLI\), a command\-line tool for working with your AWS CDK apps and stacks\. Among other functions, the Toolkit provides the ability to convert one or more AWS CDK stacks to AWS CloudFormation templates and related assets \(a process called *synthesis*\) and to deploy your stacks to an AWS account\.

The AWS CDK includes a library of AWS constructs called the AWS Construct Library\. Each AWS service has at least one corresponding module in the library containing the constructs that represent that service's resources\.

Constructs come in three fundamental flavors:
+ **AWS CloudFormation\-only** or L1 \(short for "level 1"\)\. These constructs correspond directly to resource types defined by AWS CloudFormation\. In fact, these constructs are automatically generated from the AWS CloudFormation specification, so when a new AWS service is launched, the AWS CDK supports it as soon as AWS CloudFormation does\.

  AWS CloudFormation resources always have names that begin with `Cfn`\. For example, in the Amazon S3 module, `CfnBucket` is the L1 module for an Amazon S3 bucket\.
+ **Curated** or L2\. These constructs are carefully developed by the AWS CDK team to address specific use cases and simplify infrastructure development\. For the most part, they encapsulate L1 modules, providing sensible defaults and best\-practice security policies\. For example, in the Amazon S3 module, `Bucket` is the L2 module for an Amazon S3 bucket\. 

  L2 modules may also define supporting resources needed by the primary resource\. Some services have more than one L2 module in the Construct Library for organizational purposes\. 
+ **Patterns** or L3\. Patterns declare multiple resources to create entire AWS architectures for particular use cases\. All the plumbing is already hooked up, and configuration is boiled down to a few important parameters\. In the AWS Construct Library, patterns are in separate modules from L1 and L2 constructs\.

The AWS CDK's core module \(usually imported into code as `core` or `cdk`\) contains constructs used by the AWS CDK itself as well as base classes for constructs, apps, resources, and other AWS CDK objects\.

## Supported programming languages<a name="getting_started_languages"></a>

The AWS CDK has first\-class support for TypeScript, JavaScript, Python, Java, and C\#\. \(Other JVM and \.NET CLR languages may also be used, at least in theory, but we are unable to offer support for them at this time\.\)

To facilitate supporting so many languages, the AWS CDK is developed in one language \(TypeScript\) and language bindings are generated for the other languages through the use of a tool called [JSII](https://github.com/aws/jsii)\.

We have taken pains to make AWS CDK app development in each language follow that language's usual conventions, so writing AWS CDK apps feels natural, not like writing TypeScript in Python \(for example\)\. Take a look:

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket',
  versioned: true,
  websiteRedirect: {host: 'aws.amazon.com'}});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket',
  versioned: true,
  websiteRedirect: {host: 'aws.amazon.com'}});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "MyBucket", bucket_name="my-bucket", versioned=true,
            website_redirect=s3.WebsiteRedirect(host_name="aws.amazon.com"))
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(self, "MyBucket")
                      .bucketName("my-bucket")
                      .versioned(true)
                      .websiteRedirect(new websiteRedirect.Builder()
                          .hostName("aws.amazon.com").build())
                      .build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "MyBucket", new BucketProps {
                      BucketName = "my-bucket",
                      Versioned  = true,
                      WebsiteRedirect = new WebsiteRedirect {
                              HostName = "aws.amazon.com"
                      }});
```

------

**Note**  
These code snippets are intended for illustration only\. They are incomplete and won't run as they are\.

The AWS Construct Library is distributed using each language's standard package management tools, including NPM, PyPi, Maven, and NuGet\. There's even a version of the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) for each language\.

To help you use the AWS CDK in your favorite language, this Guide includes topics that explain how to use the AWS CDK in all supported languages\.
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)

Furthermore, since TypeScript was the first language supported by the AWS CDK, much AWS CDK example code is written in TypeScript\. For this reason, this Guide also includes a topic specifically to show how to adapt TypeScript AWS CDK code for use with the other supported languages\. See [Translating TypeScript AWS CDK code to other languages](multiple_languages.md)\.

## Prerequisites<a name="getting_started_prerequisites"></a>

With the concepts out of the way, here's what you need to have on your workstation before you install the AWS CDK and start developing\.

All CDK developers need to [install Node\.js](https://nodejs.org/en/download/) 10\.3\.0 or later, even those working in languages other than TypeScript or JavaScript\. The AWS CDK Toolkit \(cdk command\-line tool\) and the AWS Construct Library are developed in TypeScript and run on Node\.js\. The bindings for other supported languages use this back end and tool set\. We suggest the latest LTS version\.

**Important**  
Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK\.

You must provide your credentials and an AWS Region to use AWS CDK, if you have not already done so\.

**Important**  
We strongly recommend against using your AWS root account for day\-to\-day tasks\. Instead, create a user in IAM and use its credentials with the CDK\. Best practices are to change this account's access key regularly and to use a least\-privileges role \(specifying `--role-arn`\) when deploying\.

If you have the AWS CLI installed, the easiest way to satisfy this requirement is to install the AWS CLI and issue the following command:

```
aws configure
```

Provide your AWS access key ID, secret access key, and default region when prompted\.

You may also manually create or edit the `~/.aws/config` and `~/.aws/credentials` \(Mac OS X or Linux\) or `%USERPROFILE%\.aws\config` and `%USERPROFILE%\.aws\credentials` \(Windows\) files to contain credentials and a default region, in the following format\.
+ In `~/.aws/config` or `%USERPROFILE%\.aws\config`

  ```
  [default]
  region=us-west-2
  ```
+ In `~/.aws/credentials` or `%USERPROFILE%\.aws\credentials`

  ```
  [default]
  aws_access_key_id=AKIAI44QH8DHBEXAMPLE
  aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
  ```

Finally, you can set the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` to appropriate values\. 

Other prerequisites depend on your development language and are as follows\.

------
#### [ TypeScript ]
+ TypeScript 2\.7 or later \(`npm -g install typescript`\)

------
#### [ JavaScript ]

No additional requirements

------
#### [ Python ]
+ Python 3\.6 or later including `pip` and `virtualenv`

------
#### [ Java ]
+ Java Development Kit \(JDK\) 8 \(a\.k\.a\. 1\.8\) or later
+ Apache Maven 3\.5 or later

Java IDE recommended \(we use Eclipse in some examples in this Developer Guide\)\. IDE must be able to import Maven projects\. Check to make sure your project is set to use Java 1\.8\. Set the JAVA\_HOME environment variable to the path where you have installed the JDK\.

------
#### [ C\# ]

A \.NET Standard 2\.1\-compatible implementation is required, such as\.
+ \.NET Core 3\.1 or later
+ \.NET Framework 4\.6\.1 or later
+ Mono 5\.4 or later

Visual Studio 2019 \(any edition\) recommended\.

------

## Install the AWS CDK<a name="getting_started_install"></a>

Install the AWS CDK Toolkit globally using the following Node Package Manager command\.

```
npm install -g aws-cdk
```

Run the following command to verify correct installation and print the version number of the AWS CDK\.

```
cdk --version
```

## AWS CDK tools<a name="getting_started_tools"></a>

We've already been using the AWS CDK Toolkit, also known as the Command Line Interface \(CLI\)\. It's the main tool you use to interact with your AWS CDK app\. It executes the AWS CDK app you wrote and compiled, interrogates the application model you defined, and produces and deploys the AWS CDK templates it generates\. It also has deployment, diff, deletion, and troubleshooting capabilities\. For more information, see cdk \-\-help or [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.

The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open\-source plug\-in for Visual Studio Code that makes it easier to create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications, including the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. [Install the plug\-in](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html) and learn more about [using the AWS CDK Explorer](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Next steps<a name="getting_started_next_steps"></a>

Where do you go now that you've dipped your toes in the AWS CDK?
+ Come on in; the water's fine\! Build [your first AWS CDK app](hello_world.md)\.
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) to begin exploring the CDK constructs available for your favorite AWS services\.
+ Dig deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Bootstrapping](bootstrapping.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Escape hatches](cfn_layer.md)\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples) of using the AWS CDK\.

The AWS CDK is an open\-source project\. Want to [contribute](https://github.com/aws/aws-cdk)?