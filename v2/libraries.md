# The AWS CDK libraries<a name="libraries"></a>

Learn about the core libraries that you will use with the AWS Cloud Development Kit \(AWS CDK\)\.

## The AWS CDK Library<a name="libraries-cdk"></a>

The AWS CDK Library, also referred to as `aws-cdk-lib`, is the main library that you will use to develop applications with the AWS CDK\. It is developed and maintained by AWS\. This library contains base classes, such as `[App](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html)` and `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)`\. It also contains the libraries you will use to define your infrastructure through constructs\.

## The AWS Construct Library<a name="libraries-construct"></a>

The AWS Construct Library is a part of the AWS CDK Library\. It contains a collection of [constructs](constructs.md) that are developed and maintained by AWS\. It is organized into various modules for each AWS service\. Each module includes constructs that you can use to define your AWS resources and properties\.

## The Constructs library<a name="libraries-constructs"></a>

The Constructs library, commonly referred to as `constructs`, is a library for defining and composing cloud infrastructure components\. It contains the core `[Construct](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html)` class, which represents the construct building block\. This class is the foundational base class of all constructs from the AWS Construct Library\. The Constructs library is a separate general\-purpose library that is used by other construct\-based tools such as *CDK for Terraform* and *CDK for Kubernetes*\.

## The AWS CDK API reference<a name="libraries-reference"></a>

The [AWS CDK API reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) contains official reference documentation for the AWS CDK Library, including the AWS Construct Library and Constructs library\. A version of the API refernce is provided for each supported programming language\.
+ For AWS CDK Library \(`aws-cdk-lib`\) documentation, see [aws\-cdk\-lib module](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html)\.
+ Documentation for constructs in the AWS Construct Library are organized by AWS service in the following format: `aws-cdk-lib.<service>`\. For example, construct documentation for Amazon Simple Storage Service \(Amazon S3\), can be found at [aws\-cdk\-lib\.aws\_s3 module](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3-readme.html)\.
+ For Constructs library \(constructs\) documentation, see [constructs module](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs-readme.html)\.

### Contribute to the AWS CDK API reference<a name="libraries-reference-contribute"></a>

The AWS CDK is open\-source and we welcome you to contribute\. Community contributions positively impact and improve the AWS CDK\. For instructions on contributing specifically to AWS CDK API reference documentation, see [Documentation](https://github.com/aws/aws-cdk/blob/main/CONTRIBUTING.md#documentation) in the *aws\-cdk GitHub repository*\.

## Learn more<a name="libraries-learn"></a>

For instructions on importing and using the CDK Library, see [Work with the CDK library](work-with.md)\.