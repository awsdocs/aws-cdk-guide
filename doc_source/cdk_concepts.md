--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# AWS CDK Concepts<a name="cdk_concepts"></a>

This topic describes some of the concepts \(the why and how\) behind the AWS CDK\. It also discusses the advantages of a AWS Construct Library over a low\-level CloudFormation Resource\.

AWS CDK apps are represented as a hierarchal structure we call the *construct tree*\. Every node in the tree is a [Construct](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#construct) object\. The root of an AWS CDK app is typically an [App](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#app) construct\. Apps contain one or more [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) constructs, which are deployable units of your app\.

This composition of constructs gives you the flexibility to architect your app, such as having a stack deployed in multiple regions\. Stacks represent a collection of AWS resources, either directly or indirectly through a child construct that itself represents an AWS resource, such as an Amazon SQS queue, an Amazon SNS topic, a Lambda function, or a DynamoDB table\.

This composition of constructs also means you can easily create sharable constructs, and make changes to any construct and have those changes available to consumers as shared class libraries\.