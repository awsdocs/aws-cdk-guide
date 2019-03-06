--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Pass in a Value from Another Stack<a name="passing_stack_value"></a>

You can pass a value from one stack to another stack in the same app by using the `export` method in one stack and the `import` method in the other stack\.

The following example creates a bucket on one stack and passes a reference to that bucket to the other stack through an interface\.

First create a stack with a bucket\. The stack includes a property we use to pass the bucket's properties to the other stack\. Notice how we use the `export` method on the bucket to get its properties and save them in the stack property\.

```
class HelloCdkStack extends cdk.Stack {
  // Property that defines the stack you are exporting from
  public readonly myBucketImportProps: s3.BucketImportProps;

  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
      super(scope, id, props);

      const mybucket = new s3.Bucket(this, "MyFirstBucket");

      // Save bucket's BucketImportProps
      this.myBucketImportProps = mybucket.export();
  }
}
```

Create an interface for the second stack's properties\. We use this interface to pass the bucket properties between the two stacks\.

```
// Interface we'll use to pass the bucket's properties to another stack
interface MyCdkStackProps {
    theBucketImportProps: s3.BucketImportProps;
}
```

Create the second stack that gets a reference to the other bucket from the properties passed in through the constructor\.

```
// The class for the other stack
class MyCdkStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyCdkStackProps) {
    super(scope, id);

    const myOtherBucket = s3.Bucket.import(this, "MyOtherBucket", props.theBucketImportProps);

    // Do something with myOtherBucket
  }
}
```

Finally, connect the dots in your app\.

```
const app = new cdk.App();

const myStack = new HelloCdkStack(app, "HelloCdkStack");

new MyCdkStack(app, "MyCdkStack", {
  theBucketImportProps: myStack.myBucketImportProps
});

app.run();
```