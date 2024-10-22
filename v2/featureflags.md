# AWS CDK feature flags<a name="featureflags"></a>

The AWS CDK uses *feature flags* to enable potentially breaking behaviors in a release\. Flags are stored as [Context values and the AWS CDK](context.md) values in `cdk.json` \(or `~/.cdk.json`\)\. They are not removed by the cdk context \-\-reset or cdk context \-\-clear commands\.

Feature flags are disabled by default\. Existing projects that do not specify the flag will continue to work as before with later AWS CDK releases\. New projects created using cdk init include flags enabling all features available in the release that created the project\. Edit `cdk.json` to disable any flags for which you prefer the earlier behavior\. You can also add flags to enable new behaviors after upgrading the AWS CDK\.

A list of all current feature flags can be found on the AWS CDK GitHub repository in [https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk-lib/cx-api/FEATURE_FLAGS.md](https://github.com/aws/aws-cdk/blob/main/packages/aws-cdk-lib/cx-api/FEATURE_FLAGS.md)\. See the `CHANGELOG` in a given release for a description of any new feature flags added in that release\. 

## Reverting to v1 behavior<a name="featureflags_disabling"></a>

In CDK v2, the defaults for some feature flags have been changed with respect to v1\. You can set these back to `false` to revert to specific AWS CDK v1 behavior\. Use the `cdk diff` command to inspect the changes to your synthesized template to see if any of these flags are needed\.

`@aws-cdk/core:newStyleStackSynthesis`  
Use the new stack synthesis method, which assumes bootstrap resources with well\-known names\. Requires [modern bootstrapping](bootstrapping.md), but in turn allows CI/CD via [CDK Pipelines](cdk_pipeline.md) and cross\-account deployments out of the box\.

`@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId`  
If your application uses multiple Amazon API Gateway API keys and associates them to usage plans\.

`@aws-cdk/aws-rds:lowercaseDbIdentifier`  
If your application uses Amazon RDS database instance or database clusters, and explicitly specifies the identifier for these\.

`@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021`  
 If your application uses the TLS\_V1\_2\_2019 security policy with Amazon CloudFront distributions\. CDK v2 uses security policy TLSv1\.2\_2021 by default\. 

`@aws-cdk/core:stackRelativeExports`  
If your application uses multiple stacks and you refer to resources from one stack in another, this determines whether absolute or relative path is used to construct AWS CloudFormation exports\.

`@aws-cdk/aws-lambda:recognizeVersionProps`  
If set to `false`, the CDK includes metadata when detecting whether a Lambda function has changed\. This can cause deployment failures when only the metadata has changed, since duplicate versions are not allowed\. There is no need to revert this flag if you've made at least one change to all Lambda Functions in your application\.

The syntax for reverting these flags in `cdk.json` is shown here\.

```
{
  "context": {
    "@aws-cdk/core:newStyleStackSynthesis": false,
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": false,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": false,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": false,
    "@aws-cdk/core:stackRelativeExports": false,
    "@aws-cdk/aws-lambda:recognizeVersionProps": false
  }
}
```
