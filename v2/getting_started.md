# Getting started with the AWS CDK<a name="getting_started"></a>

Get started with the AWS Cloud Development Kit \(AWS CDK\) by installing the AWS CDK CLI and creating your first CDK app\.

**Topics**
+ [Prerequisites](#getting_started_prerequisites)
+ [Step 1: Create an AWS account](#getting_started_account)
+ [Step 2: Configure programmatic access](#getting_started_auth)
+ [Step 3: Install the AWS CDK CLI](#getting_started_install)
+ [Step 4: Bootstrap your environment](#getting_started_bootstrap)
+ [Optional AWS CDK tools](#getting_started_tools)
+ [Next steps](#getting_started_next_steps)
+ [Learn more](#getting_started_learn_more)
+ [Your first AWS CDK app](hello_world.md)

## Prerequisites<a name="getting_started_prerequisites"></a>

**Recommended resources**  
Before getting started with the AWS CDK, we recommend a basic understanding of the following:  
+ An introduction to the AWS CDK\. To learn more, see [What is the AWS CDK?](home.md)
+ Core concepts behind the AWS CDK\. To learn more, see [AWS CDK concepts](core_concepts.md)\.
+ The AWS services that you want to manage with the AWS CDK\.
+ AWS Identity and Access Management\. For more information, see [What is IAM?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html) and [What is IAM Identity Center?](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
+ AWS CloudFormation since the AWS CDK utilizes the AWS CloudFormation service to provision resources created in the CDK\. To learn more, see [What is AWS CloudFormation?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
+ The supported programming language that you plan to use with the AWS CDK\.

**Prepare your local environment**  
All AWS CDK developers, regardless of your preferred language, need [https://nodejs.org/en/download/](https://nodejs.org/en/download/) 14\.15\.0 or later\. All supported programming languages use the same backend, which runs on Node\.js\. We recommend a version in [active long\-term support](https://nodejs.org/en/about/releases/)\. Your organization may have a different recommendation\.   
Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK due to compatibility issues with its dependencies\.
Other prerequisites depend on the language in which you develop AWS CDK applications and are as follows\.  
+ TypeScript 3\.8 or later \(`npm -g install typescript`\)
No additional requirements
+ Python 3\.7 or later including `pip` and `virtualenv`
+ Java Development Kit \(JDK\) 8 \(a\.k\.a\. 1\.8\) or later
+ Apache Maven 3\.5 or later
Java IDE recommended \(we use Eclipse in some examples in this guide\)\. IDE must be able to import Maven projects\. Check to make sure that your project is set to use Java 1\.8\. Set the JAVA\_HOME environment variable to the path where you have installed the JDK\.
\.NET Core 3\.1 or later, or \.NET 6\.0 or later\.  
Visual Studio 2019 \(any edition\) or Visual Studio Code recommended\.
Go 1\.1\.8 or later\.
For more detailed information, see the *Prerequisites* section for your language:  
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)
+ [Working with the AWS CDK in Go](work-with-cdk-go.md)
**Third\-party language deprecation**  
Each language version is only supported until it is EOL \(End Of Life\) and is subject to change with prior notice\.

## Step 1: Create an AWS account<a name="getting_started_account"></a>

If you are new to AWS, you must sign up for an AWS account and create an administrative user\. For more information, see [Getting set up with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-set-up.html) in the *IAM User Guide*\.

When you interact with AWS, you specify your AWS security credentials to verify who you are and whether you have permission to access the resources that you are requesting\. AWS uses the security credentials to authenticate and authorize your requests\. To learn more, see [AWS security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/security-creds.html) in the *IAM User Guide*\.

## Step 2: Configure programmatic access<a name="getting_started_auth"></a>

When developing with the AWS CDK in your local environment, you will rely on the AWS CDK CLI to interact with AWS services and manage your AWS resources\. To use the AWS CDK CLI, you must configure programmatic access\. To learn more about the different ways that you can configure programmatic access, see [Authentication and access](https://docs.aws.amazon.com/sdkref/latest/guide/access.html) in the *AWS SDKs and Tools Reference Guide*\.

For new users who aren’t given a method of authentication by their employer, we recommend using AWS IAM Identity Center\. This method includes installing the AWS Command Line Interface \(AWS CLI\) and using it for configuration and signing in to the AWS access portal\. To configure programmatic access using IAM Identity Center, see [IAM Identity Center authentication](https://docs.aws.amazon.com/sdkref/latest/guide/access-sso.html) in the *AWS SDKs and Tools Reference Guide*\. After completion, your environment should contain the following elements:
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
If you already have an active AWS access portal session and run `aws sso login`, you won't be required to provide credentials\.   
The sign in process may prompt you to allow the AWS CLI access to your data\. Since the AWS CLI is built on top of the SDK for Python, permission messages may contain variations of the `botocore` name\.

## Step 3: Install the AWS CDK CLI<a name="getting_started_install"></a>

Install the AWS CDK CLI globally using the following Node Package Manager command\.

```
npm install -g aws-cdk
```

**Note**  
If you get a permission error, and have administrator access on your system, try `sudo npm install -g aws-cdk`\.

Run the following command to verify a successful installation\. The AWS CDK CLI should output the version number:

```
cdk --version
```

If you receive an error message, try uninstalling the AWS CDK CLI by running the following:

```
npm uninstall -g aws-cdk
```

Then, repeat steps to reinstall the AWS CDK CLI\.

If you still receive an error, delete the `node-modules` folder from the current project and also from the global `node-modules` folder\. To locate this folder, run `npm config get prefix`\.

The AWS CDK CLI will obtain security credentials from sources that you configured in previous steps\.

**Note**  
CDK Toolkit v2 works with existing CDK v1 projects\. However, it can't initialize new CDK v1 projects\. See [New prerequisites](migrating-v2.md#migrating-v2-prerequisites) if you need to be able to do that\.

## Step 4: Bootstrap your environment<a name="getting_started_bootstrap"></a>

Each AWS [environment](environments.md) that you plan to deploy resources to must be [bootstrapped](bootstrapping.md)\.

To bootstrap, run the following:

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

**Tip**  
If you don't have your AWS account number handy, you can get it from the AWS Management Console\. Or, if you have the AWS CLI installed, the following command displays your default account information, including the account number\.  

```
aws sts get-caller-identity
```
If you created named profiles in your local AWS configuration, you can use the `--profile` option to display the account information for a specific profile\. The following example shows how to display account information for the *prod* profile\.  

```
aws sts get-caller-identity --profile prod
```
To display the default Region, use `aws configure get`\.  

```
aws configure get region
aws configure get region --profile prod
```

## Optional AWS CDK tools<a name="getting_started_tools"></a>

The [AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/) is an open source plug\-in for Visual Studio Code that helps you create, debug, and deploy applications on AWS\. The toolkit provides an integrated experience for developing AWS CDK applications\. It includes the AWS CDK Explorer feature to list your AWS CDK projects and browse the various components of the CDK application\. [Install the plug\-in](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/setup-toolkit.html) and learn more about [using the AWS CDK Explorer](https://docs.aws.amazon.com/toolkit-for-vscode/latest/userguide/cdk-explorer.html)\.

## Next steps<a name="getting_started_next_steps"></a>

Now that you've installed the AWS CDK CLI, use it to build [your first AWS CDK app](hello_world.md)\.

To learn more about using the AWS CDK in your preferred programming language, see [Working with the AWS CDK in supported programming languages](work-with.md)\.

The AWS CDK is an open\-source project\. To contribute, see [Contributing to the AWS Cloud Development Kit \(AWS CDK\)](https://github.com/aws/aws-cdk/blob/main/CONTRIBUTING.md)\.

## Learn more<a name="getting_started_learn_more"></a>

To learn more about the AWS CDK, see the following:
+ **[CDK Workshop](https://cdkworkshop.com/)** – In\-depth hands\-on workshop\.
+ **[API reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html)** – Explore constructs available for the AWS services that you will use\.
+ **[Construct Hub](https://constructs.dev/search?q=&cdk=aws-cdk&cdkver=2&sort=downloadsDesc&offset=0)** – Find constructs from the CDK community\.
+ **[AWS CDK examples](https://github.com/aws-samples/aws-cdk-examples)** – Explore code examples of AWS CDK projects\.