# API Reference<a name="reference"></a>

The [API Reference](https://docs.aws.amazon.com/cdk/api/latest) contains information about the AWS CDK libraries\.

Each library contains information about how to use the library\. For example, the [S3](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-readme.html) library demonstrates how to set default encryption on an Amazon S3 bucket\.

## Versioning<a name="versioning"></a>

Version numbers consist of three numeric version parts: *major*\.*minor*\.*patch*, and adhere to the [semantic versioning](https://semver.org) model\. This means that breaking changes to stable APIs are limited to major releases\. Minor and patch releases are backward compatible, meaning that the code written in a previous version with the same major version can be upgraded to a newer version and be expected to continue to build and run, producing the same output\.

**Note**  
This compatibility promise does not apply to APIs designated as experimental\. See [AWS CDK Stability Index](#aws_construct_lib_stability) for more details\.

### AWS CDK Toolkit \(CLI\) Compatibility<a name="aws_construct_lib_versioning"></a>

The AWS CDK Toolkit \(that is, the `cli` command line command\) is *always* compatible with construct libraries of a semantically *lower* or *equal* version number\. It is, therefore, always safe to upgrade the AWS CDK Toolkit within the same major version\. 

The AWS CDK Toolkit may be, but is *not always*, compatible with construct libraries of a semantically *higher* version, depending on whether the same cloud assembly schema version is employed by the two components\. The AWS CDK framework generates a cloud assembly during synthesis; the AWS CDK Toolkit consumes it for deployment\. The schema that defines the format of the cloud assembly is strictly specified and versioned\. AWS construct libraries using a given cloud assembly schema version are compatible with AWS CDK toolkit versions using that schema version or later, which may include releases of the AWS CDK Toolkit *older than* a given construct library release\.

When the cloud assembly version required by the construct library is not compatible with the version supported by the AWS CDK Toolkit, you receive an error message like this one\.

```
Cloud assembly schema version mismatch: Maximum schema version supported is 3.0.0, but found 4.0.0.
    Please upgrade your CLI in order to interact with this app.
```

**Note**  
For more details on the cloud assembly schema, see [Cloud Assembly Versioning](https://github.com/aws/aws-cdk/tree/master/packages/%40aws-cdk/cloud-assembly-schema#versioning)\.

### AWS CDK Stability Index<a name="aws_construct_lib_stability"></a>

Certain APIs do not adhere to the semantic versioning model\. There are three levels of stability in the AWS Construct Library, which define the level of semantic versioning that applies to each module\.

Stable  
The API is subject to the semantic versioning model\. We will not introduce non\-backward\-compatible changes or remove the API in a subsequent patch or feature release\.

CloudFormation Only  
These APIs are automatically built from the AWS CloudFormation resource specification and are subject only to changes introduced by AWS CloudFormation\.

Experimental  
The API is still under active development and subject to non\-backward\-compatible changes or removal in any future version\. Such changes are announced in the AWS CDK release notes under *BREAKING CHANGES*\. We recommend that you do not use this API in production environments\. Experimental APIs are not subject to the semantic versioning model\.

Deprecated  
The API may emit warnings\. We do not guarantee backward compatibility\.

Experimental and stable modules receive the same level of support from AWS\. The only difference is that we might change experimental APIs within a major version\. Although we don't recommend using experimental APIs in production, we vet them the same way as we vet stable APIs before we include them in a release\.

### Identifying the Support Level of an API<a name="aws_construct_lib_versioning_support"></a>

Each module in the [API Reference](https://docs.aws.amazon.com/cdk/api/latest) starts with a section outlining the module's stability index\. The libraries that include only AWS CloudFormation resources, and no hand\-curated constructs, are labeled with the maturity indicator **CloudFormation\-only**\.

The module level gives an indication of the stability of the majority of the APIs included in the module, however, individual APIs within the module can be annotated with different stability levels\.


| Stability | TypeScript | JavaScript | Python | C\#/\.NET | Java | 
| --- |--- |--- |--- |--- |--- |
| Experimental | @experimental | @stability Experimental | @experimental | Stability: Experimental | Stability: Experimental | 
| Stable | @stable | @stability Stable | @stable | Stability: Stable | Stability: Stable | 
| Deprecated | @deprecated | @Deprecated | @deprecated | \[Obsolete\] | Stability: Deprecated | 

### Language Binding Stability<a name="aws_construct_lib_versioning_binding"></a>

In addition to modules of the AWS CDK Construct Library, language support is also subject to a stability indication\. Although the API described in all the languages is the same, the way that API is expressed varies by language and may change as the language support evolves\. For this reason, language bindings are deemed experimental for a time until they are considered ready for production use\.


| Language | Stability | 
| --- |--- |
| TypeScript | Stable | 
| JavaScript | Stable | 
| Python | Stable | 
| Java | Stable | 
| C\#/\.NET | Stable | 