# Working with the AWS CDK in supported programming languages<a name="work-with"></a>

Use the AWS Cloud Development Kit \(AWS CDK\) to define your AWS Cloud infrastructure with a [supported programming language](languages.md)\.

**Topics**
+ [Importing the AWS Construct Library](#work-with-library)
+ [Managing dependencies](#work-with-cdk-dependencies)
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)
+ [Working with the AWS CDK in Go](work-with-cdk-go.md)

## Importing the AWS Construct Library<a name="work-with-library"></a>

The AWS CDK includes the AWS Construct Library, a collection of constructs organized by AWS service\. The library's stable constructs are offered in a single module, called by its TypeScript package name: `aws-cdk-lib`\. The actual package name varies by language\.

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

The `construct` base class and supporting code is in the `constructs` module\. Experimental constructs, where the API is still undergoing refinement, are distributed as separate modules\.

### The AWS CDK API Reference<a name="work-with-library-reference"></a>

The [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) provides detailed documentation of the constructs \(and other components\) in the library\. A version of the API Reference is provided for each supported programming language\.

Each module's reference material is broken into the following sections\.
+ *Overview*: Introductory material you'll need to know to work with the service in the AWS CDK, including concepts and examples\.
+ *Constructs*: Library classes that represent one or more concrete AWS resources\. These are the "curated" \(L2\) resources or patterns \(L3 resources\) that provide a high\-level interface with sane defaults\.
+ *Classes*: Non\-construct classes that provide functionality used by constructs in the module\.
+ *Structs*: Data structures \(attribute bundles\) that define the structure of composite values such as properties \(the `props` argument of constructs\) and options\.
+ *Interfaces*: Interfaces, whose names all begin with "I", define the absolute minimum functionality for the corresponding construct or other class\. The CDK uses construct interfaces to represent AWS resources that are defined outside your AWS CDK app and referenced by methods such as `Bucket.fromBucketArn()`\. 
+ *Enums*: Collections of named values for use in specifying certain construct parameters\. Using an enumerated value allows the CDK to check these values for validity during synthesis\.
+ *CloudFormation Resources*: These L1 constructs, whose names begin with "Cfn", represent exactly the resources defined in the CloudFormation specification\. They are automatically generated from that specification with each CDK release\. Each L2 or L3 construct encapsulates one or more CloudFormation resources\.
+ *CloudFormation Property Types*: The collection of named values that define the properties for each CloudFormation Resource\.

### Interfaces compared with construct classes<a name="work-with-library-interfaces"></a>

The AWS CDK uses interfaces in a specific way that may not be obvious even if you are familiar with interfaces as a programming concept\.

The AWS CDK supports using resources defined outside CDK applications using methods such as `Bucket.fromBucketArn()`\. External resources cannot be modified and may not have all the functionality available with resources defined in your CDK app using e\.g\. the `Bucket` class\. Interfaces, then, represent the bare minimum functionality available in the CDK for a given AWS resource type, *including external resources\.*

When instantiating resources in your CDK app, then, you should always use concrete classes such as `Bucket`\. When specifying the type of an argument you are accepting in one of your own constructs, use the interface type such as `IBucket` if you are prepared to deal with external resources \(that is, you won't need to change them\)\. If you require a CDK\-defined construct, specify the most general type you can use\.

Some interfaces are minimum versions of properties or options bundles associated with specific classes, rather than constructs\. Such interfaces can be useful when subclassing to accept arguments that you'll pass on to your parent class\. If you require one or more additional properties, you'll want to implement or derive from this interface, or from a more specific type\.

**Note**  
Some programming languages supported by the AWS CDK don't have an interface feature\. In these languages, interfaces are just ordinary classes\. You can identify them by their names, which follow the pattern of an initial "I" followed by the name of some other construct \(e\.g\. `IBucket`\)\. The same rules apply\.

## Managing dependencies<a name="work-with-cdk-dependencies"></a>

Dependencies for your AWS CDK app or library are managed using package management tools\. These tools are commonly used with the programming languages\.

Typically, the AWS CDK supports the language's standard or official package management tool if there is one\. Otherwise, the AWS CDK will support the language's most popular or widely supported one\. You may also be able to use other tools, especially if they work with the supported tools\. However, official support for other tools is limited\.

The AWS CDK supports the following package managers:


| Language | Supported package management tool | 
| --- | --- | 
| TypeScript/JavaScript | NPM \(Node Package Manager\) or Yarn | 
| Python | PIP \(Package Installer for Python\) | 
| Java | Maven | 
| C\# | NuGet | 
| Go | Go modules | 

When you create a new project using the AWS CDK CLI `cdk init` command, dependencies for the CDK core libraries and stable constructs are automatically specified\.

For more information on managing dependencies for supported programming languages, see the following:
+ [Managing dependencies in TypeScript](work-with-cdk-typescript.md#work-with-cdk-typescript-dependencies)\.
+ [Managing dependencies in JavaScript](work-with-cdk-javascript.md#work-with-cdk-javascript-dependencies)\.
+ [Managing dependencies in Python](work-with-cdk-python.md#work-with-cdk-python-dependencies)\.
+ [Managing dependencies in Java](work-with-cdk-java.md#work-with-cdk-java-dependencies)\.
+ [Managing dependencies in C\#](work-with-cdk-csharp.md#work-with-cdk-csharp-dependencies)\.
+ [Managing dependencies in Go](work-with-cdk-go.md#work-with-cdk-go-dependencies)\.