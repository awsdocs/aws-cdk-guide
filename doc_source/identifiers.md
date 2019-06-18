--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Identifiers<a name="identifiers"></a>

The AWS CDK deals with many types of identifiers and names\. To use the AWS CDK effectively and avoid errors, you need to understand the types of identifiers\.

Identifiers must be unique within the scope in which they are created; they do not need to be globally unique in your AWS CDK application\.

If you attempt to create an identifier with the same value within the same scope, the AWS CDK throws an exception\.

## Construct IDs<a name="identifiers_construct_ids"></a>

The most common identifier, `id`, is the identifier passed as the second argument when instantiating a construct object\. This identifier, like all identifiers, need only be unique within the scope in which it is created, which is the first argument when instantiating a construct object\.

Lets look at an example where we have two constructs with the identifier `MyBucket` in our app\. However, since they are defined in different scopes, the first in the scope of the stack with the identifier `Stack1`, and the second in the scope of a stack with the identifier `Stack2`, that doesn't cause any sort of conflict, and they can co\-exist in the same app without any issues\.

```
import { App, Construct, Stack, StackProps } from '@aws-cdk/cdk';
import s3 = require('@aws-cdk/aws-s3');

class MyStack extends Stack {
  constructor(scope: Construct, id: string, props: StackProps = {}) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyBucket');
  }
}

const app = new App();
new MyStack(app, 'Stack1');
new MyStack(app, 'Stack2');
```

## Paths<a name="identifiers_paths"></a>

As the constructs in an AWS CDK application form a hierarchy, we refer to the collection of ids from a given construct, then of its parent construct, then grandparent construct, and so on up to the root of the construct tree, which is an instance of the `App` class as a *path*\.

The AWS CDK typically displays paths in your templates as a string, with the ids from the levels separated by slashes, starting at the node just below the root `App` instance, which is usually a stack\. For example, the paths of the two Amazon S3 bucket resources in the previous code example are `Stack1/MyBucket` and `Stack2/MyBucket`\.

You can access the path of any construct programatically, as shown in the following example, which gets the path of `myConstruct`\. Since ids must be unique within the scope they are created, their paths are always unique within a AWS CDK application\.

```
const path: string = myConstruct.node.path;
```

## Unique IDs<a name="identifiers_unique_ids"></a>

Since AWS CloudFormation requires that all logical IDs in a template are unique, the AWS CDK must be able to generate unique identifier for each construct in an application\. Since the AWS CDK already has paths that are globally unique, the AWS CDK generates these unique identifiers by concatenating the elements of the path, and adds an 8\-digit hash\. The hash is necessary, as otherwise two distinct paths, such as `A/B/C` and `A/BC` would result in the same identifier\. The AWS CDK calls this concatenated path elements and hash the *unique ID* of the construct\.

You can access the unique ID of any construct programatically, as shown in the following example, which gets the unique ID of `myConstruct`\. Since ids must be unique within the scope they are created, their paths are always unique within a AWS CDK application\.

```
const uid: string = myConstruct.node.uniqueId;
```

## Logical IDs<a name="identifiers_logical_ids"></a>

Unique IDs serve as the *logical identifiers*, which are sometimes called *logical names*, of resources in the generated AWS CloudFormation templates for those constructs that represent AWS resources\.

For example, the Amazon S3 bucket in the previous example that is created within `Stack2` results in an `AWS::S3::Bucket` resource with the logical ID `Stack2MyBucket4DD88B4F` in the resulting AWS CloudFormation template\.

Think of construct IDs as part of your construct's public contract\. If you change the ID of a construct in your construct tree, AWS CloudFormation will replace the deployed resource instances of that construct, potentially causing service interruption or data loss\.

### Logical ID Stability<a name="identifiers_logical_id_stability"></a>

Avoid changing the logical ID of a resource between deployments\. Since AWS CloudFormation identifies resources by their logical ID, if you change the logical ID of a resource, AWS CloudFormation deletes the existing resource, and then creates a new resource with the new logical ID\.