# API reference<a name="reference"></a>

The [API Reference](https://docs.aws.amazon.com/cdk/api/v2) contains information about the AWS Construct Library and other APIs provided by the AWS CDK\. Most of the AWS Construct Library is actually contained in a single package called `aws-cdk-lib` in NPM \(it has other names for other ecosystems\)\. The CDK API Reference is organized into submodules, one or more for each AWS service\.

Each submodule has an overview that includes information about how to use its APIs\. For example, the [S3](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3-readme.html) overview demonstrates how to set default encryption on an Amazon S3 bucket\.

Separate versions of the API Reference are provided for TypeScript/JavaScript, Python, Java, and C\#/\.NET\.

## Versioning<a name="versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and strictly adhere to the [semantic versioning](https://semver.org) model\. This means that breaking changes to stable APIs are limited to major releases\. Minor and patch releases are backward compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version within the same major version, and will continue to build and run, producing the same output\. 

### AWS CDK Toolkit \(CLI\) compatibility<a name="cdk_toolkit_versioning"></a>

The AWS CDK Toolkit \(that is, the `cdk` command line command\) is *always* compatible with construct libraries of a semantically *lower* or *equal* version number\. It is, therefore, always safe to upgrade the AWS CDK Toolkit within the same major version\. 

The AWS CDK Toolkit may be, but is *not always*, compatible with construct libraries of a semantically *higher* version, depending on whether the same cloud assembly schema version is employed by the two components\. The AWS CDK framework generates a cloud assembly during synthesis; the AWS CDK Toolkit consumes it for deployment\. The schema that defines the format of the cloud assembly is strictly specified and versioned\. AWS construct libraries using a given cloud assembly schema version are compatible with AWS CDK toolkit versions using that schema version or later, which may include releases of the AWS CDK Toolkit *older than* a given construct library release\.

When the cloud assembly version required by the construct library is not compatible with the version supported by the AWS CDK Toolkit, you receive an error message like this one\.

```
Cloud assembly schema version mismatch: Maximum schema version supported is 3.0.0, but found 4.0.0.
    Please upgrade your CLI in order to interact with this app.
```

To resolve this error, update the AWS CDK Toolkit to a version compatible with the required cloud assembly version, or simply to the latest available version\. The alternative \(downgrading the construct library modules your app uses\) is generally not desirable\.

**Note**  
For more details on the cloud assembly schema, see [Cloud Assembly Versioning](https://github.com/aws/aws-cdk/tree/master/packages/%40aws-cdk/cloud-assembly-schema#versioning)\.

### AWS Construct Library versioning<a name="aws_construct_lib_stability"></a>

The modules in the AWS Construct Library move through various stages as they are developed from concept to mature API\. Different stages imply different promises for API stability in subsequent versions of the AWS CDK\.

APIs in the main AWS CDK library, `aws-cdk-lib`, are stable, and the library is fully semantically\-versioned\. This package includes AWS CloudFormation \(L1\) constructs for all AWS services as well as all stable higher\-level \(L2/3\) modules\. \(It also includes the core CDK classes like `App` and `Construct`\.\) No APIs will be removed from this package \(though they may be deprecated\) until the next major release of the CDK\. No individual API will ever have breaking changes; if a breaking change is required, an entirely new API will be added\.

New APIs under development for a service already incorporated in `aws-cdk-lib` are identified using a `BetaN` suffix, where `N` starts at 1 and is incremented with each breaking change to the new API\. `BetaN` APIs are never removed, only deprecated, so your existing app continues to work with newer versions of `aws-cdk-lib`\. When the API is deemed stable, a new API without the `BetaN` suffix is added\.

When higher\-level \(L2 or L3\) APIs begin to be developed for an AWS service which previously had only L1 APIs, those APIs are initially distributed in a separate package\. The name of such a package has an "Alpha" suffix, and its version matches the first version of `aws-cdk-lib` it is compatible with, with an `alpha` sub\-version\. When the module supports the intended use cases, its APIs are added to `aws-cdk-lib`\.

### Language binding stability<a name="aws_construct_lib_versioning_binding"></a>

From time to time, we may add support to the AWS CDK for additional programming languages\. Although the API described in all the languages is the same, the way that API is expressed varies by language and may change as the language support evolves\. For this reason, language bindings are deemed experimental for a time until they are considered ready for production use\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Stable | 
| C\#/\.NET | Stable | 
| Go | Experimental | 