# Getting started with the AWS CDK<a name="getting_started"></a>

This topic introduces you to important AWS CDK concepts and describes how to install and configure the AWS CDK\. When you're done, you'll be ready to create [your first AWS CDK app](hello_world.md)\.

## Your background<a name="getting_started_background"></a>

The AWS Cloud Development Kit \(AWS CDK\) lets you define your cloud infrastructure as code in one of its supported programming languages\. It is intended for moderately to highly experienced AWS users\.

Ideally, you already have experience with popular AWS services, particularly [AWS Identity and Access Management](https://aws.amazon.com/iam/) \(IAM\)\. You might already have AWS credentials on your workstation for use with an AWS SDK or the AWS CLI and experience working with AWS resources programmatically\.

Familiarity with [AWS CloudFormation](https://aws.amazon.com/cloudformation/) is also useful, as the output of an AWS CDK program is an AWS CloudFormation template\.

Finally, you should be proficient in the programming language you intend to use with the AWS CDK\. 

## Key concepts<a name="getting_started_concepts"></a>

The AWS CDK is designed around a handful of important concepts\. We will introduce a few of these here briefly\. Follow the links to learn more, or see the Concepts topics in this guide's Table of Contents\.

An AWS CDK [app](apps.md) is an application written in TypeScript, JavaScript, Python, Java, or C\# that uses the AWS CDK to define AWS infrastructure\. An app defines one or more [stacks](stacks.md)\. Stacks \(equivalent to AWS CloudFormation stacks\) contain [constructs](constructs.md), each of which defines one or more concrete AWS resources, such as Amazon S3 buckets, Lambda functions, Amazon DynamoDB tables, and so on\.

**Note**  
The AWS CDK also supports Go in a developer preview\. This Guide does not include instructions or code examples for Go aside from [Working with the AWS CDK in Go](work-with-cdk-go.md)\.

Constructs \(as well as stacks and apps\) are represented as classes \(types\) in your programming language of choice\. You instantiate constructs within a stack to declare them to AWS, and connect them to each other using well\-defined interfaces\.

The AWS CDK includes the AWS CDK Toolkit \(also called the CLI\), a command\-line tool for working with your AWS CDK apps and stacks\. Among other functions, the Toolkit provides the ability to convert one or more AWS CDK stacks to AWS CloudFormation templates and related assets \(a process called *synthesis*\) and to deploy your stacks to an AWS account\.

The AWS CDK includes a library of AWS constructs called the AWS Construct Library\. Each AWS service has at least one corresponding module in the library containing the constructs that represent that service's resources\.

Constructs come in three fundamental flavors:
+ **AWS CloudFormation\-only** or L1 \(short for "layer 1"\)\. These constructs correspond directly to resource types defined by AWS CloudFormation\. In fact, these constructs are automatically generated from the AWS CloudFormation specification, so when a new AWS service is launched, the AWS CDK supports it a short time after AWS CloudFormation does\.

  AWS CloudFormation resources always have names that begin with `Cfn`\. For example, in the Amazon S3 module, `CfnBucket` is the L1 construct for an Amazon S3 bucket\.
+ **Curated** or L2\. These constructs are carefully developed by the AWS CDK team to address specific use cases and simplify infrastructure development\. For the most part, they encapsulate L1 resources, providing sensible defaults and best\-practice security policies\. For example, in the Amazon S3 module, `Bucket` is the L2 construct for an Amazon S3 bucket\. 

  Modules may also define supporting resources needed by the primary resource\. Some services have more than one L2 module in the Construct Library for organizational purposes\. 
+ **Patterns** or L3\. Patterns declare multiple resources to create entire AWS architectures for particular use cases\. All the plumbing is already hooked up, and configuration is boiled down to a few important parameters\. In the AWS Construct Library, patterns are in separate modules from L1 and L2 constructs\.

The AWS CDK's core module \(usually imported into code as `core` or `cdk`\) contains constructs used by the AWS CDK itself as well as base classes for apps, stacks, and other AWS CDK objects\.

Numerous third parties have also published constructs compatible with the AWS CDK\. Visit [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=1&offset=0) to explore the AWS CDK construct ecosystem\.

## Supported programming languages<a name="getting_started_languages"></a>

The AWS CDK has first\-class support for TypeScript, JavaScript, Python, Java, and C\#\. \(Other JVM and \.NET CLR languages may also be used, at least in theory, but we are unable to offer support for them at this time\.\) Go support is available as a Develper Preview\.

To facilitate supporting so many languages, the AWS CDK is developed in one language \(TypeScript\) and language bindings are generated for the other languages through the use of a tool called [JSII](https://github.com/aws/jsii)\.

We have taken pains to make AWS CDK app development in each language follow that language's usual conventions, so writing AWS CDK apps feels natural, not like writing TypeScript in Python \(for example\)\. Take a look:

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket',
  versioned: true,
  websiteRedirect: {hostName: 'aws.amazon.com'}});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket',
  versioned: true,
  websiteRedirect: {hostName: 'aws.amazon.com'}});
```

------
#### [ Python ]

```
bucket = s3.Bucket(self, "MyBucket", bucket_name="my-bucket", versioned=True,
            website_redirect=s3.RedirectTarget(host_name="aws.amazon.com"))
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(self, "MyBucket")
                      .bucketName("my-bucket")
                      .versioned(true)
                      .websiteRedirect(new RedirectTarget.Builder()
                          .hostName("aws.amazon.com").build())
                      .build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "MyBucket", new BucketProps {
                      BucketName = "my-bucket",
                      Versioned  = true,
                      WebsiteRedirect = new RedirectTarget {
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

TypeScript was the first language supported by the AWS CDK, and much AWS CDK example code is written in TypeScript\. This Guide includes a topic specifically to show how to adapt TypeScript AWS CDK code for use with the other supported languages\. See [Translating TypeScript AWS CDK code to other languages](multiple_languages.md)\.

## Prerequisites<a name="getting_started_prerequisites"></a>

Here's what you need to install to use the AWS CDK\.

All AWS CDK developers, even those working in Python, Java, or C\#, need [Node\.js](https://nodejs.org/en/download/) 10\.13\.0 or later\. All supported languages use the same back end, which runs on Node\.js\. We recommend a version in [active long\-term support](https://nodejs.org/en/about/releases/), which, at this writing, is the latest 16\.x release\. Your organization may have a different recommendation\.

**Important**  
Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK due to compatibility issues with its dependencies\.

You must configure your workstation with your credentials and an AWS region, if you have not already done so\. If you have the AWS CLI installed, the easiest way to satisfy this requirement is issue the following command:

```
aws configure
```

Provide your AWS access key ID, secret access key, and default region when prompted\.

You may also manually create or edit the `~/.aws/config` and `~/.aws/credentials` \(macOS/Linux\) or `%USERPROFILE%\.aws\config` and `%USERPROFILE%\.aws\credentials` \(Windows\) files to contain credentials and a default region, in the following format\.
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

**Note**  
Although the AWS CDK uses credentials from the same configuration files as other AWS tools and SDKs, including the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html), it may behave slightly differently from these tools\. In particular, if you use a named profile from the `credentials` file, the `config` must have a profile of the same name specifying the region\. The AWS CDK does not fall back to reading the region from the `[default]` section in `config`\. Also, do not use a profile named "default" \(e\.g\. `[profile default]`\)\. See [Setting credentials](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials.html) for complete details on setting up credentials for the AWS SDK for JavaScript, which the AWS CDK uses under the hood\.

Alternatively, you can set the environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` to appropriate values\. 

**Important**  
We strongly recommend against using your AWS root account for day\-to\-day tasks\. Instead, create a user in IAM and use its credentials with the CDK\. Best practices are to change this account's access key regularly and to use a least\-privileges role \(specifying `--role-arn`\) when deploying\.

Other prerequisites depend on the language in which you develop AWS CDK applications and are as follows\.

------
#### [ TypeScript ]
+ TypeScript 3\.8 or later \(`npm -g install typescript`\)

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

\.NET Core 3\.1 or later\.

Visual Studio 2019 \(any edition\) or Visual Studio Code recommended\.

------

## Install the AWS CDK<a name="getting_started_install"></a>

Install the AWS CDK Toolkit globally using the following Node Package Manager command\.

```
npm install -g aws-cdk@1.x
```

Run the following command to verify correct installation and print the version number of the AWS CDK\.

```
cdk --version
```

## Bootstrapping<a name="getting_started_bootstrap"></a>

Many AWS CDK stacks that you write will include [assets](assets.md): external files that are deployed with the stack, such as AWS Lambda functions or Docker images\. The AWS CDK uploads these to an Amazon S3 bucket or other container so they are available to AWS CloudFormation during deployment\. Deployment requires that these containers already exist in the account and region you are deploying into\. Creating them is called [bootstrapping](bootstrapping.md)\. To bootstrap, issue:

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

**Tip**  
If you don't have your AWS account number handy, you can get it from the AWS Management Console\. Or, if you have the AWS CLI installed, the following command displays your default account information, including the account number\.  

```
aws sts get-caller-identity
```
If you have created named profiles in your local AWS configuration, you can use the `--profile` option to display the account information for a specific profile's account, such as the *prod* profile as shown here\.  

```
aws sts get-caller-identity --profile prod
```
To display the default region, use `aws configure get`\.  

```
aws configure get region
aws configure get region --profile prod
```

## AWS CDK tools<a name="getting_started_tools"></a>

The AWS CDK Toolkit, also known as the Command Line Interface \(CLI\), is the main tool you use to interact with your AWS CDK app\. It executes your code and produces and deploys the AWS CloudFormation templates it generates\. It also has deployment, diff, deletion, and troubleshooting capabilities\. For more information, see cdk \-\-help or [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.

The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open\-source plug\-in for Visual Studio Code that makes it easier to create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications, including the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. [Install the plug\-in](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html) and learn more about [using the AWS CDK Explorer](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Next steps<a name="getting_started_next_steps"></a>

Where do you go now that you've dipped your toes in the AWS CDK?
+ Come on in; the water's fine\! Build [your first AWS CDK app](hello_world.md)\.
+ Try the [CDK Workshop](https://cdkworkshop.com/) for a more in\-depth tour involving a more complex project\.
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) to begin exploring the provided constructs available for your favorite AWS services\.
+ Visit the [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=1&sort=downloadsDesc&offset=0) to find constructs from the CDK community as well as from AWS\.
+ Dig deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Bootstrapping](bootstrapping.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Escape hatches](cfn_layer.md)\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples/tree/CDKv1) of using the AWS CDK\.

The AWS CDK is an open\-source project\. Want to [contribute](https://github.com/aws/aws-cdk)?