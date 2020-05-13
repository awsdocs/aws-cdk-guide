# Feature flags<a name="featureflags"></a>

The AWS CDK uses *feature flags* to enable potentially breaking behaviors in a release\. Flags are stored as [Runtime context](context.md) values in `cdk.json` \(or `~/.cdk.json`\) as shown here\.

```
{
  "app": "npx ts-node bin/tscdk.ts",
  "context": {
    "@aws-cdk/core:enableStackNameDuplicates": "true"
  }
}
```

The names of all feature flags begin with the NPM name of the package affected by the particular flag\. In the example above, this is `@aws-cdk/core`, the AWS CDK framework itself, since the flag affects stack naming rules, a core AWS CDK function\. AWS Construct Library modules can also use feature flags\.

Feature flags are disabled by default, so existing projects that do not specify the flag will continue to work as expected with later AWS CDK releases\. New projects created using cdk init include flags enabling all features available in the release that created the project\. Edit `cdk.json` to disable any flags for which you prefer the old behavior, or to add flags to enable new behaviors after upgrading the AWS CDK\.

See the `CHANGELOG` in a given release for a description of any new feature flags added in that release\. The AWS CDK source file [https://github.com/aws/aws-cdk/blob/master/packages/@aws-cdk/cx-api/lib/features.ts](https://github.com/aws/aws-cdk/blob/master/packages/@aws-cdk/cx-api/lib/features.ts) provides a complete list of all current feature flags\.

As feature flags are stored in `cdk.json`, they are not removed by the cdk context \-\-reset or cdk context \-\-clear commands\.