--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS Construct Library<a name="aws_construct_lib"></a>

The AWS Construct Library is a set of modules that expose APIs for defining AWS resources in AWS CDK apps\. Each module is based on the AWS service to which the resource belongs\. For example, [EC2](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ec2-readme.html) includes the [Vpc](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-ec2/vpc.html) construct, which makes it easy to define an Amazon VPC in your AWS CDK app\.

The AWS Construct Library modules are described in the [AWS CDK Reference](https://awslabs.github.io/aws-cdk/)\.

## Versioning<a name="aws_construct_lib_versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and adhere to the [Semantic Versioning](https://semver.org) model\. This means that breaking changes to stable APIs \(see “The AWS CDK Stability Index”\) are limited to major releases\. Minor and patch releases are backwards\-compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version and be expected to continue to build and run, producing the same output\.

### AWS CDK Stability Index<a name="aws_construct_lib_versioning_stability"></a>

However, certain APIs do not adher to the semantic versioning model\. The AWS CDK team has added a stability index to help developers determine which APIs deviate from the semantic versioning model\.

There are three levels of stability in the AWS CDK Construct Library:

Stable  
The API is subject to the semantic versioning model\. We will not introduce non\-backward compatible changes or remove the API in a subsequent patch or feature release\.

CloudFormation Only  
These APIs are automatically built from the AWS CloudFormation resource specification and are subject to any changes introduced by AWS CloudFormation\.

Experimental  
The API is still under active development and subject to non\-backward compatible changes or removal in any future version\. We recommend that you do not use this API in production environments\. Experimental APIs are not subject to the semantic versioning model\.

Deprecated  
The API may emit warnings\. We do not guarantee backward compatibility\.

Experimental and stable modules receive the same level of support from AWS\. The only difference is that we might change experimental APIs within a major version\. While we don not recommend using experimental APIs in production, we vet them the same way as we vet stable APIs before we include them in an AWS CDK release\.

### Identifying the Support Level of an API<a name="aws_construct_lib_versioning_support"></a>

The AWS CDK Developer guide and API Reference for each module starts with a section outlining the module's stability index\. The libraries that include only AWS CloudFormation Resources, and no hand\-curated constructs, are labelled with the maturity indicator **CloudFormation\-only**\.

The module\-level gives an indication of the stability of the majority of the APIs included in the module, however individual APIs within the module can be annotated with different stability levels:


| Stability | TypeScript | JavaScript | Python | C\#/\.NET | Java | 
| --- |--- |--- |--- |--- |--- |
| Experimental | @experimental | @stability Experimental | @experimental | Stability: Experimental | Stability: Experimental | 
| Stable | @stable | @stability Stable | @stable | Stability: Stable | Stability: Stable | 
| Deprecated | @deprecated | @Deprecated | @deprecated | \[Obsolete\] | Stability: Deprecated | 

### Language Binding Stability<a name="aws_construct_lib_versioning_binding"></a>

In addition to modules of the AWS CDK Construct Library, language support is also subject to a stability indication\. While the API described in all the languages is the same, the specific programming model and naming of language bindings considered experimental can change as it stabilizes\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Experimental | 
| C\#/\.NET | Experimental | 