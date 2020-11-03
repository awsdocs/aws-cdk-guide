# Aspects<a name="aspects"></a>

Aspects are a way to apply an operation to all constructs in a given scope\. The aspect could modify the constructs, such as by adding tags, or it could verify something about the state of the constructs, such as ensuring that all buckets are encrypted\.

To apply an aspect to a construct and all constructs in the same scope, call [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Aspects.html#static-ofscope](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Aspects.html#static-ofscope)`.of(SCOPE).add()` with a new aspect, as shown in the following example\.

------
#### [ TypeScript ]

```
Aspects.of(myConstruct).add(new SomeAspect(...));
```

------
#### [ JavaScript ]

```
Aspects.of(myConstruct).add(new SomeAspect(...));
```

------
#### [ Python ]

```
Aspects.of(my_construct).add(SomeAspect(...))
```

------
#### [ Java ]

```
Aspects.of(myConstruct).add(new SomeAspect(...));
```

------
#### [ C\# ]

```
Aspects.Of(myConstruct).add(new SomeAspect(...));
```

------

The AWS CDK currently uses aspects only to [tag resources](tagging.md), but the framework is extensible and can also be used for other purposes\. For example, you can use it to validate or change the AWS CloudFormation resources that are defined for you by higher\-level constructs\.

## Aspects in detail<a name="aspects_detail"></a>

Aspects employ the [visitor pattern](https://en.wikipedia.org/wiki/Visitor_pattern)\. An aspect is a class that implements the following interface\.

------
#### [ TypeScript ]

```
interface IAspect {
   visit(node: IConstruct): void;}
```

------
#### [ JavaScript ]

JavaScript doesn't have interfaces as a language feature, so an aspect is simply an instance of a class having a `visit` method that accepts the node to be operated on\.

------
#### [ Python ]

Python doesn't have interfaces as a language feature, so an aspect is simply an instance of a class having a `visit` method that accepts the node to be operated on\.

------
#### [ Java ]

```
public interface IAspect {
    public void visit(Construct node);
}
```

------
#### [ C\# ]

```
public interface IAspect
{
    void Visit(IConstruct node);
}
```

------

When you call `Aspects.of(SCOPE).add(...)`, the construct adds the aspect to an internal list of aspects\. You can obtain the list with `Aspects.of(SCOPE)`\.

During the [prepare phase](apps.md#lifecycle), the AWS CDK calls the `visit` method of the object for the construct and each of its children in top\-down order\.

Although the aspect object is free to change any aspect of the construct, it only operates on a specific subset of construct types\. After determining the construct type, it can call any method and inspect or assign any property on the construct\.

## Example<a name="aspects_example"></a>

The following example validates that all buckets created in the stack have versioning enabled\. The aspect adds an error to the constructs that fail the validation, which results in the synth operation failing and prevents deploying the resulting cloud assembly\.

------
#### [ TypeScript ]

```
class BucketVersioningChecker implements IAspect {
  public visit(node: IConstruct): void {
    // See that we're dealing with a CfnBucket
    if (node instanceof s3.CfnBucket) {

      // Check for versioning property, exclude the case where the property
      // can be a token (IResolvable).
      if (!node.versioningConfiguration 
        || (!Tokenization.isResolvable(node.versioningConfiguration)
            && node.versioningConfiguration.status !== 'Enabled')) {
        node.node.addError('Bucket versioning is not enabled');
      }
    }
  }
}

// Later, apply to the stack
Aspects.of(stack).add(new BucketVersioningChecker());
```

------
#### [ JavaScript ]

```
class BucketVersioningChecker {
   visit(node) {
    // See that we're dealing with a CfnBucket
    if ( node instanceof s3.CfnBucket) {

      // Check for versioning property, exclude the case where the property
      // can be a token (IResolvable).
      if ( !node.versioningConfiguration 
        || !Tokenization.isResolvable(node.versioningConfiguration)
            && node.versioningConfiguration.status !== 'Enabled') {
        node.node.addError('Bucket versioning is not enabled');
      }
    }
  }
}

// Later, apply to the stack
Aspects.of(stack).add(new BucketVersioningChecker());
```

------
#### [ Python ]

```
@jsii.implements(core.IAspect)
class BucketVersioningChecker:
    
  def visit(self, node):
    # See that we're dealing with a CfnBucket
    if isinstance(node, s3.CfnBucket):

      # Check for versioning property, exclude the case where the property
      # can be a token (IResolvable).
      if (!node.versioning_configuration or
         !Tokenization.is_resolvable(node.versioning_configuration)
            and node.versioning_configuration.status != "Enabled"):
        
        node.node.add_error('Bucket versioning is not enabled')

# Later, apply to the stack
Aspects.of(stack).add(BucketVersioningChecker())
```

------
#### [ Java ]

```
public class BucketVersioningChecker implements IAspect
{
    @Override
    public void visit(Construct node)
    {
        // See that we're dealing with a CfnBucket
        if (node instanceof CfnBucket)
        {
            CfnBucket bucket = (CfnBucket)node;
            Object versioningConfiguration = bucket.getVersioningConfiguration();
            if (versioningConfiguration == null ||
                    !Tokenization.isResolvable(versioningConfiguration.toString()) &&
                    !versioningConfiguration.toString().contains("Enabled")
                bucket.getNode().addError("Bucket versioning is not enabled");
        }
    }
}


// Later, apply to the stack
Aspects.of(stack).add(new BucketVersioningChecker());
```

------
#### [ C\# ]

```
class BucketVersioningChecker : Amazon.Jsii.Runtime.DeputyBase, IAspect
{
    public void Visit(IConstruct node)
    {
        // See that we're dealing with a CfnBucket
        if (node is CfnBucket)
        {
            var bucket = (CfnBucket)node;
            if (bucket.VersioningConfiguration is null ||
                    !Tokenization.IsResolvable(bucket.VersioningConfiguration) &&
                    !bucket.VersioningConfiguration.ToString().Contains("Enabled")
                bucket.Node.AddError("Bucket versioning is not enabled");
        }
    }
}

// Later, apply to the stack
Aspects.Of(stack).add(new BucketVersioningChecker());
```

------