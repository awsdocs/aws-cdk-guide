# API Reference<a name="reference"></a>

The [API Reference](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-construct-library.html) contains information about the AWS CDK libraries\.

Each library contains information about how to use the library\. For example, the [S3](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-readme.html) library demonstrates how to set default encryption on an Amazon S3 bucket\.

## Versioning and Stability Model<a name="versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and adhere to the [semantic versioning](https://semver.org) model\. This means that breaking changes to stable APIs \(see “The AWS CDK Stability Index”\) are limited to major releases\. Minor and patch releases are backward compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version and be expected to continue to build and run, producing the same output\.

### AWS CDK Stability Index<a name="aws_construct_lib_versioning_stability"></a>

However, certain APIs do not adhere to the semantic versioning model\. The AWS CDK team has added a stability index to help developers determine which APIs deviate from the semantic versioning model\.

There are three levels of stability in the AWS CDK Construct Library:

Stable  
The API is subject to the semantic versioning model\. We will not introduce non\-backward\-compatible changes or remove the API in a subsequent patch or feature release\.

CloudFormation Only  
These APIs are automatically built from the AWS CloudFormation resource specification and are subject to any changes introduced by AWS CloudFormation\.

Experimental  
The API is still under active development and subject to non\-backward\-compatible changes or removal in any future version\. We recommend that you do not use this API in production environments\. Experimental APIs are not subject to the semantic versioning model\.

Deprecated  
The API may emit warnings\. We do not guarantee backward compatibility\.

Experimental and stable modules receive the same level of support from AWS\. The only difference is that we might change experimental APIs within a major version\. Although we don't recommend using experimental APIs in production, we vet them the same way as we vet stable APIs before we include them in an AWS CDK release\.

### Identifying the Support Level of an API<a name="aws_construct_lib_versioning_support"></a>

The AWS CDK Developer Guide and API Reference for each module starts with a section outlining the module's stability index\. The libraries that include only AWS CloudFormation resources, and no hand\-curated constructs, are labeled with the maturity indicator **CloudFormation\-only**\.

The module level gives an indication of the stability of the majority of the APIs included in the module, however, individual APIs within the module can be annotated with different stability levels\.


| Stability | TypeScript | JavaScript | Python | C\#/\.NET | Java | 
| --- |--- |--- |--- |--- |--- |
| Experimental | @experimental | @stability Experimental | @experimental | Stability: Experimental | Stability: Experimental | 
| Stable | @stable | @stability Stable | @stable | Stability: Stable | Stability: Stable | 
| Deprecated | @deprecated | @Deprecated | @deprecated | \[Obsolete\] | Stability: Deprecated | 

### Language Binding Stability<a name="aws_construct_lib_versioning_binding"></a>

In addition to modules of the AWS CDK Construct Library, language support is also subject to a stability indication\. Although the API described in all the languages is the same, the specific programming model and naming of language bindings is considered experimental and can change as it stabilizes\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Experimental | 
| C\#/\.NET | Experimental | 
