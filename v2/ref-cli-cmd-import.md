# cdk import<a name="ref-cli-cmd-import"></a>

Use [AWS CloudFormation resource imports](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import.html) to import existing AWS resources into a CDK stack\.

With this command, you can take existing resources that were created using other methods and start managing them using the AWS CDK\.

When considering moving resources into CDK management, sometimes creating new resources is acceptable, such as with IAM roles, Lambda functions, and event rules\. For other resources, such as stateful resources like Amazon S3 buckets and DynamoDB tables, creating new resources can cause impacts to your service\. You can use `cdk import` to import existing resources with minimal disruption to your services\. For a list of supported AWS resources, see [Resource type support](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resource-import-supported-resources.html) in the *AWS CloudFormation User Guide*\.

**To import an existing resource to a CDK stack**

1. Run a `cdk diff` to make sure your CDK stack has no pending changes\. When performing a `cdk import`, the only changes allowed in an import operation are the addition of new resources being imported\.

1. Add constructs for the resources you want to import to your stack\. For example, add the following for an Amazon S3 bucket:

   ```
   new s3.Bucket(this, 'ImportedS3Bucket', {});
   ```

   Do not add any other changes\. You must also make sure to exactly model the state that the resource currently has\. For the bucket example, be sure to include AWS KMS keys, lifecycle policies, and anything else that is relevant about the bucket\. Otherwise, subsequent update operations may not do what you expect\.

1. Run `cdk import`\. If there are multiple stacks in the CDK app, pass a specific stack name as an argument\.

1. The CDK CLI will prompt you to pass in the actual names of the resources you are importing\. After you provide this information, import will begin\.

1. When `cdk import` reports success, the resource will be managed by the CDK\. Any subsequent changes in the construct configuration will be reflected on the resource\.

This feature currently has the following limitations:
+ Importing resources into nested stacks isn’t possible\.
+ There is no check on whether the properties you specify are correct and complete for the imported resource\. Try starting a drift detection operation after importing\.
+ Resources that depend on other resources must all be imported together, or individually, in the right order\. Otherwise, the CloudFormation deployment will fail with unresolved references\.
+ This command uses the deploy role credentials, which is necessary to read the encrypted staging bucket\. This requires version 12 of the bootstrap template, which includes the necessary IAM permissions for the deploy role\.

## Usage<a name="ref-cli-cmd-import-usage"></a>

```
$ cdk import <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-import-args"></a>

**CDK stack ID**  <a name="ref-cli-cmd-import-args-stack-name"></a>
The construct ID of the CDK stack from your app to import resources to\. This argument can be provided multiple times in a single command\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-import-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--change-set-name STRING`  <a name="ref-cli-cmd-import-options-change-set-name"></a>
The name of the CloudFormation change set to create\.

`--execute BOOLEAN`  <a name="ref-cli-cmd-import-options-execute"></a>
Specify whether to execute change set\.  
*Default value*: `true`

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-import-options-force"></a>
By default, the CDK CLI exits the process if the template diff includes updates or deletions\. Specify `true` to override this behavior and always continue with importing\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-import-options-help"></a>
Show command reference information for the `cdk import` command\.

`--record-resource-mapping, -r STRING`  <a name="ref-cli-cmd-import-options-record-resource-mapping"></a>
Use this option to generate a mapping of existing physical resources to the CDK resources that will be imported\. The mapping will be written to the file path that you provide\. No actual import operations will be performed\. 

`--resource-mapping, -m STRING`  <a name="ref-cli-cmd-import-options-resource-mapping"></a>
Use this option to specify a file that defines your resource mapping\. The CDK CLI will use this file to map physical resources to resources for import instead of interactively asking you\.  
This option can be run from scripts\.

`--rollback BOOLEAN`  <a name="ref-cli-cmd-import-options-rollback"></a>
Roll back the stack to stable state on failure\.  
To specify `false`, you can use `--no-rollback` or `-R`\.  
Specify `false` to iterate more rapidly\. Deployments containing resource replacements will always fail\.   
*Default value*: `true`

`--toolkit-stack-name STRING`  <a name="ref-cli-cmd-import-options-toolkit-stack-name"></a>
The name of the CDK Toolkit stack to create\.  
By default, `cdk bootstrap` deploys a stack named `CDKToolkit` into the specified AWS environment\. Use this option to provide a different name for your bootstrap stack\.  
The CDK CLI uses this value to verify your bootstrap stack version\.