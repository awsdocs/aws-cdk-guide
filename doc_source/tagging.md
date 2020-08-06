# Tagging<a name="tagging"></a>

The `Tag` class includes two methods that you can use to create and delete tags:
+  [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-addscope-key-value-props](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-addscope-key-value-props) applies a new tag to a construct and all of its children\. 
+  [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-removescope-key-props](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html#static-removescope-key-props) removes a tag from a construct and any of its children, including tags a child construct may have applied to itself\. 

**Note**  
Tagging is implemented using [Aspects](aspects.md)\. Aspects are a way to apply an operation \(such as tagging\) to all constructs in a given scope\.

Let's look at a couple of examples\. The following example applies the tag **key** with the value **value** to a construct\.

------
#### [ TypeScript ]

```
Tag.add(myConstruct, 'key', 'value');
```

------
#### [ JavaScript ]

```
Tag.add(myConstruct, 'key', 'value');
```

------
#### [ Python ]

```
Tag.add(my_construct, "key", "value")
```

------
#### [ Java ]

```
Tag.add(myConstruct, "key", "value");
```

------
#### [ C\# ]

```
Tag.Add(myConstruct, "key", "value");
```

------

The following example deletes the tag **key** from a construct\.

------
#### [ TypeScript ]

```
Tag.remove(my_construct, 'key');
```

------
#### [ JavaScript ]

```
Tag.remove(my_construct, 'key');
```

------
#### [ Python ]

```
Tag.remove(my_construct, "key")
```

------
#### [ Java ]

```
Tag.remove(myConstruct, "key");
```

------
#### [ C\# ]

```
Tag.Remove(myConstruct, "key");
```

------

The AWS CDK applies and removes tags recursively\. If there are conflicts, the tagging operation with the highest priority wins\. If the priorities are the same, the tagging operation closest to the bottom of the construct tree wins\. By default, applying a tag has a priority of 100 and removing a tag has a priority of 200\. To change the priority of applying a tag, pass a `priority` property to `Tag.add()` or `Tag.remove()`\. 

The following applies a tag with a priority of 300 to a construct\.

------
#### [ TypeScript ]

```
Tag.add(myConstruct, 'key', 'value', {
  priority: 300
});
```

------
#### [ JavaScript ]

```
Tag.add(myConstruct, 'key', 'value', {
  priority: 300
});
```

------
#### [ Python ]

```
Tag.add(my_construct, "key", "value", priority=300)
```

------
#### [ Java ]

```
Tag.add(myConstruct, "key", "value", TagProps.builder()
        .priority(300).build());
```

------
#### [ C\# ]

```
Tag.Add(myConstruct, "key", "value", new TagProps { Priority = 300 });
```

------

## Tag\.add\(\)<a name="tagging_tag"></a>

`Tag.add()` supports properties that fine\-tune how tags are applied to resources\. All properties are optional\.

The following example applies the tag **tagname** with the value **value** and priority **100** to resources of type **AWS::Xxx::Yyy** in the construct, but not to instances launched in an Amazon EC2 Auto Scaling group or to resources of type **AWS::Xxx::Zzz**\. \(These are placeholders for two arbitrary but different AWS CloudFormation resource types\.\)

------
#### [ TypeScript ]

```
Tag.add(myConstruct, 'tagname', 'value', {
  applyToLaunchedInstances: false,
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 100,
});
```

------
#### [ JavaScript ]

```
Tag.add(myConstruct, 'tagname', 'value', {
  applyToLaunchedInstances: false,
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 100
});
```

------
#### [ Python ]

```
Tag.add(my_construct, "tagname", "value",
    apply_to_launched_instances=False,
    include_resource_types=["AWS::Xxx::Yyy"],
    exclude_resource_types=["AWS::Xxx::Zzz"],
    priority=100,)
```

------
#### [ Java ]

```
Tag.add(myConstruct, "key", "value", TagProps.builder()
                .applyToLaunchedInstances(false)
                .includeResourceTypes(Arrays.asList("AWS::Xxx::Yyy"))
                .excludeResourceTypes(Arrays.asList("AWS::Xxx::Zzz"))
                .priority(100).build());
```

------
#### [ C\# ]

```
Tag.Add(myConstruct, "tagname", "value", new TagProps
{
    ApplyToLaunchedInstances = false,
    IncludeResourceTypes = ["AWS::Xxx::Yyy"],
    ExcludeResourceTypes = ["AWS::Xxx::Zzz"],
    Priority = 100
});
```

------

These properties have the following meanings\.

applyToLaunchedInstances \(Python: apply\_to\_launched\_instances\)  
By default, tags are applied to instances launched in an Auto Scaling group\. Set this property to **false** to not apply tags to instances launched in an Auto Scaling group\.

includeResourceTypes/excludeResourceTypes \(Python: include\_resource\_types, exclude\_resource\_types\)  
 Use these to apply tags only to a subset of resources, based on AWS CloudFormation resource types\. By default, the tag is applied to all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

priority  
Use this to set the priority of this operation with respect to other `Tag.add()` and `Tag.remove()` operations\. Higher values take precedence over lower values\. The default is 100\.

## Tag\.remove\(\)<a name="tagging_removetag"></a>

 `Tag.remove()` supports properties to fine\-tune how tags are removed from resources\. All properties are optional\.

The following example removes the tag **tagname** with priority **200** from resources of type **AWS::Xxx::Yyy** in the construct, but not from resources of type **AWS::Xxx::Zzz**\.

------
#### [ TypeScript ]

```
Tag.remove(myConstruct, 'tagname', {
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 200,
});
```

------
#### [ JavaScript ]

```
Tag.remove(myConstruct, 'tagname', {
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 200
});
```

------
#### [ Python ]

```
Tag.remove(my_construct, "tagname",
    include_resource_types=["AWS::Xxx::Yyy"],
    exclude_resource_types=["AWS::Xxx::Zzz"],
    priority=200,)
```

------
#### [ Java ]

```
Tag.remove(myConstruct, "tagname", TagProps.builder()
        .includeResourceTypes(Arrays.asList("AWS::Xxx::Yyy"))
        .excludeResourceTypes(Arrays.asList("AWS::Xxx::Zzz"))
        .priority(100).build());
```

------
#### [ C\# ]

```
Tag.Remove(myConstruct, "tagname", "value", new TagProps
{
    IncludeResourceTypes = ["AWS::Xxx::Yyy"],
    ExcludeResourceTypes = ["AWS::Xxx::Zzz"],
    Priority = 100
});
```

------

These properties have the following meanings\.

includeResourceTypes/excludeResourceTypes  
\(Python: include\_resource\_types/exclude\_resource\_types\) Use these properties to remove tags only from subset of resources based on AWS CloudFormation resource types\. By default, the tag is removed from all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

priority  
Use this property to specify the priority of this operation with respect to other `Tag.add()` and `Tag.remove()` operations\. Higher values take precedence over lower values\. The default is 200\.

## Example<a name="tagging_example"></a>

The following example adds the tag key **StackType** with value **TheBest** to any resource created within the Stack named `MarketingSystem`\. Then it removes it again from all resources except Amazon EC2 VPC subnets\. The result is that only the subnets have the tag applied\.

------
#### [ TypeScript ]

```
import { App, Stack, Tag } from '@aws-cdk/core';

const app = new App();
const theBestStack = new Stack(app, 'MarketingSystem');

// Add a tag to all constructs in the stack
Tag.add(theBestStack, 'StackType', 'TheBest');

// Remove the tag from all resources except subnet resources
Tag.remove(theBestStack, 'StackType', {
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
Tag.add(theBestStack, 'StackType', 'TheBest');

// Remove the tag from all resources except subnet resources
Tag.remove(theBestStack, 'StackType', {
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
Tag.add(the_best_stack, "StackType", "TheBest")

# Remove the tag from all resources except subnet resources
Tag.remove(the_best_stack, "StackType", 
    exclude_resource_types=["AWS::EC2::Subnet"])
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Tag;

// Add a tag to all constructs in the stack
Tag.add(theBestStack, "StackType", "TheBest");

// Remove the tag from all resources except subnet resources
Tag.remove(theBestStack, "StackType", TagProps.builder()
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
Tag.Add(theBestStack, "StackType", "TheBest");

// Remove the tag from all resources except subnet resources
Tag.Remove(theBestStack, "StackType", new TagProps 
{
    ExcludeResourceTypes = ["AWS::EC2::Subnet"]
});
```

------

The following code achieves the same result\. Consider which approach \(inclusion or exclusion\) makes your intent clearer\.

------
#### [ TypeScript ]

```
Tag.add(theBestStack, 'StackType', 'TheBest', 
  { includeResourceTypes: ['AWS::EC2::Subnet']});
```

------
#### [ JavaScript ]

```
Tag.add(theBestStack, 'StackType', 'TheBest', 
  { includeResourceTypes: ['AWS::EC2::Subnet']});
```

------
#### [ Python ]

```
Tag.add(the_best_stack, "StackType", "TheBest", 
    include_resource_types=["AWS::EC2::Subnet"])
```

------
#### [ Java ]

```
Tag.add(theBestStack, "StackType", "TheBest", TagProps.builder()
        .includeResourceTypes(Arrays.asList("AWS::EC2::Subnet"))
        .build());
```

------
#### [ C\# ]

```
Tag.Add(theBestStack, "StackType", "TheBest", new TagProps {
    IncludeResourceTypes = ["AWS::EC2::Subnet"]
});
```

------