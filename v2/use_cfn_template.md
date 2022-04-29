# Import or migrate an existing AWS CloudFormation template<a name="use_cfn_template"></a>

The [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include-readme.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include-readme.html) construct converts the resources in an imported AWS CloudFormation template to AWS CDK L1 constructs\. You can work with these in your app just as if they were defined in AWS CDK code, even using them within higher\-level AWS CDK constructs, letting you use \(for example\) the L2 permission grant methods with the resources they define\. 

This construct essentially adds an AWS CDK API wrapper to any resource in the template\. You can use this capability to migrate your existing AWS CloudFormation templates to the AWS CDK a piece at a time in order to take advantage of the AWS CDK's convenient higher\-level abstractions, or just to vend your AWS CloudFormation templates to AWS CDK developers by providing an AWS CDK construct API\.

**Note**  
AWS CDK v1 also included [https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cdk-lib.CfnInclude.html](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cdk-lib.CfnInclude.html), which was previously used for the same general purpose\. However, it lacks much of the functionality of `cloudformation-include.CfnInclude`\.

## Importing an AWS CloudFormation template<a name="w317aac29b9b9"></a>

 Here is a simple AWS CloudFormation template we'll use for the examples in this topic\. Save it as `my-template.json`\. After you've tried these examples with the provided template, you might explore further using a template for an actual stack you've already deployed, which you can obtain from the AWS CloudFormation console\.

**Tip**  
You can use either a JSON or YAML template\. We recommend JSON if available, since YAML parsers can vary slightly in what they accept\.

```
{
  "Resources": {
    "MyBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "MyBucket",
      }
    }
  }
}
```

And here's how you import it into your stack using `cloudformation-include`\.

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import * as cfninc from 'aws-cdk-lib/cloudformation-include';
import { Construct } from 'constructs';

export class MyStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const template = new cfninc.CfnInclude(this, 'Template', { 
      templateFile: 'my-template.json',
    });
  }
}
```

------
#### [ JavaScript ]

```
const cdk = require('aws-cdk-lib');
const cfninc = require('aws-cdk-lib/cloudformation-include');

class MyStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const template = new cfninc.CfnInclude(this, 'Template', { 
      templateFile: 'my-template.json',
    });
  }
}

module.exports = { MyStack }
```

------
#### [ Python ]

```
import aws_cdk as cdk
from aws_cdk import cloudformation_include as cfn_inc
from constructs import Construct

class MyStack(cdk.Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        template = cfn_inc.CfnInclude(self, "Template",  
            template_file="my-template.json")
```

------
#### [ Java ]

```
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.cloudformation.include.CfnInclude;
import software.constructs.Construct;

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
using Constructs;
using cfnInc = Amazon.CDK.CloudFormation.Include;

namespace MyApp
{
    public class MyStack : Stack
    {
        internal MyStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var template = new cfnInc.CfnInclude(this, "Template", new cfnInc.CfnIncludeProps
            {
                TemplateFile = "my-template.json"
            });
        }
    }
}
```

------

By default, importing a resource preserves the resource's original logical ID from the template\. This behavior is suitable for migrating an AWS CloudFormation template to the AWS CDK, where the logical IDs must be retained for AWS CloudFormation to recognize these as the same resources from the AWS CloudFormation template\.

If you are instead developing an AWS CDK construct wrapper for the template so it can be used by AWS CDK developers \("vending"\), have the AWS CDK generate new resource IDs instead, so the construct can be used multiple times in a stack without name conflicts\. To do this, set the `preserveLogicalIds` property to false when importing the template\.

------
#### [ TypeScript ]

```
const template = new cfninc.CfnInclude(this, 'MyConstruct', {
  templateFile: 'my-template.json',
  preserveLogicalIds: false
});
```

------
#### [ JavaScript ]

```
const template = new cfninc.CfnInclude(this, 'MyConstruct', {
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
var template = new cfnInc.CfnInclude(this, "Template", new cfn_inc.CfnIncludeProps
{
    TemplateFile = "my-template.json",
    PreserveLogicalIds = false
});
```

------

To put the imported resources under the control of your AWS CDK app, add the stack to the `App` as usual\.

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import { MyStack } from '../lib/my-stack';

const app = new cdk.App();
new MyStack(app, 'MyStack');
```

------
#### [ JavaScript ]

```
const cdk = require('aws-cdk-lib');
const { MyStack } = require('../lib/my-stack');

const app = new cdk.App();
new MyStack(app, 'MyStack');
```

------
#### [ Python ]

```
import aws_cdk as cdk
from mystack.my_stack import MyStack

app = cdk.App()
MyStack(app, "MyStack")
```

------
#### [ Java ]

```
import software.amazon.awscdk.App;

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

## Accessing imported resources<a name="use_cfn_template_cfninclude_access"></a>

The name `template` in the example code represents the imported AWS CloudFormation template\. To access a resource from it, use this object's [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrresourcelogicalid](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrresourcelogicalid) method\. To access the returned resource as a specific kind of resource, cast the result to the desired type\. \(Casting is not necessary in Python and JavaScript\.\)

------
#### [ TypeScript ]

```
const cfnBucket = template.getResource('MyBucket') as s3.CfnBucket;
```

------
#### [ JavaScript ]

```
const cfnBucket = template.getResource('MyBucket');
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

In our example, `cfnBucket` is now an instance of the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) class, a L1 construct that exactly represents the corresponding AWS CloudFormation resource\. You can treat it like any other resource of its type, for example getting its ARN by way of the `bucket.attrArn` property\.

To wrap the L1 `CfnBucket` resource in a L2 [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) instance instead, use the static methods [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrarnscope-id-bucketarn](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrarnscope-id-bucketarn), [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrattributesscope-id-attrs](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrattributesscope-id-attrs), or [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrnamescope-id-bucketname](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#static-fromwbrbucketwbrnamescope-id-bucketname)\. Usually the `fromBucketName()` method is the most convenient\. For example:

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
bucket = s3.Bucket.from_bucket_name(self, "Bucket", cfn_bucket.ref)
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

In the example, `bucket` is now an L2 `Bucket` construct that you can use as you would one you declared yourself\. For example, if `lambdaFunc` is an AWS Lambda function, and you wish to grant it write access to the bucket, you can do so using the bucket's convenient [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#grantwbrwriteidentity-objectskeypattern](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#grantwbrwriteidentity-objectskeypattern) method, without needing to construct the necessary IAM policy yourself\.

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

## Replacing parameters<a name="use_cfn_template_cfninclude_params"></a>

If your included AWS CloudFormation template has parameters, you can replace these with build\-time values when you import the template, using the `parameters` property\. In the example below, we replace the `UploadBucket` parameter with the ARN of a bucket defined elsewhere in our AWS CDK code\.

------
#### [ TypeScript ]

```
const template = new cfninc.CfnInclude(this, 'Template', {
  templateFile: 'my-template.json',
  parameters: {
    'UploadBucket': bucket.bucketArn,
  },
});
```

------
#### [ JavaScript ]

```
const template = new cfninc.CfnInclude(this, 'Template', {
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
	.parameters(java.util.Map.of(    // Map.of requires Java 9+
			"UploadBucket", bucket.getBucketArn()))
	.build();
```

------
#### [ C\# ]

```
var template = new cfnInc.CfnInclude(this, "Template", new cfnInc.CfnIncludeProps
{
    TemplateFile = "my-template.json",
    Parameters = new Dictionary<string, string>
    {
        { "UploadBucket", bucket.BucketArn }
    }
});
```

------

## Other template elements<a name="use_cfn_template_cfninclude_other"></a>

You can import any AWS CloudFormation template element, not just resources\. The imported elements become part of the AWS CDK stack\. To import these elements, use the following methods of the `CfnInclude` object\.
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrconditionconditionname](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrconditionconditionname) \- AWS CloudFormation [conditions](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/conditions-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrhookhooklogicalid](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrhookhooklogicalid) \- AWS CloudFormation [hooks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/blue-green.html) for blue\-green deployments
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrmappingmappingname](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrmappingmappingname) \- AWS CloudFormation [mappings](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/mappings-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbroutputlogicalid](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbroutputlogicalid) \- AWS CloudFormation [outputs](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrparameterparametername](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrparameterparametername) \- AWS CloudFormation [parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html)
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrrulerulename](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrrulerulename) \- AWS CloudFormation [rules](https://docs.aws.amazon.com/servicecatalog/latest/adminguide/reference-template_constraint_rules.html) for Service Catalog templates

Each of these methods returns an instance of a class representing the specific type of AWS CloudFormation element\. These objects are mutable; changes you make to them will appear in the template generated from the AWS CDK stack\. The code below, for example, imports a parameter from the template and modifies its default\.

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

## Nested stacks<a name="use_cfn_template_cfninclude_nested"></a>

You may import [nested stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html) by specifying them either when you import their main template, or at some later point\. The nested template must be stored in a local file, but referenced as a `NestedStack` resource in the main template, and the resource name used in the AWS CDK code must match the name used for the nested stack in the main template\.

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
const mainTemplate = new cfninc.CfnInclude(this, 'MainStack', {
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
const mainTemplate = new cfninc.CfnInclude(this, 'MainStack', {
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
	.loadNestedStacks(java.util.Map.of(    // Map.of requires Java 9+
	   "NestedStack", CfnIncludeProps.builder()
				.templateFile("nested-template.json").build()))
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
var mainTemplate = new cfnInc.CfnInclude(this, "MainStack", new cfnInc.CfnIncludeProps
{
    TemplateFile = "main-template.json",
    LoadNestedStacks = new Dictionary<string, cfnInc.ICfnIncludeProps>
    {
        { "NestedStack", new cfnInc.CfnIncludeProps { TemplateFile = "nested-template.json" } }
    }
});

// or add it some time after importing the main stack
var nestedTemplate = mainTemplate.LoadNestedStack("NestedTemplate", new cfnInc.CfnIncludeProps {
    TemplateFile = 'nested-template.json'
});
```

------

You can import multiple nested stacks with either or both methods\. When importing the main template, you provide a mapping between the resource name of each nested stack and its template file, and this mapping can contain any number of entries\. To do it after the initial import, call `loadNestedStack()` once for each nested stack\.

After importing a nested stack, you can access it using the main template's [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrnestedwbrstacklogicalid](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrnestedwbrstacklogicalid) method\.

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

The `getNestedStack()` method returns an [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrnestedwbrstacklogicalid](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.cloudformation_include.CfnInclude.html#getwbrnestedwbrstacklogicalid) instance, from which you can access the AWS CDK [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.NestedStack.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.NestedStack.html) instance via the `stack` property \(as shown in the example\) or the original AWS CloudFormation template object via `includedTemplate`, from which you can load resources and other AWS CloudFormation elements\.