# Feature flags<a name="featureflags"></a>

The AWS CDK uses *feature flags* to enable potentially breaking behaviors in a release\. Flags are stored as [Runtime context](context.md) values in `cdk.json` \(or `~/.cdk.json`\)\. They are not removed by the cdk context \-\-reset or cdk context \-\-clear commands\.

Feature flags are disabled by default\. Existing projects that do not specify the flag will continue to work as before with later AWS CDK releases\. New projects created using cdk init include flags enabling all features available in the release that created the project\. Edit `cdk.json` to disable any flags for which you prefer the earlier behavior\. You can also add flags to enable new behaviors after upgrading the AWS CDK\.

See the `CHANGELOG` in a given release for a description of any new feature flags added in that release\. The AWS CDK source file [https://github.com/aws/aws-cdk/blob/master/packages/@aws-cdk/cx-api/lib/features.ts](https://github.com/aws/aws-cdk/blob/master/packages/@aws-cdk/cx-api/lib/features.ts) provides a complete list of all current feature flags\.

## Enabling features with flags<a name="w322aac23c34b9"></a>

The following feature flags may be set to `true` to enable the described behavior\.

`@aws-cdk/core:checkSecretUsage`  
Makes it impossible to use Secrets Manager values in unsafe locations\.

`@aws-cdk/aws-lambda:recognizeLayerVersion`  
Verify that updating a layer associated with a Lambda function creates a new version of the function\.

`@aws-cdk/core:target-partitions`  
Verify that Amazon EC2 Systems Manager service principals are generated correctly\.

`@aws-cdk-containers/ecs-service-extensions:enableDefaultLogDriver`  
Enables logging in service extensions containers by default\.

`@aws-cdk/aws-ec2:uniqueImdsv2TemplateName`  
Causes [https://docs.aws.amazon.com/cdk/api/v1/docs/@aws-cdk_aws-ec2.InstanceRequireImdsv2Aspect.html](https://docs.aws.amazon.com/cdk/api/v1/docs/@aws-cdk_aws-ec2.InstanceRequireImdsv2Aspect.html) to verify that the generated name is unique\.

`@aws-cdk/aws-iam:minimizePolicies`  
Minimize the creation of IAM policies when possible\.

`@aws-cdk/aws-sns-subscriptions:restrictSqsDecryption`  
In an Amazon SQS queue subscribed to an Amazon SNS topic, restrict decryption permissions to only the topic instead of all of Amazon SNS\.

`@aws-cdk/aws-s3:createDefaultLoggingPolicy`  
When using an S3 bucket with a service that will automatically create a bucket policy at deployment, have the AWS CDK configure the necessary policy\.

`@aws-cdk/aws-codepipeline:crossAccountKeyAliasStackSafeResourceName`  
Make sure cross\-account key alias is unique in pipelines\.

`@aws-cdk/core:validateSnapshotRemovalPolicy`  
The AWS CDK fails at synthesis time if the `SNAPSHOT` removal policy is not supported for a given resource\.

`@aws-cdk/aws-ecs:arnFormatIncludesClusterName`  
Use the new ARN format when importing an Amazon EC2 or Fargate cluster\.

## Disabling features with flags<a name="featureflags_disabling"></a>

In CDK v2, a few feature flags are supported to revert certain behaviors to their v1 defaults\. The following flags, set to `false`, revert to specific AWS CDK v1 behaviors\. Use the `cdk diff` command to inspect the changes to your synthesized template to see if any of these flags are needed\.

`@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId`  
If your application uses multiple Amazon API Gateway API keys and associates them to usage plans\.

`@aws-cdk/aws-rds:lowercaseDbIdentifier`  
If your application uses Amazon RDS database instance or database clusters, and explicitly specifies the identifier for these\.

`@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021`  
 If your application uses the TLS\_V1\_2\_2019 security policy with Amazon CloudFront distributions\. CDK v2 uses security policy TLSv1\.2\_2021 by default\. 

`@aws-cdk/core:stackRelativeExports`  
If your application uses multiple stacks and you refer to resources from one stack in another, this determines whether absolute or relative path is used to construct AWS CloudFormation exports\.

`@aws-cdk/aws-lambda:recognizeVersionProps`  
If set to `false`, the CDK includes metadata when detecting whether a Lambda function has changed\. This can cause deployment failures when only the metadata has changed, since duplicate versions are not allowed\.

The syntax for reverting these flags in `cdk.json` is shown here\.

```
{
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": false,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": false,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": false,
    "@aws-cdk/core:stackRelativeExports": false,
  }
}
```