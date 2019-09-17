# Use an Existing AWS CloudFormation Template<a name="use_cfn_template"></a>

The AWS CDK provides a mechanism that you can use to incorporate resources from an existing AWS CloudFormation template into your AWS CDK app\. For example, suppose you have a template, `my-template.json`, with the following resource, where *S3Bucket* is the logical ID of the bucket in your template:

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

You can include this bucket in your AWS CDK app, as shown in the following example:

```
import cdk = require("@aws-cdk/core");
import fs = require("fs");

new cdk.CfnInclude(this, "ExistingInfrastructure", {
  template: JSON.parse(fs.readFileSync("my-template.json").toString())
});
```

Then to access an attribute of the resource, such as the bucket's ARN:

```
const bucketArn = cdk.Fn.getAtt("S3Bucket", "Arn");
```