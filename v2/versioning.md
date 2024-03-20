# AWS CDK versioning<a name="versioning"></a>

This topic provides reference information on how the AWS Cloud Development Kit \(AWS CDK\) handles versioning\.

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and strictly adhere to the [semantic versioning](https://semver.org) model\. This means that breaking changes to stable APIs are limited to major releases\.

Minor and patch releases are backward compatible\. The code written in a previous version with the same major version can be upgraded to a newer version within the same major version\. It will also continue to build and run, producing the same output\.

**Topics**
+ [AWS CDK CLI compatibility](#cdk_toolkit_versioning)
+ [AWS Construct Library versioning](#aws_construct_lib_stability)
+ [Language binding stability](#aws_construct_lib_versioning_binding)

## AWS CDK CLI compatibility<a name="cdk_toolkit_versioning"></a>

The AWS CDK CLI is *always* compatible with construct libraries of a semantically *lower* or *equal* version number\. It is, therefore, always safe to upgrade the AWS CDK CLI within the same major version\.

The AWS CDK CLI is *not always* compatible with construct libraries of a semantically *higher* version\. Compatibility depends on whether the same cloud assembly schema version is employed by the two components\. The AWS CDK framework generates a cloud assembly during synthesis and the AWS CDK CLI consumes it for deployment\. The schema that defines the format of the cloud assembly is strictly specified and versioned\.

AWS construct libraries using a given cloud assembly schema version are compatible with AWS CDK CLI versions using that schema version or later\. This might include releases of the AWS CDK CLI that are *earlier than* a given construct library release\.

When the cloud assembly version required by the construct library is not compatible with the version supported by the AWS CDK CLI, you receive an error message like the following:

```
Cloud assembly schema version mismatch: Maximum schema version supported is 3.0.0, but found 4.0.0.
    Please upgrade your CLI in order to interact with this app.
```

To resolve this error, update the AWS CDK CLI to a version compatible with the required cloud assembly version, or to the latest available version\. The alternative \(downgrading the construct library modules your app uses\) is generally not recommended\.

**Note**  
For more details on the cloud assembly schema, see [Cloud Assembly Versioning](https://github.com/aws/aws-cdk/tree/master/packages/%40aws-cdk/cloud-assembly-schema#versioning)\.

## AWS Construct Library versioning<a name="aws_construct_lib_stability"></a>

The modules in the AWS Construct Library move through various stages as they are developed from concept to mature API\. Different stages offer varying degrees of API stability in subsequent versions of the AWS CDK\.

APIs in the main AWS CDK library, `aws-cdk-lib`, are stable, and the library is fully semantically versioned\. This package includes AWS CloudFormation \(L1\) constructs for all AWS services and all stable higher\-level \(L2 and L3\) modules\. \(It also includes the core CDK classes like `App` and `Stack`\)\. APIs will not be removed from this package \(though they may be deprecated\) until the next major release of the CDK\. No individual API will ever have breaking changes\. When a breaking change is required, an entirely new API will be added\.

New APIs under development for a service already incorporated in `aws-cdk-lib` are identified using a `BetaN` suffix, where `N` starts at 1 and is incremented with each breaking change to the new API\. `BetaN` APIs are never removed, only deprecated, so your existing app continues to work with newer versions of `aws-cdk-lib`\. When the API is deemed stable, a new API without the `BetaN` suffix is added\.

When higher\-level \(L2 or L3\) APIs begin to be developed for an AWS service that previously had only L1 APIs, those APIs are initially distributed in a separate package\. The name of such a package has an "Alpha" suffix, and its version matches the first version of `aws-cdk-lib` it is compatible with, with an `alpha` sub\-version\. When the module supports the intended use cases, its APIs are added to `aws-cdk-lib`\.

## Language binding stability<a name="aws_construct_lib_versioning_binding"></a>

Over time, we might add support to the AWS CDK for additional programming languages\. Although the API described in all the languages is the same, the way that API is expressed varies by language and might change as the language support evolves\. For this reason, language bindings are deemed experimental for a time until they are considered ready for production use\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Stable | 
| C\#/\.NET | Stable | 
| Go | Stable | 