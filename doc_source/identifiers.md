# Identifiers<a name="identifiers"></a>

The AWS CDK deals with many types of identifiers and names\. To use the AWS CDK effectively and avoid errors, you need to understand the types of identifiers\.

Identifiers must be unique within the scope in which they are created; they do not need to be globally unique in your AWS CDK application\.

If you attempt to create an identifier with the same value within the same scope, the AWS CDK throws an exception\.

## Construct IDs<a name="identifiers_construct_ids"></a>

The most common identifier, `id`, is the identifier passed as the second argument when instantiating a construct object\. This identifier, like all identifiers, need only be unique within the scope in which it is created, which is the first argument when instantiating a construct object\.

**Note**  
The `id` of a stack is also the identifier you use to refer to it in the [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.

Let's look at an example where we have two constructs with the identifier `MyBucket` in our app\. However, since they are defined in different scopes, the first in the scope of the stack with the identifier `Stack1`, and the second in the scope of a stack with the identifier `Stack2`, that doesn't cause any sort of conflict, and they can co\-exist in the same app without any issues\.

------
#### [ TypeScript ]

```
import { App, Construct, Stack, StackProps } from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

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
const { App , Stack } = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

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
from aws_cdk.core import App, Construct, Stack, StackProps
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

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.s3.Bucket;

public class MyStack extends Stack {
    public MyStack(final App scope, final String id) {
        this(scope, id, null);
    }
    
    public MyStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);
        new Bucket(this, "MyBucket");
    }
}

// Main.java
package com.myorg;

import software.amazon.awscdk.core.App;

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
using core = Amazon.CDK;
using s3 = Amazon.CDK.AWS.S3;

public class MyStack : core.Stack
{
    public MyStack(core.App scope, string id, core.IStackProps props) : base(scope, id, props)
    {
        new s3.Bucket(this, "MyBucket");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var app = new core.App();
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

Since AWS CloudFormation requires that all logical IDs in a template are unique, the AWS CDK must be able to generate a unique identifier for each construct in an application\. Since the AWS CDK already has paths that are globally unique, the AWS CDK generates these unique identifiers by concatenating the elements of the path, and adds an 8\-digit hash\. The hash is necessary, as otherwise two distinct paths, such as `A/B/C` and `A/BC` would result in the same identifier\. The AWS CDK calls this concatenated path elements and hash the *unique ID* of the construct\.

You can access the unique ID of any construct programmatically, as shown in the following example, which gets the unique ID of `myConstruct` \(or `my_construct` in Python conventions\)\. Since ids must be unique within the scope they are created, their paths are always unique within a AWS CDK application\.

------
#### [ TypeScript ]

```
const uid: string = myConstruct.node.uniqueId;
```

------
#### [ JavaScript ]

```
const uid = myConstruct.node.uniqueId;
```

------
#### [ Python ]

```
uid = my_construct.node.unique_id
```

------
#### [ Java ]

```
String uid  = myConstruct.getNode().getUniqueId();
```

------
#### [ C\# ]

```
string uid = myConstruct.Node.UniqueId;
```

------

## Logical IDs<a name="identifiers_logical_ids"></a>

Unique IDs serve as the *logical identifiers*, which are sometimes called *logical names*, of resources in the generated AWS CloudFormation templates for those constructs that represent AWS resources\.

For example, the Amazon S3 bucket in the previous example that is created within `Stack2` results in an `AWS::S3::Bucket` resource with the logical ID `Stack2MyBucket4DD88B4F` in the resulting AWS CloudFormation template\.

Think of construct IDs as part of your construct's public contract\. If you change the ID of a construct in your construct tree, AWS CloudFormation will replace the deployed resource instances of that construct, potentially causing service interruption or data loss\.

### Logical ID stability<a name="identifiers_logical_id_stability"></a>

Avoid changing the logical ID of a resource between deployments\. Since AWS CloudFormation identifies resources by their logical ID, if you change the logical ID of a resource, AWS CloudFormation deletes the existing resource, and then creates a new resource with the new logical ID\.