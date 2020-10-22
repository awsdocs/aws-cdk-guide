# Import or migrate an existing AWS CloudFormation template<a name="use_cfn_template"></a>

The AWS CDK provides two mechanisms to incorporate resources from an existing AWS CloudFormation template into your AWS CDK app\.
+ [`core.CfnInclude`](#use_cfn_template_core) \- Includes an AWS CloudFormation template in the current stack, merging its resources with those defined in your AWS CDK code\. You can access attributes of the imported resources using [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Fn.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Fn.html), but these resources are not actually AWS CDK constructs and do not provide the functionality of real constructs\.
+ [`cloudformation-include.CfnInclude`](#use_cfn_template_cfninclude) \- This construct, currently in developer preview, actually converts the resources in the imported AWS CloudFormation template to AWS CDK L1 constructs\. You can work with these in your app just as if they were defined in AWS CDK code, even using them within higher\-level AWS CDK constructs, letting you use \(for example\) the L2 permission grant methods with the resources they define\. 

  This construct essentially adds an AWS CDK API wrapper to any resource in the template\. You can use this capability to migrate your existing AWS CloudFormation templates to the AWS CDK a piece at a time in order to take advantage of the AWS CDK's convenient higher\-level abstractions, or just to vend your AWS CloudFormation templates to AWS CDK developers by giving them the API they expect\.

## Using `core.CfnInclude`<a name="use_cfn_template_core"></a>

An AWS CloudFormation template included using [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnInclude.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnInclude.html) provides your AWS CDK app basic access to the resources contained in the template\.

Suppose you have a template, `my-template.json`, with the following resource, where *S3Bucket* is the logical ID of the bucket in your template:

```
{
  "MyBucket": {
    "Type": "AWS::S3::Bucket",
    "Properties": {
      "BucketName": "MyBucket",
    }
  }
}
```

You can include this bucket in your AWS CDK app, as shown in the following example\.

------
#### [ TypeScript ]

```
import * as cdk from "@aws-cdk/core";
import * as fs from "fs";

new cdk.CfnInclude(this, "ExistingInfrastructure", {
  template: JSON.parse(fs.readFileSync("my-template.json").toString())
});
```

------
#### [ JavaScript ]

```
const cdk = require("@aws-cdk/core");
const fs = require("fs");

new cdk.CfnInclude(this, "ExistingInfrastructure", {
  template: JSON.parse(fs.readFileSync("my-template.json").toString())
});
```

------
#### [ Python ]

```
import json

cdk.CfnInclude(self, "ExistingInfrastructure",
    template=json.load(open("my-template.json"))
```

------
#### [ Java ]

```
import java.util.*;
import java.io.File;

import software.amazon.awscdk.core.CfnInclude;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

CfnInclude.Builder.create(this, "ExistingInfrastructure")
        .template((ObjectNode)new ObjectMapper().readTree(new File("my-template.json")))
        .build();
```

------
#### [ C\# ]

```
using Newtonsoft.Json.Linq;

new CfnInclude(this, "ExistingInfrastructure", new CfnIncludeProps
{
    Template = JObject.Parse(File.ReadAllText("my-template.json"))
});
```

------

Then to access an attribute of the resource, such as the bucket's ARN, call `Fn.getAtt()` with the logical name of the resource in the AWS CloudFormation template and the name of the resource's attribute\. \(The desired resource and attribute must be defined in the template; `Fn.getAtt()` does not query actual resources you have deployed using the template\.\)

------
#### [ TypeScript ]

```
const bucketArn = cdk.Fn.getAtt("S3Bucket", "Arn");
```

------
#### [ JavaScript ]

```
const bucketArn = cdk.Fn.getAtt("S3Bucket", "Arn");
```

------
#### [ Python ]

```
bucket_arn = cdk.Fn.get_att("S3Bucket", "Arn")
```

------
#### [ Java ]

```
IResolvable bucketArn = Fn.getAtt("S3Bucket", "Arn");
```

------
#### [ C\# ]

```
var bucketArn = Fn.GetAtt("S3Bucket", "Arn");
```

------

The result of a `getAtt()` call is a [token](tokens.md), a type of placeholder\. The actual value of the attribute isn't available until later in the synthesis process\. If you need to pass such an attribute to another API that requires a concrete value, such as a string or a number, use the following static methods of the `[Token](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html)` class to convert the token to a string, number, or list\. 
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-stringvalue-options](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-stringvalue-options) to generate a string encoding \(or call `.toString()` on the token object\)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-listvalue-options](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-listvalue-options) to generate a list encoding
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-numbervalue](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-numbervalue) to generate a numeric encoding

In our example of getting a bucket's ARN, you'd convert it to a string, but that string is still a token, just in a string encoding\. You still don't have the actual ARN\. But in many ways, you can treat the string as if you did have the real value \(for example, adding text to the beginning or end\) and it will work as you expect\.

## Using `cloudformation-include.CfnInclude`<a name="use_cfn_template_cfninclude"></a>

The [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html) construct imports the resources in an AWS CloudFormation template as AWS CDK L1 constructs, allowing you to use them in your app as you would any other AWS CDK construct\. You can then wrap these in L2 constructs to gain their higher\-level abstractions\. This capability can help you migrate an AWS CloudFormation stack to the AWS CDK without making any unexpected changes to the underlying AWS resources, or to add an AWS CDK API "wrapper" to an existing AWS CloudFormation template\.

**Warning**  
This construct is currently in developer preview, which means its API is generally considered stable, but may still undergo breaking changes when necessary\.

 Here is a simple AWS CloudFormation template \(the same one we used in the preceding section, in fact\) which we'll use for the examples in this topic\. Save it as `my-template.json`\. You might also use a template for an actual stack you have already deployed, obtained from the AWS CloudFormation console\.

**Tip**  
You can use either a JSON or YAML template\. We recommend JSON if available, since YAML parsers can vary slightly in what they accept\.

```
{
  "MyBucket": {
    "Type": "AWS::S3::Bucket",
    "Properties": {
      "BucketName": "MyBucket",
    }
  }
}
```

And here's how you might import it into your stack using the new `cloudformation-include `module\.

------
#### [ TypeScript ]

```
import * as cdk from '@aws-cdk/core';
import * as cfn_inc from '@aws-cdk/cloudformation-include';

export class MyStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const template = new cfn_inc.CfnInclude(this, 'Template', { 
      templateFile: 'my-template.json',
    });
  }
}
```

------
#### [ JavaScript ]

```
cost cdk = require('@aws-cdk/core');
const cfn_inc = require('@aws-cdk/cloudformation-include');

class MyStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const template = new cfn_inc.CfnInclude(this, 'Template', { 
      templateFile: 'my-template.json',
    });
  }
}

module.exports = { MyStack }
```

------
#### [ Python ]

```
from aws_cdk import core
from aws_cdk import cloudformation_include as cfn_inc

class MyStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        template = cfn_inc.CfnInclude(self, "Template",  
            template_file="my-template.json")
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.cloudformation.include.CfnInclude;

public class MyStack extends Stack {
    public MyStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        CfnInclude template = CfnInclude.Builder.create(this, "Template")
        	.templateFile("my-template.json")
        	.build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using cfn_inc = Amazon.CDK.CloudFormation.Include;

namespace MyApp
{
    public class MyStack : Stack
    {
        internal MyStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var template = new cfn_inc.CfnInclude(this, "Template", new cfn_inc.CfnIncludeProps
            {
                TemplateFile = "my-template.json"
            });
        }
    }
}
```

------

By default, importing preserves the original logical IDs from the template\. This behavior is suitable for migrating an AWS CloudFormation template to the AWS CDK\. If you are instead developing an AWS CDK construct wrapper for the template so it can be reused elsewhere \("vending"\), have the AWS CDK generate new resource IDs instead, so the construct can be used multiple times in a stack without name conflicts\. To do this, set the `preserveLogicalIds` property to false when importing the template\.

------
#### [ TypeScript ]

```
const template = new cfn_inc.CfnInclude(this, 'MyConstruct', {
  templateFile: 'my-template.json',
  preserveLogicalIds: false
});
```

------
#### [ JavaScript ]

```
const template = new cfn_inc.CfnInclude(this, 'MyConstruct', {
  templateFile: 'my-template.json',
  preserveLogicalIds: false
});
```

------
#### [ Python ]

```
template = cfn_inc.CfnInclude(self, "Template",  
    template_file="my-template.json",
    preserve_logical_ids=False)
```

------
#### [ Java ]

```
CfnInclude template = CfnInclude.Builder.create(this, "Template")
	.templateFile("my-template.json")
	.preserveLogicalIds(false)
	.build();
```

------
#### [ C\# ]

```
var template = new cfn_inc.CfnInclude(this, "Template", new cfn_inc.CfnIncludeProps
{
    TemplateFile = "my-template.json",
    PreserveLogicalIds = false
});
```

------

To put the imported resources under the control of your AWS CDK app, simply add the stack to the `App` as usual\.

------
#### [ TypeScript ]

```
import * as cdk from '@aws-cdk/core';
import { MyStack } from '../lib/my-stack';

const app = new cdk.App();
new MyStack(app, 'MyStack');
```

------
#### [ JavaScript ]

```
const cdk = require('@aws-cdk/core');
const { MyStack } = require('../lib/my-stack');

const app = new cdk.App();
new MyStack(app, 'MyStack');
```

------
#### [ Python ]

```
from aws_cdk import core
from mystack.my_stack import MyStack

app = core.App()
MyStack(app, "MyStack")
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.App;

public class MyApp {
    public static void main(final String[] args) {
        App app = new App();

        new MyStack(app, "MyStack");
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;

namespace CdkApp
{
    sealed class Program
    {
        public static void Main(string[] args)
        {
            var app = new App();
            new MyStack(app, "MyStack");
        }
    }
}
```

------

To verify that there will be no unintended changes to the AWS resources in the stack, perform a diff, omitting the AWS CDK\-specific metadata\.

```
cdk diff --no-version-reporting --no-path-metadata --no-asset-metadata
```

When you `cdk deploy` the stack, your AWS CDK app becomes the source of truth for the stack\. Going forward, make changes to the AWS CDK app, not to the AWS CloudFormation template\.

### Accessing imported resources<a name="use_cfn_template_cfninclude_access"></a>

The name `template` in the example code represents the included AWS CloudFormation template\. To access a resource from it, use this object's [https://docs.aws.amazon.com/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-resourcelogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-resourcelogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) method\. To access the returned resource as a specific kind of resource, cast the result to the desired type \(except in Python and JavaScript, where types are not enforced\)\.

------
#### [ TypeScript ]

```
const cfnBucket = template.getResource('MyBucket') as s3.CfnBucket;
```

------
#### [ JavaScript ]

```
const cfnBucket = template.getResource('MyBucket') as s3.CfnBucket;
```

------
#### [ Python ]

```
cfn_bucket = template.get_resource("MyBucket")
```

------
#### [ Java ]

```
CfnBucket cfnBucket = (CfnBucket)template.getResource("MyBucket");
```

------
#### [ C\# ]

```
var cfnBucket = (CfnBucket)template.GetResource("MyBucket");
```

------

In our example, `cfnBucket` is now an instance of the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.CfnBucket.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.CfnBucket.html) class, a L1 construct that exactly represents the corresponding AWS CloudFormation resource\. You can treat it like any other resource of its type, for example getting its ARN by way of the `bucket.attrArn` property\.

To wrap the L1 `CfnBucket` resource in a L2 [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) instance instead, use the static methods [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-arnscope-id-bucketarn](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-arnscope-id-bucketarn), [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-attributesscope-id-attrs](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-attributesscope-id-attrs), or [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-namescope-id-bucketname](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#static-from-wbr-bucket-wbr-namescope-id-bucketname)\. Usually the `fromBucketName()` method is the most convenient\. For example:

------
#### [ TypeScript ]

```
const bucket = s3.Bucket.fromBucketName(this, 'Bucket', cfnBucket.ref);
```

------
#### [ JavaScript ]

```
const bucket = s3.Bucket.fromBucketName(this, 'Bucket', cfnBucket.ref);
```

------
#### [ Python ]

```
bucket = s3.Bucket.from_bucket_name(this, "Bucket", cfn_bucket.ref)
```

------
#### [ Java ]

```
Bucket bucket = (Bucket)Bucket.fromBucketName(this, "Bucket", cfnBucket.getRef());
```

------
#### [ C\# ]

```
var bucket = (Bucket)Bucket.FromBucketName(this, "Bucket", cfnBucket.Ref);
```

------

Other L2 constructs have similar methods for creating the construct from an existing resource\.

Constructing the `Bucket` this way doesn't create a second Amazon S3 bucket; instead, the new `Bucket` instance encapsulates the existing `CfnBucket`\.

In the example, `bucket` is now an L2 `Bucket` construct that you can use as you would one you declared yourself\. For example, if `lambdaFunc` is an AWS Lambda function, and you wish to grant it write access to the bucket, you can do so using the bucket's convenient [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-wbr-read-wbr-writeidentity-objectskeypattern](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-wbr-read-wbr-writeidentity-objectskeypattern) method, without needing to construct the necessary IAM policy yourself\.

------
#### [ TypeScript ]

```
bucket.grantWrite(lambdaFunc);
```

------
#### [ JavaScript ]

```
bucket.grantWrite(lambdaFunc);
```

------
#### [ Python ]

```
bucket.grant_write(lambda_func)
```

------
#### [ Java ]

```
bucket.grantWrite(lambdaFunc);
```

------
#### [ C\# ]

```
bucket.GrantWrite(lambdaFunc);
```

------

### Replacing parameters<a name="use_cfn_template_cfninclude_params"></a>

If your included AWS CloudFormation template has parameters, you can replace these with build\-time values when you import the template, using the `parameters` property\. In the example below, we replace the `UploadBucket` parameter with the ARN of a bucket defined elsewhere in our AWS CDK code\.

------
#### [ TypeScript ]

```
const template = new cfn_inc.CfnInclude(this, 'Template', {
  templateFile: 'my-template.json',
  parameters: {
    'UploadBucket': bucket.bucketArn,
  },
});
```

------
#### [ JavaScript ]

```
const template = new cfn_inc.CfnInclude(this, 'Template', {
  templateFile: 'my-template.json',
  parameters: {
    'UploadBucket': bucket.bucketArn,
  },
});
```

------
#### [ Python ]

```
template = cfn_inc.CfnInclude(self, "Template",  
    template_file="my-template.json",
    parameters=dict(UploadBucket=bucket.bucket_arn)
)
```

------
#### [ Java ]

```
CfnInclude template = CfnInclude.Builder.create(this, "Template")
	.templateFile("my-template.json")
	.parameters(new HashMap<String, String>() {{
			put("UploadBucket", bucket.getBucketArn());
	}})
	.build();
```

------
#### [ C\# ]

```
var template = new cfn_inc.CfnInclude(this, "Template", new cfn_inc.CfnIncludeProps
{
    TemplateFile = "my-template.json",
    Parameters = new Dictionary<string, string>
    {
        { "UploadBucket", bucket.BucketArn }
    }
});
```

------

### Other template elements<a name="use_cfn_template_cfninclude_other"></a>

You can import any AWS CloudFormation template element, not just resources\. The imported elements become part of the AWS CDK stack\. To import these elements, use the following methods of the `CfnInclude` object\.
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-conditionconditionname-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-conditionconditionname-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [conditions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-hookhooklogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-hookhooklogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [hooks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/blue-green.html) for blue\-green deployments
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-mappingmappingname-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-mappingmappingname-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [mappings](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-outputlogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-outputlogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [outputs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-parameterparametername-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-parameterparametername-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-rulerulename-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-rulerulename-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) \- AWS CloudFormation [rules](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/reference-template_constraint_rules.html) for Service Catalog templates

Each of these methods returns an instance of a class representing the specific type of AWS CloudFormation element\. These objects are mutable; changes will appear in the template generated from the AWS CDK stack\. The code below, for example, imports a parameter from the template and modifies its default\.

------
#### [ TypeScript ]

```
const param = template.getParameter('MyParameter');
param.default = "AWS CDK"
```

------
#### [ JavaScript ]

```
const param = template.getParameter('MyParameter');
param.default = "AWS CDK"
```

------
#### [ Python ]

```
param = template.get_parameter("MyParameter")
param.default = "AWS CDK"
```

------
#### [ Java ]

```
CfnParameter param = template.getParameter("MyParameter");
param.setDefaultValue("AWS CDK")
```

------
#### [ C\# ]

```
var cfnBucket = (CfnBucket)template.GetResource("MyBucket");
var param = template.GetParameter("MyParameter");
param.Default = "AWS CDK";
```

------

### Nested stacks<a name="use_cfn_template_cfninclude_nested"></a>

You may import [nested stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html) by specifying them either when you import their main template, or at some later point\. The nested template must be stored in a local file, but referenced in as a `NestedStack` resource in the main template, and the resource name used in the AWS CDK code must match the name used for the nested stack in the main template\.

Given this resource definition in the main template, the following code shows how to import the referenced nested stack both ways\.

```
"NestedStack": {
  "Type": "AWS::CloudFormation::Stack",
  "Properties": {
    "TemplateURL": "https://my-s3-template-source.s3.amazonaws.com/nested-stack.json"
  }
```

------
#### [ TypeScript ]

```
// include nested stack when importing main stack
const mainTemplate = new cfn_inc.CfnInclude(this, 'MainStack', {
  templateFile: 'main-template.json',
  loadNestedStacks: {
    'NestedStack': {
      templateFile: 'nested-template.json',
    },
  },
});

// or add it some time after importing the main stack
const nestedTemplate = mainTemplate.loadNestedStack('NestedTemplate', {
  templateFile: 'nested-template.json',
});
```

------
#### [ JavaScript ]

```
// include nested stack when importing main stack
const mainTemplate = new cfn_inc.CfnInclude(this, 'MainStack', {
  templateFile: 'main-template.json',
  loadNestedStacks: {
    'NestedStack': {
      templateFile: 'nested-template.json',
    },
  },
});

// or add it some time after importing the main stack
const nestedTemplate = mainTemplate.loadNestedStack('NestedStack', {
  templateFile: 'my-nested-template.json',
});
```

------
#### [ Python ]

```
# include nested stack when importing main stack
main_template = cfn_inc.CfnInclude(self, "MainStack",  
    template_file="main-template.json",
    load_nested_stacks=dict(NestedStack=
        cfn_inc.CfnIncludeProps(template_file="nested-template.json")))

# or add it some time after importing the main stack
nested_template = main_template.load_nested_stack("NestedStack", 
    template_file="nested-template.json")
```

------
#### [ Java ]

```
CfnInclude mainTemplate = CfnInclude.Builder.create(this, "MainStack")
	.templateFile("main-template.json")
	.loadNestedStacks(new HashMap<String, CfnIncludeProps>() {{
		put("NestedStack", CfnIncludeProps.builder()
				.templateFile("nested-template.json").build());
	}})
	.build();

// or add it some time after importing the main stack
IncludedNestedStack nestedTemplate = mainTemplate.loadNestedStack("NestedTemplate", CfnIncludeProps.builder()
		.templateFile("nested-template.json")
		.build());
```

------
#### [ C\# ]

```
// include nested stack when importing main stack
var mainTemplate = new cfn_inc.CfnInclude(this, "MainStack", new cfn_inc.CfnIncludeProps
{
    TemplateFile = "main-template.json",
    LoadNestedStacks = new Dictionary<string, cfn_inc.ICfnIncludeProps>
    {
        { "NestedStack", new cfn_inc.CfnIncludeProps { TemplateFile = "nested-template.json" } }
    }
});

// or add it some time after importing the main stack
var nestedTemplate = mainTemplate.LoadNestedStack("NestedTemplate", new cfn_inc.CfnIncludeProps {
    TemplateFile = 'nested-template.json'
});
```

------

You can import multiple nested stacks with either or both methods\. When importing the main template, you provide a mapping between the resource name of each nested stack and its template file, and this mapping can contain any number of entries\. To do it after the initial import, call `loadNestedStack()` for each nested stack\.

After importing a nested stack, you can access it using the main template's [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-nested-wbr-stacklogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.CfnInclude.html#get-wbr-nested-wbr-stacklogicalid-span-class-api-icon-api-icon-experimental-title-this-api-element-is-experimental-it-may-change-without-notice-span) method\.

------
#### [ TypeScript ]

```
const nestedStack = mainTemplate.getNestedStack('NestedStack').stack;
```

------
#### [ JavaScript ]

```
const nestedStack = mainTemplate.getNestedStack('NestedStack').stack;
```

------
#### [ Python ]

```
nested_stack = main_template.get_nested_stack("NestedStack").stack
```

------
#### [ Java ]

```
NestedStack nestedStack = mainTemplate.getNestedStack("NestedStack").getStack();
```

------
#### [ C\# ]

```
var nestedStack = mainTemplate.GetNestedStack("NestedStack").Stack;
```

------

The `getNestedStack()` method returns an [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.IncludedNestedStack.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_cloudformation-include.IncludedNestedStack.html) instance, from which you can access the AWS CDK [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.NestedStack.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.NestedStack.html) instance via the `stack` property \(as shown in the example\) or the original AWS CloudFormation template object via `includedTemplate`, from which you can load resources and other AWS CloudFormation elements\.