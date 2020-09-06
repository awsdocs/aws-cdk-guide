# Use an existing AWS CloudFormation template<a name="use_cfn_template"></a>

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

Then to access an attribute of the resource, such as the bucket's ARN, call `Fn.getAtt()` with the logical name of the resource in the AWS CloudFormation template and the desired attribute of the resource\. \(The resource must be defined in the template; `Fn.getAtt()` does not query actual resources you have deployed using the template\.

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
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-stringvalue-options](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-stringvalue-options) to generate a string encocding \(or call `.toString()` on the token object\)
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-listvalue-options](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-listvalue-options) to generate a list encoding
+ [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-numbervalue](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Token.html#static-as-numbervalue) to generate a numeric encoding

In our example of getting a bucket's ARN, you'd convert it to a string, but that string is still a token, just in a string encoding\. You still don't have the actual ARN\. But in many ways, you can treat the string as if you did have the real value \(for example, adding text to the beginning or end\) and it will work as you expect\.