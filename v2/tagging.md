# Tagging<a name="tagging"></a>

Tags are informational key\-value elements that you can add to constructs in your AWS CDK app\. A tag applied to a given construct also applies to all of its taggable children\. Tags are included in the AWS CloudFormation template synthesized from your app and are applied to the AWS resources it deploys\. You can use tags to identify and categorize resources for the following purposes:
+ Simplifying management
+ Cost allocation
+ Access control
+ Any other purposes that you devise

**Tip**  
For more information about how you can use tags with your AWS resources, see the whitepaper [Tagging Best Practices](https://d1.awsstatic.com/whitepapers/aws-tagging-best-practices.pdf) \(PDF\)\.

The [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html) class includes the static method `of()`, through which you can add tags to, or remove tags from, the specified construct\. 
+  [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html#addkey-value-props](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html#addkey-value-props) applies a new tag to the given construct and all of its children\. 
+  [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html#removekey-props](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tags.html#removekey-props) removes a tag from the given construct and any of its children, including tags a child construct may have applied to itself\. 

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

If you are using `Stage` constructs, apply the tag at the `Stage` level or below\. Tags are not applied across `Stage` boundaries\.

## Tag priorities<a name="w52aac21c26c25"></a>

The AWS CDK applies and removes tags recursively\. If there are conflicts, the tagging operation with the highest priority wins\. \(Priorities are set using the optional `priority` property\.\) If the priorities of two operations are the same, the tagging operation closest to the bottom of the construct tree wins\. By default, applying a tag has a priority of 100 \(except for tags added directly to an AWS CloudFormation resource, which has a priority of 50\)\. The default priority for removing a tag is 200\. 

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

Tags support [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.TagProps.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.TagProps.html) that fine\-tune how tags are applied to, or removed from, resources\. All properties are optional\.

`applyToLaunchedInstances` \(Python: `apply_to_launched_instances`\)  
Available for add\(\) only\. By default, tags are applied to instances launched in an Auto Scaling group\. Set this property to **false** to ignore instances launched in an Auto Scaling group\.

`includeResourceTypes`/`excludeResourceTypes` \(Python: `include_resource_types`/`exclude_resource_types`\)  
Use these to manipulate tags only on a subset of resources, based on AWS CloudFormation resource types\. By default, the operation is applied to all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

`priority`  
Use this to set the priority of this operation with respect to other `Tags.add()` and `Tags.remove()` operations\. Higher values take precedence over lower values\. The default is 100 for add operations \(50 for tags applied directly to AWS CloudFormation resources\) and 200 for remove operations\.

The following example applies the tag **tagname** with the value **value** and priority **100** to resources of type **AWS::Xxx::Yyy** in the construct\. It doesn't apply the tag to instances launched in an Amazon EC2 Auto Scaling group or to resources of type **AWS::Xxx::Zzz**\. \(These are placeholders for two arbitrary but different AWS CloudFormation resource types\.\)

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
Tags.of(my_construct).add("tagname", "value",
    apply_to_launched_instances=False,
    include_resource_types=["AWS::Xxx::Yyy"],
    exclude_resource_types=["AWS::Xxx::Zzz"],
    priority=100)
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
Tags.of(my_construct).remove("tagname",
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

The following example adds the tag key **StackType** with value **TheBest** to any resource created within the `Stack` named `MarketingSystem`\. Then it removes it again from all resources except Amazon EC2 VPC subnets\. The result is that only the subnets have the tag applied\.

------
#### [ TypeScript ]

```
import { App, Stack, Tags } from 'aws-cdk-lib';

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
const { App, Stack, Tags } = require('aws-cdk-lib');

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
#### [ Python ]

```
from aws_cdk import App, Stack, Tags

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
import software.amazon.awscdk.App;
import software.amazon.awscdk.Tags;

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

## Tagging single constructs<a name="tagging_single"></a>

`Tags.of(scope).add(key, value)` is the standard way to add tags to constructs in the AWS CDK\. Its tree\-walking behavior, which recursively tags all taggable resources under the given scope, is almost always what you want\. Sometimes, however, you need to tag a specific, arbitrary construct \(or constructs\)\.

One such case involves applying tags whose value is derived from some property of the construct being tagged\. The standard tagging approach recursively applies the same key and value to all matching resources in the scope\. However, here the value could be different for each tagged construct\.

Tags are implemented using [aspects](aspects.md), and the CDK calls the tag's `visit()` method for each construct under the scope you specified using `Tags.of(scope)`\. We can call `Tag.visit()` directly to apply a tag to a single construct\.

------
#### [ TypeScript ]

```
new cdk.Tag(key, value).visit(scope);
```

------
#### [ JavaScript ]

```
new cdk.Tag(key, value).visit(scope);
```

------
#### [ Python ]

```
cdk.Tag(key, value).visit(scope)
```

------
#### [ Java ]

```
Tag.Builder.create(key, value).build().visit(scope);
```

------
#### [ C\# ]

```
new Tag(key, value).Visit(scope);
```

------

You can tag all constructs under a scope but let the values of the tags derive from properties of each construct\. To do so, write an aspect and apply the tag in the aspect's `visit()` method as shown in the preceding example\. Then, add the aspect to the desired scope using `Aspects.of(scope).add(aspect)`\.

The following example applies a tag to each resource in a stack containing the resource's path\.

------
#### [ TypeScript ]

```
class PathTagger implements cdk.IAspect {
  visit(node: IConstruct) {
    new cdk.Tag("aws-cdk-path", node.node.path).visit(node);
  }
}
 
stack = new MyStack(app);
cdk.Aspects.of(stack).add(new PathTagger())
```

------
#### [ JavaScript ]

```
class PathTagger {
  visit(node) {
    new cdk.Tag("aws-cdk-path", node.node.path).visit(node);
  }
}

stack = new MyStack(app);
cdk.Aspects.of(stack).add(new PathTagger())
```

------
#### [ Python ]

```
@jsii.implements(cdk.IAspect)
class PathTagger:
    def visit(self, node: IConstruct):
        cdk.Tag("aws-cdk-path", node.node.path).visit(node)

stack = MyStack(app) 
cdk.Aspects.of(stack).add(PathTagger())
```

------
#### [ Java ]

```
final class PathTagger implements IAspect {
	public void visit(IConstruct node) {
		Tag.Builder.create("aws-cdk-path", node.getNode().getPath()).build().visit(node);
	}
}

stack stack = new MyStack(app);
Aspects.of(stack).add(new PathTagger());
```

------
#### [ C\# ]

```
public class PathTagger : IAspect
{
    public void Visit(IConstruct node)
    {
        new Tag("aws-cdk-path", node.Node.Path).Visit(node);
    }
}

var stack = new MyStack(app);
Aspects.Of(stack).Add(new PathTagger);
```

------

**Tip**  
The logic of conditional tagging, including priorities, resource types, and so on, is built into the `Tag` class\. You can use these features when applying tags to arbitrary resources; the tag is not applied if the conditions aren't met\. Also, the `Tag` class only tags taggable resources, so you don't need to test whether a construct is taggable before applying a tag\.