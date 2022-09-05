# Feature flags<a name="featureflags"></a>

The AWS CDK uses *feature flags* to enable potentially breaking behaviors in a release\. Flags are stored as [Runtime context](context.md) values in `cdk.json` \(or `~/.cdk.json`\)\. They are not removed by the cdk context \-\-reset or cdk context \-\-clear commands\.

Feature flags are disabled by default, so existing projects that do not specify the flag will continue to work as expected with later AWS CDK releases\. New projects created using cdk init include flags enabling all features available in the release that created the project\. Edit `cdk.json` to disable any flags for which you prefer the old behavior, or to add flags to enable new behaviors after upgrading the AWS CDK\.

See the `CHANGELOG` for a given release for a description of any new feature flags added in that release\. The AWS CDK source file [https://github.com/aws/aws-cdk/blob/main/packages/@aws-cdk/cx-api/lib/features.ts](https://github.com/aws/aws-cdk/blob/main/packages/@aws-cdk/cx-api/lib/features.ts) provides a complete list of all current feature flags\. 

In CDK v2, feature flags are also used to revert certain behaviors to their v1 defaults\. The flags listed below, set to `false`, revert to specific v1 AWS CDK v1 behaviors\. Use the `cdk diff` command to inspect the changes to your synthesized template to see if any of these flags are needed\.

`@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId`  
If your application uses multiple Amazon API Gateway API keys and associates them to usage plans

`@aws-cdk/aws-rds:lowercaseDbIdentifier`  
If your application uses Amazon RDS database instance or database clusters, and explicitly specifies the identifier for these

`@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021`  
 If your application uses the TLS\_V1\_2\_2019 security policy with Amazon CloudFront distributions\. CDK v2 uses security policy TLSv1\.2\_2021 by default\. 

`@aws-cdk/core:stackRelativeExports`  
If your application uses multiple stacks and you refer to resources from one stack in another, this determines whether absolute or relative path is used to construct AWS CloudFormation exports

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

