# Translating TypeScript AWS CDK code to other languages<a name="multiple_languages"></a>

TypeScript was the first language supported for developing AWS CDK applications, and for that reason, there is a substantial amount of example CDK code written in TypeScript\. If you are developing in another language, it may be useful to compare how AWS CDK code is implemented in TypeScript and your language of choice, so you can, with a little effort, make use of these examples\.

For more details on working with the AWS CDK in its supported programming languages, see:
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)

## Importing a module<a name="multiple_languages_import"></a>

------
#### [ TypeScript/JavaScript ]

TypeScript supports importing either an entire module, or individual objects from a module\. 

```
// Import entire module as s3 into current namespace
import * as s3 from '@aws-cdk/aws-s3';

// Import an entire module using Node.js require() (import * as s3 generally preferred)
const s3 = require('@aws-cdk/aws-s3');

// TypeScript version of require() (again, import * as s3 generally preferred)
import s3 = require('@aws-cdk/aws-s3');

// Now use s3 to access the S3 types
const bucket = s3.Bucket(...);

// Selective import of s3.Bucket into current namespace
import { Bucket } from '@aws-cdk/aws-s3';

// Selective import of Bucket and EventType into current namespace
import { Bucket, EventType } from '@aws-cdk/aws-s3';

// Now use Bucket to instantiate an S3 bucket
const bucket = Bucket(...);
```

------

------
#### [ Python ]

Like TypeScript, Python supports namespaced module imports and selective imports\. Module names in Python look like **aws\_cdk\.***xxx*, where *xxx* represents an AWS service name, such as **s3** for Amazon S3 \(we'll use Amazon S3 for our examples\)\.

```
# Import entire module as s3 into current namespace
import aws_cdk.aws_s3 as s3

# s3 can now be used to access classes it contains
bucket = s3.Bucket(...)

# Selective import of s3.Bucket into current namespace
from aws_cdk.s3 import Bucket

# Selective import of Bucket and EventType into current namespace
from aws_cdk.s3 import Bucket, EventType

# Bucket can now be used to instantiate a bucket
bucket = Bucket(...)
```

------
#### [ Java ]

Java's imports work differently from TypeScript's\. Each import statement imports either a single class name from a given package, or all classes defined in that package \(using `*`\)\. After importing, classes may be accessed using either the class name by itself or \(in case of name conflicts\) the *qualified* class name including its package\.

Packages are named like `software.amazon.awscdk.services.xxx` for AWS Construct Library packages \(the core module is `software.amazon.awscdk.core`\)\. The Maven group ID for AWS CDK packages is `software.amazon.awscdk`\.

```
// Make all Amazon S3 construct library classes available
import software.amazon.awscdk.services.s3.*;

// Make only Bucket and EventType classes available
import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.s3.EventType;

// An imported class may now be accessed using the simple class name (assuming that name
// does not conflict with another class)
Bucket bucket = new Bucket(...);

// We can always use the qualified name of a class (including its package) even without an
// import directive
software.amazon.awscdk.services.s3.Bucket bucket = 
     new software.amazon.awscdk.services.s3.Bucket(...);
```

------
#### [ C\# ]

In C\#, you import types with the `using` directive\. There are two styles, which give you access either all the types in the specified namespace using their plain names, or to refer to the namespace itself using an alias\.

Packages are named like `Amazon.CDK.AWS.xxx` for AWS Construct Library packages \(the core module is `Amazon.CDK`\)\.

```
// Make all Amazon S3 construct library classes available
using Amazon.CDK.AWS.S3;

// Now we can access any S3 type using its name
var bucket = new Bucket(...);

// Import the S3 namespace under an alias
using s3 = Amazon.CDK.AWS.S3;

// Now we can access an S3 type through the namespace alias
var bucket = new s3.Bucket(...);

// We can always use the qualified name of a type (including its namespace) even without a
// using directive
var bucket = new Amazon.CDK.AWS.S3.Bucket(...)
```

------

## Instantiating a construct<a name="multiple_languages_class"></a>

AWS CDK construct classes have the same name in all supported languages\. Most languages use the `new` keyword to instantiate a class \(Python is the only one that doesn't\)\. Also, in most languages, the keyword `this` refers to the current instance\. Python, again, is the exception \(it uses `self` by convention\)\. You should pass a reference to the current instance as the `scope` parameter to every construct you create\.

 The third argument to a AWS CDK construct is `props`, an object containing attributes needed to build the construct\. This argument may be optional, but when it is required, the supported languages handle it in idiomatic ways\. The names of the attributes are also adapted to the language's standard naming patterns\. 

------
#### [ TypeScript/JavaScript ]

```
// Instantiate default Bucket
const bucket = new s3.Bucket(this, 'MyBucket');

// Instantiate Bucket with bucketName and versioned properties
const bucket = new s3.Bucket(this, 'MyBucket', {
  bucketName: 'my-bucket',
   versioned: true,
});

// Instantiate Bucket with websiteRedirect, which has its own sub-properties
const bucket = new s3.Bucket(this, 'MyBucket', {
  websiteRedirect: {host: 'aws.amazon.com'}});
```

------

------
#### [ Python ]

Python doesn't use a `new` keyword when instantiating a class\. The properties argument is represented using keyword arguments, and the arguments are named using `snake_case`\.

If a props value is itself a bundle of attributes, it is represented by a class named after the property, which accepts keyword arguments for the sub\-properties\.

In Python, the current instance is passed to methods as the first argument, which is named `self` by convention\.

```
# Instantiate default Bucket
bucket = s3.Bucket(self, "MyBucket")

# Instantiate Bucket with bucket_name and versioned properties
bucket = s3.Bucket(self, "MyBucket", bucket_name="my-bucket", versioned=true)

# Instantiate Bucket with website_redirect, which has its own sub-properties
bucket = s3.Bucket(self, "MyBucket", website_redirect=s3.WebsiteRedirect(
            host_name="aws.amazon.com"))
```

------
#### [ Java ]

In Java, the props argument is represented by a class named `XxxxProps` \(for example, `BucketProps` for the `Bucket` construct's props\)\. You build the props argument using a builder pattern\.

Each `XxxxProps` class has a builder, and there is also a convenient builder for each construct that builds the props and the construct in one step, as shown here\.

Props are named the same as in TypeScript, using `camelCase`\.

```
// Instantiate default Bucket
Bucket bucket = Bucket(self, "MyBucket");

// Instantiate Bucket with bucketName and versioned properties
Bucket bucket = Bucket.Builder.create(self, "MyBucket")
                      .bucketName("my-bucket").versioned(true)
                      .build();

# Instantiate Bucket with websiteRedirect, which has its own sub-properties
Bucket bucket = Bucket.Builder.create(self, "MyBucket")
                      .websiteRedirect(new websiteRedirect.Builder()
                          .hostName("aws.amazon.com").build())
                      .build();
```

------
#### [ C\# ]

In C\#, props are specified using an object initializer to a class named `XxxxProps` \(for example, `BucketProps` for the `Bucket` construct's props\)\.

Props are named similarly to TypeScript, except using `PascalCase`\.

It is convenient to use the `var` keyword when instantiating a construct, so you don't need to type the class name twice\. However, your local code style guide may vary\.

```
// Instantiate default Bucket
var bucket = Bucket(self, "MyBucket");

// Instantiate Bucket with BucketName and versioned properties
var bucket =  Bucket(self, "MyBucket", new BucketProps {
                      BucketName = "my-bucket",
                      Versioned  = true});

// Instantiate Bucket with WebsiteRedirect, which has its own sub-properties
var bucket = Bucket(self, "MyBucket", new BucketProps {
                      WebsiteRedirect = new WebsiteRedirect {
                              HostName = "aws.amazon.com"
                      }});
```

------

## Accessing members<a name="multiple_languages_members"></a>

It is common to refer to attributes or properties of constructs and other AWS CDK classes and use these values as, for examples, inputs to build other constructs\. The naming differences described above for methods apply\. Furthermore, in Java, it is not possible to access members directly; instead, a getter method is provided\.

------
#### [ TypeScript/JavaScript ]

Names are `camelCase`\.

```
bucket.bucketArn
```

------

------
#### [ Python ]

Names are `snake_case`\.

```
bucket.bucket_arn
```

------
#### [ Java ]

A getter method is provided for each property; these names are `camelCase`\.

```
bucket.getBucketArn()
```

------
#### [ C\# ]

Names are `PascalCase`\.

```
bucket.BucketArn
```

------

## Enum constants<a name="multiple_languages_enums"></a>

Enum constants are scoped to a class, and have uppercase names with underscores in all languages \(sometimes referred to as `SCREAMING_SNAKE_CASE`\)\. Since class names also use the same casing in all supported languages, qualified enum names are also the same\.

```
s3.BucketEncryption.KMS_MANAGED
```

## Object interfaces<a name="multiple_languages_object"></a>

The AWS CDK uses TypeScript object interfaces to indicate that a class implements an expected set of methods and properties\. You can recognize an object interface because its name starts with `I`\. A concrete class indicates the interface\(s\) it implements using the `implements` keyword\.

------
#### [ TypeScript/JavaScript ]

**Note**  
JavaScript doesn't have an interface feature\. You can ignore the `implements` keyword and the class names following it\.

```
import { IAspect, IConstruct } from '@aws-cdk/core';

class MyAspect implements IAspect {
  public visit(node: IConstruct) {
    console.log('Visited', node.node.path);
  }
}
```

------

------
#### [ Python ]

Python doesn't have an interface feature\. However, for the AWS CDK you can indicate interface implementation by decorating your class with `@jsii.implements(interface)`\. 

```
from aws_cdk.core import IAspect, IConstruct
import jsii

@jsii.implements(IAspect)
class MyAspect():
  def visit(self, node: IConstruct) -> None:
    print("Visited", node.node.path)
```

------
#### [ Java ]

```
import software.amazon.awscdk.core.IAspect;
import software.amazon.awscdk.core.IConstruct;

public class MyAspect implements IAspect {
    public void visit(IConstruct node) {
        System.out.format("Visited %s", node.getNode().getPath());
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;

public class MyAspect : IAspect
{
    public void Visit(IConstruct node)
    {
        System.Console.WriteLine($"Visited ${node.Node.Path}");
    }
}
```

------