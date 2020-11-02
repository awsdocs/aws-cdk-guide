# Tagging<a name="tagging"></a>

Tags are informational key\-value elements that you can add to constructs in your AWS CDK app\. A tag applied to a given construct also applies to all of its taggable children\. The [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tags.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tags.html) class includes the static method `of()`, through which you can add tags to, or remove tags from, the specified construct\. 
+  [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-addscope-key-value-props](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-addscope-key-value-props) applies a new tag to the given construct and all of its children\. 
+  [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-removescope-key-props](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-removescope-key-props) removes a tag from the given construct and any of its children, including tags a child construct may have applied to itself\. 

**Note**  
Tagging is implemented using [Aspects](aspects.md)\. Aspects are a way to apply an operation \(such as tagging\) to all constructs in a given scope\.

The following example applies the tag **key** with the value **value** to a construct\.

------
#### [ TypeScript ]

```
Tags.of(myConstruct).add('key', 'value');
```

------
#### [ JavaScript ]

```
Tags.of(myConstruct).add('key', 'value');
```

------
#### [ Python ]

```
Tags.of(my_construct).add("key", "value")
```

------
#### [ Java ]

```
Tags.of(myConstruct).add("key", "value");
```

------
#### [ C\# ]

```
Tags.Of(myConstruct).Add("key", "value");
```

------

The following example deletes the tag **key** from a construct\.

------
#### [ TypeScript ]

```
Tags.of(myConstruct).remove('key');
```

------
#### [ JavaScript ]

```
Tags.of(myConstruct).remove('key');
```

------
#### [ Python ]

```
Tags.of(my_construct).remove("key")
```

------
#### [ Java ]

```
Tags.of(myConstruct).remove("key");
```

------
#### [ C\# ]

```
Tags.Of(myConstruct).Remove("key");
```

------

## Tag priorities<a name="w338aac15c23c17"></a>

The AWS CDK applies and removes tags recursively\. If there are conflicts, the tagging operation with the highest priority wins\. \(Priorities are set using the optional `priority` property\.\) If the priorities of two operations are the same, the tagging operation closest to the bottom of the construct tree wins\. By default, applying a tag has a priority of 100 \(except for tags added directly to an AWS CloudFormation resource, which has a priority of 50\) and removing a tag has a priority of 200\. 

The following applies a tag with a priority of 300 to a construct\.

------
#### [ TypeScript ]

```
Tags.of(myConstruct).add('key', 'value', {
  priority: 300
});
```

------
#### [ JavaScript ]

```
Tags.of(myConstruct).add('key', 'value', {
  priority: 300
});
```

------
#### [ Python ]

```
Tags.of(my_construct).add("key", "value", priority=300)
```

------
#### [ Java ]

```
Tags.of(myConstruct).add("key", "value", TagProps.builder()
        .priority(300).build());
```

------
#### [ C\# ]

```
Tags.Of(myConstruct).Add("key", "value", new TagProps { Priority = 300 });
```

------

## Optional properties<a name="tagging_props"></a>

Tags support [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.TagProps.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.TagProps.html) that fine\-tune how tags are applied to, or removed from, resources\. All properties are optional\.

`applyToLaunchedInstances` \(Python: `apply_to_launched_instances`\)  
Available for add\(\) only\. By default, tags are applied to instances launched in an Auto Scaling group\. Set this property to **false** to ignore instances launched in an Auto Scaling group\.

`includeResourceTypes`/`excludeResourceTypes` \(Python: `include_resource_types`/`exclude_resource_types`\)  
Use these to manipulate tags only on a subset of resources, based on AWS CloudFormation resource types\. By default, the operation is applied to all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

`priority`  
Use this to set the priority of this operation with respect to other `Tags.add()` and `Tags.remove()` operations\. Higher values take precedence over lower values\. The default is 100 for add operations \(50 for tags applied directly to AWS CloudFormation resources\) and 200 for remove operations\.

The following example applies the tag **tagname** with the value **value** and priority **100** to resources of type **AWS::Xxx::Yyy** in the construct, but not to instances launched in an Amazon EC2 Auto Scaling group or to resources of type **AWS::Xxx::Zzz**\. \(These are placeholders for two arbitrary but different AWS CloudFormation resource types\.\)

------
#### [ TypeScript ]

```
Tags.of(myConstruct).add('tagname', 'value', {
  applyToLaunchedInstances: false,
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 100,
});
```

------
#### [ JavaScript ]

```
Tags.of(myConstruct).add('tagname', 'value', {
  applyToLaunchedInstances: false,
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 100
});
```

------
#### [ Python ]

```
Tags.of((my_construct).add("tagname", "value",
    apply_to_launched_instances=False,
    include_resource_types=["AWS::Xxx::Yyy"],
    exclude_resource_types=["AWS::Xxx::Zzz"],
    priority=100,)
```

------
#### [ Java ]

```
Tags.of(myConstruct).add("key", "value", TagProps.builder()
                .applyToLaunchedInstances(false)
                .includeResourceTypes(Arrays.asList("AWS::Xxx::Yyy"))
                .excludeResourceTypes(Arrays.asList("AWS::Xxx::Zzz"))
                .priority(100).build());
```

------
#### [ C\# ]

```
Tags.Of(myConstruct).Add("tagname", "value", new TagProps
{
    ApplyToLaunchedInstances = false,
    IncludeResourceTypes = ["AWS::Xxx::Yyy"],
    ExcludeResourceTypes = ["AWS::Xxx::Zzz"],
    Priority = 100
});
```

------

The following example removes the tag **tagname** with priority **200** from resources of type **AWS::Xxx::Yyy** in the construct, but not from resources of type **AWS::Xxx::Zzz**\.

------
#### [ TypeScript ]

```
Tags.of(myConstruct).remove('tagname', {
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 200,
});
```

------
#### [ JavaScript ]

```
Tags.of(myConstruct).remove('tagname', {
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 200
});
```

------
#### [ Python ]

```
Tags.rof(my_construct).remove("tagname",
    include_resource_types=["AWS::Xxx::Yyy"],
    exclude_resource_types=["AWS::Xxx::Zzz"],
    priority=200,)
```

------
#### [ Java ]

```
Tags.of((myConstruct).remove("tagname", TagProps.builder()
        .includeResourceTypes(Arrays.asList("AWS::Xxx::Yyy"))
        .excludeResourceTypes(Arrays.asList("AWS::Xxx::Zzz"))
        .priority(100).build());
```

------
#### [ C\# ]

```
Tags.Of(myConstruct).Remove("tagname", new TagProps
{
    IncludeResourceTypes = ["AWS::Xxx::Yyy"],
    ExcludeResourceTypes = ["AWS::Xxx::Zzz"],
    Priority = 100
});
```

------

## Example<a name="tagging_example"></a>

The following example adds the tag key **StackType** with value **TheBest** to any resource created within the Stack named `MarketingSystem`\. Then it removes it again from all resources except Amazon EC2 VPC subnets\. The result is that only the subnets have the tag applied\.

------
#### [ TypeScript ]

```
import { App, Stack, Tags } from '@aws-cdk/core';

const app = new App();
const theBestStack = new Stack(app, 'MarketingSystem');

// Add a tag to all constructs in the stack
Tags.of(theBestStack).add('StackType', 'TheBest');

// Remove the tag from all resources except subnet resources
Tags.of(theBestStack).remove('StackType', {
  excludeResourceTypes: ['AWS::EC2::Subnet']
});
```

------
#### [ JavaScript ]

```
const { App , Stack , Tag } = require('@aws-cdk/core');

const app = new App();
const theBestStack = new Stack(app, 'MarketingSystem');

// Add a tag to all constructs in the stack
Tags.of(theBestStack).add('StackType', 'TheBest');

// Remove the tag from all resources except subnet resources
Tags.of(theBestStack).remove'StackType', {
  excludeResourceTypes: ['AWS::EC2::Subnet']
});
```

------
#### [ Python ]

```
from aws_cdk.core import App, Stack, Tag

app = App();
the_best_stack = Stack(app, 'MarketingSystem')

# Add a tag to all constructs in the stack
Tags.of(the_best_stack).add("StackType", "TheBest")

# Remove the tag from all resources except subnet resources
Tags.of(the_best_stack).remove("StackType", 
    exclude_resource_types=["AWS::EC2::Subnet"])
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Tag;

// Add a tag to all constructs in the stack
Tags.of(theBestStack).add("StackType", "TheBest");

// Remove the tag from all resources except subnet resources
Tags.of(theBestStack).remove("StackType", TagProps.builder()
        .excludeResourceTypes(Arrays.asList("AWS::EC2::Subnet"))
        .build());
```

------
#### [ C\# ]

```
using Amazon.CDK;

var app = new App();
var theBestStack = new Stack(app, 'MarketingSystem');

// Add a tag to all constructs in the stack
Tags.Of(theBestStack).Add("StackType", "TheBest");

// Remove the tag from all resources except subnet resources
Tags.Of(theBestStack).Remove("StackType", new TagProps 
{
    ExcludeResourceTypes = ["AWS::EC2::Subnet"]
});
```

------

The following code achieves the same result\. Consider which approach \(inclusion or exclusion\) makes your intent clearer\.

------
#### [ TypeScript ]

```
Tags.of(theBestStack).add('StackType', 'TheBest', 
  { includeResourceTypes: ['AWS::EC2::Subnet']});
```

------
#### [ JavaScript ]

```
Tags.of(theBestStack).add('StackType', 'TheBest', 
  { includeResourceTypes: ['AWS::EC2::Subnet']});
```

------
#### [ Python ]

```
Tags.of(the_best_stack).add("StackType", "TheBest", 
    include_resource_types=["AWS::EC2::Subnet"])
```

------
#### [ Java ]

```
Tags.of(theBestStack).add("StackType", "TheBest", TagProps.builder()
        .includeResourceTypes(Arrays.asList("AWS::EC2::Subnet"))
        .build());
```

------
#### [ C\# ]

```
Tags.Of(theBestStack).Add("StackType", "TheBest", new TagProps {
    IncludeResourceTypes = ["AWS::EC2::Subnet"]
});
```

------