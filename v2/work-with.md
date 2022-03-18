# Working with the AWS CDK<a name="work-with"></a>

The AWS Cloud Development Kit \(CDK\) lets you define your AWS cloud infrastructure in a general\-purpose programming language\. Currently, the AWS CDK supports TypeScript, JavaScript, Python, Java, C\#, and \(in developer preview\) Go\. It is also possible to use other JVM and \.NET languages, though we are unable to provide support for every such language\.

**Note**  
This Guide does not currently include instructions or code examples for Go aside from [Working with the AWS CDK in Go](work-with-cdk-go.md)\. Go support is currently in developer preview\.

We develop the AWS CDK in TypeScript and use [JSII](https://github.com/aws/jsii) to provide a "native" experience in other supported languages\. For example, we distribute AWS Construct Library modules using your preferred language's standard repository, and you install them using the language's standard package manager\. Methods and properties are even named using your language's recommended naming patterns\.

## AWS CDK prerequisites<a name="work-with-prerequisites"></a>

To use the AWS CDK, you need an AWS account and a corresponding access key\. If you don't have an AWS account yet, see [Create and Activate an AWS Account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)\. To find out how to obtain an access key ID and secret access key for your AWS account, see [Understanding and Getting Your Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html)\. To find out how to configure your workstation so the AWS CDK uses your credentials, see [Setting Credentials in Node\.js](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html)\.

**Tip**  
If you have the [AWS CLI](https://aws.amazon.com/cli/) installed, the simplest way to set up your workstation with your AWS credentials is to open a command prompt and type:  

```
aws configure
```

All AWS CDK applications require Node\.js 10\.13 or later, even if you work in Python, Java, or C\#\. You may download a compatible version at [nodejs\.org](https://nodejs.org/)\. We recommend the [active LTS version](https://nodejs.org/en/about/releases/) \(at this writing, the latest 16\.x release\)\. Node\.js versions 13\.0\.0 through 13\.6\.0 are not compatible with the AWS CDK due to compatibility issues with its dependencies\.

After installing Node\.js, install the AWS CDK Toolkit \(the `cdk` command\):

```
npm install -g aws-cdk
```

**Note**  
If you get a permission error, and have administrator access on your system, try `sudo npm install -g aws-cdk`\.

Test the installation by issuing `cdk --version`\. 

If you get an error message at this point, try uninstalling \(`npm uninstall -g aws-cdk`\) and reinstalling\. As a last resort, delete the `node-modules` folder from the current project as well as the global `node-modules` folder\. To figure out where this folder is, issue `npm config get prefix`\.

### Language\-specific prerequisites<a name="work-with-prerequisites-language"></a>

The specific language you work in also has its own prerequisites, described in the corresponding topic listed here\.
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)
+ [Working with the AWS CDK in Go](work-with-cdk-go.md)

## AWS Construct Library<a name="work-with-library"></a>

The AWS CDK includes the AWS Construct Library, a collection of constructs organized by AWS service\. The library's constructs are mainly in a single module, colloquially called `aws-cdk-lib` because that's its name in TypeScript\. The actual package name of the main CDK package varies by language\.

------
#### [ TypeScript ]

| Install | `npm install aws-cdk-lib` | 
| --- |--- |
| Import | `const cdk = require('aws-cdk-lib');` | 
| --- |--- |

------
#### [ JavaScript ]

| Install | `npm install aws-cdk-lib` | 
| --- |--- |
| Import | `const cdk = require('aws-cdk-lib');` | 
| --- |--- |

------
#### [ Python ]

| Install | `python -m pip install aws-cdk-lib` | 
| --- |--- |
| Import | `import aws_cdk as cdk` | 
| --- |--- |

------
#### [ Java ]

| Add to `pom.xml` | Group `software.amazon.awscdk`; artifact `aws-cdk-lib` | 
| --- |--- |
| Import | `import software.amazon.awscdk.App;` \(for example\) | 
| --- |--- |

------
#### [ C\# ]

| Install | `dotnet add package Amazon.CDK.Lib` | 
| --- |--- |
| Import | `using Amazon.CDK;` | 
| --- |--- |

------

**Note**  
Experimental constructs are provided as separate modules\.

The [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) provides detailed documentation of the constructs \(and other components\) in the library\. A version of the API Reference is provided for each supported programming language\.

Each module's reference material is broken into the following sections\. 
+ *Overview*: Introductory material you'll need to know to work with the service in the AWS CDK, including concepts and examples\.
+ *Constructs*: Library classes that represent one or more concrete AWS resources\. These are the "curated" \(L2\) resources or patterns \(L3 resources\) that provide a high\-level interface with sane defaults\.
+ *Classes*: Non\-construct classes that provide functionality used by constructs in the module\.
+ *Structs*: Data structures \(attribute bundles\) that define the structure of composite values such as properties \(the `props` argument of constructs\) and options\.
+ *Interfaces*: Interfaces, whose names all begin with "I", define the absolute minimum functionality for the corresponding construct or other class\. The CDK uses construct interfaces to represent AWS resources that are defined outside your AWS CDK app and imported by methods such as `Bucket.fromBucketArn()`\. 
+ *Enums*: Collections of named values for use in specifying *certain construct parameters*. Using an enumerated value allows the CDK to check these values for validity during synthesis\.
+ *CloudFormation Resources*: These L1 constructs, whose names begin with "Cfn", represent exactly the resources defined in the CloudFormation specification\. They are automatically generated from that specification with each CDK release\. Each L2 or L3 construct encapsulates one or more CloudFormation resources\.
+ *CloudFormation Property Types*: The collection of named values that define the properties for each CloudFormation Resource\.

### Interfaces vs\. construct classes<a name="work-with-library-interfaces"></a>

The AWS CDK uses interfaces in a specific way that might not be obvious even if you are familiar with interfaces as a programming concept\.

The AWS CDK supports importing resources defined outside CDK applications using methods such as `Bucket.fromBucketArn()`\. Imported resources cannot be modified and may not have all the functionality available with resources defined in your CDK app using e\.g\. the `Bucket` class\. Interfaces, then, represent the bare minimum functionality available in the CDK for a given AWS resource type, *including imported resources\.*

When instantiating resources in your CDK app, then, you should always use concrete classes such as `Bucket`\. When specifying the type of an argument you are accepting in one of your own constructs, use the interface type such as `IBucket` if you are prepared to deal with imported resources \(that is, you won't need to change them\)\. If you require a CDK\-defined construct, specify the most general type you can use\.

Some interfaces are minimum versions of properties or options bundles \(shown in the AWS CDK API Reference as Structs\) that are associated with specific constructs\. For example, `IBucketProps` is the smallest set of properties required to instantiate a bucket\. Such interfaces can be useful when subclassing constructs to accept arguments that you'll pass on to your parent class\. If you require one or more additional properties, you'll want to implement or derive from this interface, or from a more specific type such as `BucketProps`\.

**Note**  
Some programming languages supported by the AWS CDK don't have an interface feature\. In these languages, interfaces are just ordinary classes\. You can identify them by their names, which follow the pattern of an initial "I" followed by the name of some other construct \(e\.g\. `IBucket`\)\. The same rules apply\.
