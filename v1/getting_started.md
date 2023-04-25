# Getting started with the AWS CDK<a name="getting_started"></a>

This topic introduces you to important AWS CDK concepts and describes how to install and configure the AWS CDK\. When you're done, you'll be ready to create [your first AWS CDK app](hello_world.md)\.

## Your background<a name="getting_started_background"></a>

The AWS Cloud Development Kit \(AWS CDK\) lets you define your cloud infrastructure as code in one of its supported programming languages\. It is intended for moderately to highly experienced AWS users\.

Ideally, you already have experience with popular AWS services, particularly [AWS Identity and Access Management](http://aws.amazon.com/iam/) \(IAM\)\. You might already have AWS credentials on your workstation for use with an AWS SDK or the AWS CLI and experience working with AWS resources programmatically\.

Familiarity with [AWS CloudFormation](http://aws.amazon.com/cloudformation/) is also useful, as the output of an AWS CDK program is an AWS CloudFormation template\.

Finally, you should be proficient in the programming language you intend to use with the AWS CDK\. 

## Key concepts<a name="getting_started_concepts"></a>

The AWS CDK is designed around a handful of important concepts\. We will introduce a few of these here briefly\. Follow the links to learn more, or see the Concepts topics in this guide's Table of Contents\.

An AWS CDK [app](apps.md) is an application written in TypeScript, JavaScript, Python, Java, C\#, or Go that uses the AWS CDK to define AWS infrastructure\. An app defines one or more [stacks](stacks.md)\. Stacks \(equivalent to AWS CloudFormation stacks\) contain [constructs](constructs.md), each of which defines one or more concrete AWS resources, such as Amazon S3 buckets, Lambda functions, Amazon DynamoDB tables, and so on\.

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

The AWS CDK has first\-class support for TypeScript, JavaScript, Python, Java, C\#, and Go\. Other JVM and \.NET CLR languages may also be used, at least in theory, but we are unable to offer support for them at this time\.

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
bucket = s3.Bucket("MyBucket", bucket_name="my-bucket", versioned=True,
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

The AWS Construct Library is distributed using each language's standard package management tools, including NPM, PyPi, Maven, and NuGet\. There's even a version of the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v1/docs/aws-construct-library.html) for each language\.

To help you use the AWS CDK in your favorite language, this Guide includes topics that explain how to use the AWS CDK in all supported languages\.
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)

TypeScript was the first language supported by the AWS CDK, and much AWS CDK example code is written in TypeScript\. This Guide includes a topic specifically to show how to adapt TypeScript AWS CDK code for use with the other supported languages\. See [Translating TypeScript AWS CDK code to other languages](multiple_languages.md)\.

## Prerequisites<a name="getting_started_prerequisites"></a>

Here's what you need to install to use the AWS CDK\.

All AWS CDK developers, even those working in Python, Java, C\#, or Go, need [Node\.js](https://nodejs.org/en/download/) 10\.13\.0 or later\. All supported languages use the same back end, which runs on Node\.js\. We recommend a version in [active long\-term support](https://nodejs.org/en/about/releases/), which, at this writing, is the latest 16\.x release\. Your organization may have a different recommendation\.

**Important**  
Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK due to compatibility issues with its dependencies\.

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

**Note**  
Third\-party Language Deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

## Authentication with AWS<a name="getting_started_auth"></a>

You must establish how the AWS CDK authenticates with AWS when developing with AWS services\. There are different ways in which you can configure programmatic access to AWS resources, depending on the environment and the AWS access available to you\. 

To choose your method of authentication and configure it for the AWS CDK, see [Authentication and access](https://docs.aws.amazon.com/sdkref/latest/guide/access.html) in the *AWS SDKs and Tools Reference Guide*\. 

The recommended approach for new users developing locally, who are not given a method of authentication by their employer, is to set up AWS IAM Identity Center \(successor to AWS Single Sign\-On\)\. This method includes installing the AWS CLI for ease of configuration and for regularly signing in to the AWS access portal\. If you choose this method, your environment should contain the following elements after you complete the procedure for [IAM Identity Center authentication](https://docs.aws.amazon.com/sdkref/latest/guide/access-sso.html) in the *AWS SDKs and Tools Reference Guide*:
+ The AWS CLI, which you use to start an AWS access portal session before you run your application\.
+ A [shared AWS`config` file](https://docs.aws.amazon.com/sdkref/latest/guide/file-format.html) having a `[default]` profile with a set of configuration values that can be referenced from the AWS CDK\. To find the location of this file, see [Location of the shared files](https://docs.aws.amazon.com/sdkref/latest/guide/file-location.html) in the *AWS SDKs and Tools Reference Guide*\.
+  The shared `config` file sets the [https://docs.aws.amazon.com/sdkref/latest/guide/feature-region.html](https://docs.aws.amazon.com/sdkref/latest/guide/feature-region.html) setting\. This sets the default AWS Region the AWS CDK uses for AWS requests\. 
+  The AWS CDK uses the profile's [SSO token provider configuration](https://docs.aws.amazon.com/sdkref/latest/guide/feature-sso-credentials.html#feature-sso-credentials-profile) to acquire credentials before sending requests to AWS\. The `sso_role_name` value, which is an IAM role connected to an IAM Identity Center permission set, should allow access to the AWS services used in your application\.

  The following sample `config` file shows a default profile set up with SSO token provider configuration\. The profile's `sso_session` setting refers to the named [`sso-session` section](https://docs.aws.amazon.com/sdkref/latest/guide/file-format.html#section-session)\. The `sso-session` section contains settings to initiate an AWS access portal session\.

  ```
  [default]
  sso_session = my-sso
  sso_account_id = 111122223333
  sso_role_name = SampleRole
  region = us-east-1
  output = json
  
  [sso-session my-sso]
  sso_region = us-east-1
  sso_start_url = https://provided-domain.awsapps.com/start
  sso_registration_scopes = sso:account:access
  ```

### Start an AWS access portal session<a name="accessportal"></a>

Before accessing AWS services, you need an active AWS access portal session for the AWS CDK to use IAM Identity Center authentication to resolve credentials\. Depending on your configured session lengths, your access will eventually expire and the AWS CDK will encounter an authentication error\. Run the following command in the AWS CLI to sign in to the AWS access portal\.

```
aws sso login
```

 If your SSO token provider configuration is using a named profile instead of the default profile, the command is `aws sso login --profile NAME`\. Also specify this profile when issuing cdk commands using the \-\-profile option or the `AWS_PROFILE` environment variable\.

To test if you already have an active session, run the following AWS CLI command\.

```
aws sts get-caller-identity
```

The response to this command should report the IAM Identity Center account and permission set configured in the shared `config` file\.

**Note**  
If you already have an active AWS access portal session and run `aws sso login`, you will not be required to provide credentials\.   
The sign in process may prompt you to allow the AWS CLI access to your data\. Since the AWS CLI is built on top of the SDK for Python, permission messages may contain variations of the `botocore` name\.

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
+ See the [API reference](https://docs.aws.amazon.com/cdk/api/v1/docs/aws-construct-library.html) to begin exploring the provided constructs available for your favorite AWS services\.
+ Visit the [Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=1&sort=downloadsDesc&offset=0) to find constructs from the CDK community as well as from AWS\.
+ Dig deeper into concepts like [Environments](environments.md), [Assets](assets.md), [Bootstrapping](bootstrapping.md), [Permissions](permissions.md), [Runtime context](context.md), [Parameters](parameters.md), and [Abstractions and escape hatches](cfn_layer.md)\.
+ Explore [Examples](https://github.com/aws-samples/aws-cdk-examples/tree/CDKv1) of using the AWS CDK\.

The AWS CDK is an open\-source project\. Want to [contribute](https://github.com/aws/aws-cdk)?