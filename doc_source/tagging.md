# Tagging<a name="tagging"></a>

The AWS CDK includes two classes that you can use to create and delete tags:
+  [Tag](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Tag.html) applies a new tag to a construct and all of its children\. 
+  [RemoveTag](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.RemoveTag.html) removes a tag from a construct and any of its children, including tags a child construct may have applied to itself\. 

Let's look at a couple of examples\.

The following example applies the tag **key** with the value **value** to `myConstruct`\.

```
myConstruct.node.applyAspect(new Tag('key', 'value'));
```

The following example deletes the tag **key** from `myConstruct`\.

```
myConstruct.node.applyAspect(new RemoveTag('key'));
```

The AWS CDK applies and removes tags recursively\. If there are conflicts, the tagging operation with the highest priority wins\. If the priorities are the same, the tagging operation closest to the bottom of the construct tree wins\. By default, applying a tag has a priority of 100 and removing a tag has a priority of 200\. To change the priority of applying a tag, pass a `priority` option to `Tag` or `RemoveTag`\. 

The following applies a tag with a priority of 300 to `myConstruct`\.

```
myConstruct.node.applyAspect(new Tag('key', 'value', {
  priority: 300
}));
```

## Tag<a name="tagging_tag"></a>

The `Tag` operation includes some properties to fine\-tune how tags are applied from the resources that the construct creates, all of which are optional\.

The following example applies the tag **tagname** with the value **value** and priority **100** to resources of type **AWS::Xxx::Yzz** in `myConstruct`, but not to instances launched in an Amazon EC2 Auto Scaling group or to resources of type **AWS::Xxx::Zss**\.

```
myConstruct.node.applyAspect(new Tag('tagname', 'value', {
  applyToLaunchedInstances: false,
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 100,
}));
```

These properties have the following meanings\.

applyToLaunchedInstances  
By default, tags are applied to instances launched in an Auto Scaling group\. Set this property to **false** to not apply tags to instances launched in an Auto Scaling group\.

includeResourceTypes/excludeResourceTypes  
Use these to apply tags only to a subset of resources, based on AWS CloudFormation resource types\. By default, the tag is applied to all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

priority  
Use this to set the priority of this operation with respect to other `Tag` and `RemoveTag` operations\. Higher values take precedence over lower values\. The default is 100\.

## RemoveTag<a name="tagging_removetag"></a>

The `RemoveTag` class includes properties to fine\-tune how tags are removed, all of which are optional\.

The following example removes the tag **tagname** with priority **200** from resources of type **AWS::Xxx::Yzz** in `myConstruct`, but not to resources of type **AWS::Xxx::Zzz**\.

```
myConstruct.node.applyAspect(new RemoveTag('tagname', {
  includeResourceTypes: ['AWS::Xxx::Yyy'],
  excludeResourceTypes: ['AWS::Xxx::Zzz'],
  priority: 200,
}));
```

These properties have the following meanings\.

includeResourceTypes/excludeResourceTypes  
Use these properties to remove tags only from subset of resources based on AWS CloudFormation resource types\. By default, the tag is removed from all resources in the construct subtree, but this can be changed by including or excluding certain resource types\. Exclude takes precedence over include, if both are specified\.

priority  
Use this property to specify the priority of this operation with respect to other `Tag` and `RemoveTag` operations\. Higher values take precedence over lower values\. The default is 200\.

## Example<a name="tagging_example"></a>

The following example adds the tag key **StackType** with value **TheBest** to any resource created within the Stack named `MarketingSystem`\. Then it removes it again from all resources except Amazon EC2 VPC subnets\. The result is that only the subnets have the tag applied\.

```
import { App, RemoveTag, Stack, Tag } from require('@aws-cdk/cdk');

const app = new App();
const theBestStack = new Stack(app, 'MarketingSystem');

// Add a tag to all constructs in the stack
theBestStack.node.applyAspect(new Tag('StackType', 'TheBest'));

// Remove the tag from all resources except subnet resources
theBestStack.node.applyAspect(new RemoveTag('StackType'), {
  exludeResourceTypes: ['AWS::EC2::Subnet']
}));
```

**Note**  
You can achieve the same result by using the following\.  

```
theBestStack.node.applyAspect(new Tag('StackType', 'TheBest', { includeResourceTypes: [‘AWS::EC2::Subnet’]})); 
```