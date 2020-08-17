# API reference<a name="reference"></a>

The [API Reference](https://docs.aws.amazon.com/cdk/api/latest) contains information about the AWS CDK libraries\.

Each library contains information about how to use the library\. For example, the [S3](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-readme.html) library demonstrates how to set default encryption on an Amazon S3 bucket\.

## Versioning<a name="versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and generally adhere to the [semantic versioning](https://semver.org) model\. This means that breaking changes to stable APIs are limited to major releases\. Minor and patch releases are backward compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version and be expected to continue to build and run, producing the same output\. 

**Note**  
This compatibility promise does not apply to APIs under active development, which are designated as experimental\. See [AWS CDK stability index](#aws_construct_lib_stability) for more details\.

### AWS CDK Toolkit \(CLI\) compatibility<a name="cdk_toolkit_versioning"></a>

The AWS CDK Toolkit \(that is, the `cdk` command line command\) is *always* compatible with construct libraries of a semantically *lower* or *equal* version number\. It is, therefore, always safe to upgrade the AWS CDK Toolkit within the same major version\. 

The AWS CDK Toolkit may be, but is *not always*, compatible with construct libraries of a semantically *higher* version, depending on whether the same cloud assembly schema version is employed by the two components\. The AWS CDK framework generates a cloud assembly during synthesis; the AWS CDK Toolkit consumes it for deployment\. The schema that defines the format of the cloud assembly is strictly specified and versioned\. AWS construct libraries using a given cloud assembly schema version are compatible with AWS CDK toolkit versions using that schema version or later, which may include releases of the AWS CDK Toolkit *older than* a given construct library release\.

When the cloud assembly version required by the construct library is not compatible with the version supported by the AWS CDK Toolkit, you receive an error message like this one\.

```
Cloud assembly schema version mismatch: Maximum schema version supported is 3.0.0, but found 4.0.0.
    Please upgrade your CLI in order to interact with this app.
```

**Note**  
For more details on the cloud assembly schema, see [Cloud Assembly Versioning](https://github.com/aws/aws-cdk/tree/master/packages/%40aws-cdk/cloud-assembly-schema#versioning)\.

### AWS CDK stability index<a name="aws_construct_lib_stability"></a>

The modules in the AWS Construct Library move through various stages as they are developed from concept to mature API\. Different stages imply different promises for API stability in subsequent versions of the AWS CDK\.

Stage 0: CFN resources  
All construct library modules start in stage 0 when they are auto\-generated from the AWS CloudFormation resource specification\. The goal of stage 0 is to make new AWS CloudFormation resources/properties available to AWS CDK customers as quickly as possible\. We capture feedback from customers to better understand what L2 resources to add\.  
AWS CloudFormation resources themselves are considered stable APIs, regardless of whether other constructs in the module are under active development\.

Stage 1: Experimental  
The goal of the experimental stage is to retain the freedom to make breaking changes to APIs while we design and build a module\. During this stage, the primary use cases and the set of L2 constructs required to support them are incrementally identified, implemented, and validated\.  
Development of L2 constructs is community\-oriented and transparent\. For large and/or complex changes, we author a Request for Comments \(RFC\) that outlines our intended design and publish it for feedback\. We also use pull requests to conduct API design reviews\.  
At this stage, individual APIs may be in flux, and breaking changes may occur from release to release if we deem these necessary to support customer use cases\.

Stage 2: Developer preview \(DP\)  
At the developer preview stage, our aim is to deliver a release candidate with a stable API with which to conduct user acceptance testing\. When the API passes acceptance, it is deemed suitable for general availability\.  
We make breaking changes at this stage only when required to address unforeseen customer use cases or issues\. Since breaking changes are still possible, the package itself retains the "experimental" label while in developer preview\.

Stage 3: General availability \(GA\)  
The module is generally available with a compatibility guarantee across minor versions\. We will only make backward\-compatible changes to the API, so that your existing apps will continue to work until the next major AWS CDK release\.  
In some cases, we may use [feature flags](featureflags.md) to optionally enable new behavior while retaining the previous behavior to support existing apps\.

Each module's Overview in the [API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) describes its stability level\.

For more information about these maturity stages, see [AWS Construct Library Module Lifecycle](https://github.com/aws/aws-cdk-rfcs/blob/master/text/0107-construct-library-module-lifecycle.md)\.

### Language binding stability<a name="aws_construct_lib_versioning_binding"></a>

From time to time, we may add support to the AWS CDK for additional programming languages\. Although the API described in all the languages is the same, the way that API is expressed varies by language and may change as the language support evolves\. For this reason, language bindings are deemed experimental for a time until they are considered ready for production use\. Currently, all supported languages are considered stable\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Stable | 
| C\#/\.NET | Stable | 