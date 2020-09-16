# Working with the AWS CDK<a name="work-with"></a>

The AWS Cloud Development Kit \(AWS CDK\) lets you define your AWS cloud infrastructure in a general\-purpose programming language\. Currently, the AWS CDK supports TypeScript, JavaScript, Python, Java, and C\#\. It is also possible to use other JVM and \.NET languages, though we are unable to provide support for every such language\.

We develop the AWS CDK in TypeScript and use [JSII](https://github.com/aws/jsii) to provide a "native" experience in other supported languages\. For example, we distribute AWS Construct Library modules using your preferred language's standard repository, and you install them using the language's standard package manager\. Methods and properties are even named using your language's recommended naming patterns\.<a name="work-with-prerequisites"></a>

**AWS CDK prerequisites**  
To use the AWS CDK, you need an AWS account and a corresponding access key\. If you don't have an AWS account yet, see [Create and Activate an AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)\. To find out how to obtain an access key ID and secret access key for your AWS account, see [Understanding and Getting Your Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)\. To find out how to configure your workstation so the AWS CDK uses your credentials, see [Setting Credentials in Node\.js](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html)\.

**Tip**  
If you have the [AWS CLI](https://aws.amazon.com/cli/) installed, the simplest way to set up your workstation with your AWS credentials is to open a command prompt and type:  

```
aws configure
```

All AWS CDK applications require Node\.js 10\.3 or later, even when your app is written in Python, Java, or C\#\. You may download a compatible version for your platform at [nodejs\.org](https://nodejs.org/)\. We recommend the [current LTS version](https://nodejs.org/en/about/releases/) \(at this writing, the latest 12\.x release\)\.

After installing Node\.js, install the AWS CDK Toolkit \(the `cdk` command\):

```
npm install -g aws-cdk
```

**Note**  
If you get a permission error, and have administrator access on your system, try `sudo npm install -g aws-cdk`\.

Test the installation by issuing `cdk --version`\.

The specific language you work in also has its own prerequisites, described in the corresponding topic listed here\.

**Topics**
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)