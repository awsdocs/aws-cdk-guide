# Identifiers<a name="identifiers"></a>

The AWS CDK deals with many types of identifiers and names\. To use the AWS CDK effectively and avoid errors, you need to understand the types of identifiers\.

Identifiers must be unique within the scope in which they are created; they do not need to be globally unique in your AWS CDK application\.

If you attempt to create an identifier with the same value within the same scope, the AWS CDK throws an exception\.

## Construct IDs<a name="identifiers_construct_ids"></a>

The most common identifier, `id`, is the identifier passed as the second argument when instantiating a construct object\. This identifier, like all identifiers, need only be unique within the scope in which it is created, which is the first argument when instantiating a construct object\.

**Note**  
The `id` of a stack is also the identifier you use to refer to it in the [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.

Let's look at an example where we have two constructs with the identifier `MyBucket` in our app\. However, since they are defined in different scopes, the first in the scope of the stack with the identifier `Stack1`, and the second in the scope of a stack with the identifier `Stack2`, that doesn't cause any sort of conflict, and they can coexist in the same app without any issues\.

------
#### [ TypeScript ]

```
import { App, Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

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

------
#### [ JavaScript ]

```
const { App , Stack } = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');

class MyStack extends Stack {
  constructor(scope, id, props = {}) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyBucket');
  }
}

const app = new App();
new MyStack(app, 'Stack1');
new MyStack(app, 'Stack2');
```

------
#### [ Python ]

```
from aws_cdk import App, Construct, Stack, StackProps
from constructs import Construct
from aws_cdk import aws_s3 as s3

class MyStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs):

        super().__init__(scope, id, **kwargs)
        s3.Bucket(self, "MyBucket")

app = App()
MyStack(app, 'Stack1')
MyStack(app, 'Stack2')
```

------
#### [ Java ]

```
// MyStack.java
package com.myorg;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.constructs.Construct;
import software.amazon.awscdk.services.s3.Bucket;

public class MyStack extends Stack {
    public MyStack(final Construct scope, final String id) {
        this(scope, id, null);
    }
    
    public MyStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);
        new Bucket(this, "MyBucket");
    }
}

// Main.java
package com.myorg;

import software.amazon.awscdk.App;

public class Main {
    public static void main(String[] args) {
        App app = new App();
        new MyStack(app, "Stack1");
        new MyStack(app, "Stack2");
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using constructs;
using Amazon.CDK.AWS.S3;

public class MyStack : Stack
{
    public MyStack(Construct scope, string id, IStackProps props) : base(scope, id, props)
    {
        new Bucket(this, "MyBucket");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var app = new App();
        new MyStack(app, "Stack1");
        new MyStack(app, "Stack2");
    }
}
```

------

## Paths<a name="identifiers_paths"></a>

The constructs in an AWS CDK application form a hierarchy rooted in the `App` class\. We refer to the collection of IDs from a given construct, its parent construct, its grandparent, and so on to the root of the construct tree, as a *path*\.

The AWS CDK typically displays paths in your templates as a string, with the IDs from the levels separated by slashes, starting at the node just below the root `App` instance, which is usually a stack\. For example, the paths of the two Amazon S3 bucket resources in the previous code example are `Stack1/MyBucket` and `Stack2/MyBucket`\.

You can access the path of any construct programmatically, as shown in the following example, which gets the path of `myConstruct` \(or `my_construct`, as Python developers would write it\)\. Since IDs must be unique within the scope they are created, their paths are always unique within a AWS CDK application\.

------
#### [ TypeScript ]

```
const path: string = myConstruct.node.path;
```

------
#### [ JavaScript ]

```
const path = myConstruct.node.path;
```

------
#### [ Python ]

```
path = my_construct.node.path
```

------
#### [ Java ]

```
String path = myConstruct.getNode().getPath();
```

------
#### [ C\# ]

```
string path = myConstruct.Node.Path;
```

------

## Unique IDs<a name="identifiers_unique_ids"></a>

Since AWS CloudFormation requires that all logical IDs in a template are unique, the AWS CDK must be able to generate a unique identifier for each construct in an application\. Resources have paths that are globally unique \(the names of all scopes from the stack to a specific resource\) so the AWS CDK generates the necessary unique identifiers by concatenating the elements of the path and adding an 8\-digit hash\. \(The hash is necessary to distinguish distinct paths, such as `A/B/C` and `A/BC`, that would result in the same AWS CloudFormation identifier, since AWS CloudFormation identifiers are alphanumeric and cannot contain slashes or other separator characters\.\) The AWS CDK calls this string the *unique ID* of the construct\.

In general, your AWS CDK app should not need to know about unique IDs\. You can, however, access the unique ID of any construct programmatically, as shown in the following example\.

------
#### [ TypeScript ]

```
const uid: string = Names.uniqueId(myConstruct);
```

------
#### [ JavaScript ]

```
const uid = Names.uniqueId(myConstruct);
```

------
#### [ Python ]

```
uid = Names.unique_id(my_construct)
```

------
#### [ Java ]

```
String uid = Names.uniqueId(myConstruct);
```

------
#### [ C\# ]

```
string uid = Names.Uniqueid(myConstruct);
```

------

The *address* is another kind of unique identifier that uniquely distinguishes CDK resources\. Derived from the SHA\-1 hash of the path, it is not human\-readable, but its constant, relatively short length \(always 42 hexadecimal characters\) makes it useful in situations where the "traditional" unique ID might be too long\. Some constructs may use the address in the synthesized AWS CloudFormation template instead of the unique ID\. Again, your app generally should not need to know about its constructs' addresses, but you can retrieve a construct's address as follows\.

------
#### [ TypeScript ]

```
const addr: string = myConstruct.node.addr;
```

------
#### [ JavaScript ]

```
const addr = myConstruct.node.addr;
```

------
#### [ Python ]

```
addr = my_construct.node.addr
```

------
#### [ Java ]

```
String addr = myConstruct.getNode().getAddr();
```

------
#### [ C\# ]

```
string addr = myConstruct.Node.Addr;
```

------

## Logical IDs<a name="identifiers_logical_ids"></a>

Unique IDs serve as the *logical identifiers*, which are sometimes called *logical names*, of resources in the generated AWS CloudFormation templates for those constructs that represent AWS resources\.

For example, the Amazon S3 bucket in the previous example that is created within `Stack2` results in an `AWS::S3::Bucket` resource with the logical ID `Stack2MyBucket4DD88B4F` in the resulting AWS CloudFormation template\.

Think of construct IDs as part of your construct's public contract\. If you change the ID of a construct in your construct tree, AWS CloudFormation will replace the deployed resource instances of that construct, potentially causing service interruption or data loss\.

### Logical ID stability<a name="identifiers_logical_id_stability"></a>

Avoid changing the logical ID of a resource between deployments\. Since AWS CloudFormation identifies resources by their logical ID, if you change the logical ID of a resource, AWS CloudFormation deletes the existing resource, and then creates a new resource with the new logical ID\.

## The "Default" Construct ID<a name="default_construct_id"></a>

Using "Default" as a construct ID has special behavior in the CDK. Unlike other
IDs you may use, "Default" will:

1. Not affect the hash suffix of the construct's logical ID.
2. Not include the construct ID ("Default") in the human-readable portion of the logical ID.
3. Allow the `construct.node.defaultChild` API to return the "Default" construct.

Using "Default" as a construct ID for is a good choice when implementing a
construct that wraps one primary, child construct. For instance, an author
writing a construct named `ReplicatedBucket` that wraps a single `s3.Bucket`
could choose to give the `s3.Bucket` the logical ID "Bucket". When a user
instantiates a `ReplicatedBucket` and provides "ReplicatedBucket" as the ID, the
logical ID of the s3 bucket would be "ReplicatedBucketBucket\<hash\>". If the
construct author instead chose to use "Default" for the `s3.Bucket` construct
ID, the logical ID would be "ReplicatedBucket\<hash\>", eliminating the
additional "Bucket" name for the `s3.Bucket`.

Using "Default" as a construct ID can also help in cases where you would like to
refactor your code to extract a new construct class. Consider an example where
you have deployed a CDK app and would like to move an S3 bucket to a new
construct called "MyBucket".

```ts
export class MyStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Path: MyStack/Bucket/Resource 
    // Logical ID: Bucket83908E77
    // Bucket Name: mystack-bucket83908e77-1mwpz4z08f2eq
    new s3.Bucket(this, "Bucket");
  }
}
```

The logical ID for the s3 Bucket in this case is "Bucket83908E77". Notice that
the special name "Resource" from the path is omitted from the logical ID and the
generated bucket name.

Now let's extract this Bucket to a new class called `MyBucket` where we plan to
add other, related constructs.

```ts
export class MyBucket extends Construct {
  constructor(scope: Construct, id: string, props?: MyBucketProps) {
    super(scope, id);
    
    new s3.Bucket(this, "Bucket");
  }
}
```

When we replace the `s3.Bucket` with our new `MyBucket` construct, the unique
ID, path and bucket name will change.

```ts
export class MyStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Path: MyStack/Bucket/Bucket/Resource 
    // Logical ID: BucketD7FEB781
    // Bucket Name: mystack-bucketd7feb781-1nnqpcke5tgoj
    new MyBucket(this, "Bucket");
  }
}
```

This change will orphan the existing bucket and create a new one during
deployment.  However, the human-readable portion of the logical ID (Bucket) and
bucket name (my-stack-bucket) did not change.  CDK removes repeated construct
IDs from the path when building the logical ID to avoid adding noise to logical
IDs like `BucketBucketD7FEB781`.

If we instead update our extracted `MyBucket` construct to use "Default" as the
logical ID instead of "Bucket", we'll see no change in our synthesized output.

```ts
export class MyBucket extends Construct {
  constructor(scope: Construct, id: string, props?: MyBucketProps) {
    super(scope, id);
    
    new s3.Bucket(this, "Default");
  }
}
```

When we `synth` our app this time, the logical ID matches the original before
the refactoring (Bucket83908E77) and the path reflects the new nesting
(MyStack/Bucket/Default/Resource). This code change won't affect our deployed
infrastructure. 

A limitation to this approach is that, like any Construct ID, "Default" can't be
repeated in the current construct's scope. So if you want to extract two s3
Buckets into a new construct, only one could keep its existing logical ID and
use "Default" as its new construct ID. To keep existing logical IDs from
changing during refactoring also consider using the [`Stack.renameLogicalId()
API`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#renamewbrlogicalwbridoldid-newid).