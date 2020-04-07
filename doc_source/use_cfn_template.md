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

Then to access an attribute of the resource, such as the bucket's ARN:

------
#### [ TypeScript ]

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