--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Use an Existing AWS CloudFormation Template<a name="use_cfn_template"></a>

The CDK provides a mechanism that you can use to incorporate resources from an existing AWS CloudFormation template into your CDK app\. For example, suppose you have a template, `my-template.json`, with the following resource, where *S3Bucket* is the logical ID of the bucket in your template:

```
{
  "S3Bucket": {
    "Type": "AWS::S3::Bucket",
    "Properties": {
      "prop1": "value1"
    }
  }
}
```

You can include this bucket in your CDK app, as shown in the following example \(note that you cannot use this method in an AWS Construct Library construct\):

```
import cdk = require("@amp;aws-cdk/cdk");
import fs = require("fs");

new cdk.Include(this, "ExistingInfrastructure", {
  template: JSON.parse(fs.readFileSync("my-template.json").toString())
});
```

Then to access an attribute of the resource, such as the bucket's ARN:

```
const bucketArn = new cdk.Fn.getAtt("S3Bucket", "Arn");
```