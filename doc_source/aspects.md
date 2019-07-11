# Aspects<a name="aspects"></a>

Aspects are the way to apply an operation to all constructs in a given scope\. The functionality could modify the constructs, such as by adding tags, or it could be verifying something about the state of the constructs, such as ensuring that all buckets are encrypted\.

To apply an aspect to a construct and all constructs in the same scope, call [node\.applyAspect](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.ConstructNode.html#apply-aspectaspect) with a new aspect, as shown in the following example\.

```
myConstruct.node.applyAspect(new SomeAspect(...));
```

The AWS CDK currently uses aspects only to [tag resources](tagging.md), but the framework is extensible and can also be used for other purposes\. For example, you can use it to validate or change the AWS CloudFormation resources that are defined for you\.

## Aspects in Detail<a name="aspects_detail"></a>

The AWS CDK implements tagging using a more generic system, called aspects, which is an instance of the visitor pattern\. An aspect is a class that implements the following interface\.

```
interface IAspect {
   visit(node: IConstruct): void;}
```

When you call `construct.node.applyAspect(aspect)`, the construct adds the aspect to an internal list of aspects\.

During the [prepare phase](apps.md#lifecycle), the AWS CDK calls the visit method of the object for the construct and each of its children in top\-down order\.

Although the aspect object is free to change any aspect of the construct object, it only operates on a specific subset of construct types\. After determining the construct type, it can call any method and inspect or assign any property on the construct\.

## Example<a name="aspects_example"></a>

The following example validates that all buckets created in the stack have versioning enabled\. The aspect adds an error to the constructs that fail the validation, which results in the synth operation failing and prevents deploying the resulting cloud assembly\.

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

// Apply to the stack
stack.node.applyAspect(new BucketVersioningChecker());
```