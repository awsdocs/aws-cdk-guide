# Working with the AWS CDK in Go<a name="work-with-cdk-go"></a>

Go is a fully\-supported client language for the AWS CDK and is considered stable\. Working with the AWS CDK in Go uses familiar tools\. The Go version of the AWS CDK even uses Go\-style identifiers\.

Unlike the other languages the CDK supports, Go is not a traditional object\-oriented programming language\. Go uses composition where other languages often leverage inheritance\. We have tried to employ idiomatic Go approaches as much as possible, but there are places where the CDK charts its own course\.

This topic explains the ins and outs of working with the AWS CDK in Go\. See the [announcement blog post](https://aws.amazon.com/blogs/developer/getting-started-with-the-aws-cloud-development-kit-and-go/) for a walkthrough of a simple Go project for the AWS CDK\.

## Prerequisites<a name="go-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

The Go bindings for the AWS CDK use the standard [Go toolchain](https://golang.org/dl/), v1\.16 or later\. You can use the editor of your choice\.

**Note**  
Third\-party Language Deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

## Creating a project<a name="go-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language go
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. Hyphens in the folder name are converted to underscores\. However, the name should otherwise follow the form of a Go identifier; for example, it should not start with a number or contain spaces\. 

The resulting project includes a reference to the core AWS CDK Go module, `github.com/aws/aws-cdk-go/awscdk/v2`, in `go.mod`\. Issue go get to install this and other required modules\.

## Managing AWS Construct Library modules<a name="go-managemodules"></a>

In most AWS CDK documentation and examples, the word "module" is often used to refer to AWS Construct Library modules, one or more per AWS service, which differs from idiomatic Go usage of the term\. The CDK Construct Library is provided in one Go module with the individual Construct Library modules, which support the various AWS services, provided as Go packages within that module\.

Some services' AWS Construct Library support is in more than one Construct Library module \(Go package\)\. For example, Amazon Route 53 has three Construct Library modules in addition to the main `awsroute53` package, named `awsroute53patterns`, `awsroute53resolver`, and `awsroute53targets`\.

The AWS CDK's core package, which you'll need in most AWS CDK apps, is imported in Go code as `github.com/aws/aws-cdk-go/awscdk/v2`\. Packages for the various services in the AWS Construct Library live under `github.com/aws/aws-cdk-go/awscdk/v2`\. For example, the Amazon S3 module's namespace is `github.com/aws/aws-cdk-go/awscdk/v2/awss3`\.

```
import (
        "github.com/aws/aws-cdk-go/awscdk/v2/awss3"
        // ...
)
```

Once you have imported the Construct Library modules \(Go packages\) for the services you want to use in your app, you access constructs in that module using, for example, `awss3.Bucket`\.

## AWS CDK idioms in Go<a name="go-cdk-idioms"></a>

### Field and method names<a name="go-naming"></a>

Field and method names use camel casing \(`likeThis`\) in TypeScript, the CDK's language of origin\. In Go, these follow Go conventions, so are Pascal\-cased \(`LikeThis`\)\.

### Missing values and pointer conversion<a name="go-missing-values"></a>

In Go, missing values in AWS CDK objects such as property bundles are represented by `nil`\. Go doesn't have nullable types; the only type that can contain `nil` is a pointer\. To allow values to be optional, then, all CDK properties, arguments, and return values are pointers, even for primitive types\. This applies to required values as well as optional ones, so if a required value later becomes optional, no breaking change in type is needed\.

When passing literal values or expressions, use the following helper functions to create pointers to the values\.
+ `jsii.String`
+ `jsii.Number`
+ `jsii.Bool`
+ `jsii.Time`

For consistency, we recommend that you use pointers similarly when defining your own constructs, even though it may seem more convenient to, for example, receive your construct's `id` as a string rather than a pointer to a string\.

When dealing with optional AWS CDK values, including primitive values as well as complex types, you should explicitly test pointers to make sure they are not `nil` before doing anything with them\. Go does not have "syntactic sugar" to help handle empty or missing values as some other languages do\. However, required values in property bundles and similar structures are guaranteed to exist \(construction fails otherwise\), so these values need not be `nil`\-checked\.

### Constructs and Props<a name="go-props"></a>

Constructs, which represent one or more AWS resources and their associated attributes, are represented in Go as interfaces\. For example, `awss3.Bucket` is an interface\. Every construct has a factory function, such as `awss3.NewBucket`, to return a struct that implements the corresponding interface\.

All factory functions take three arguments: the `scope` in which the construct is being defined \(its parent in the construct tree\), an `id`, and `props`, a bundle of key/value pairs that the construct uses to configure the resources it creates\. The "bundle of attributes" pattern is also used elsewhere in the AWS CDK\.

In Go, props are represented by a specific struct type for each construct\. For example, an `awss3.Bucket` takes a props argument of type `awss3.BucketProps`\. Use a struct literal to write props arguments\.

```
var bucket = awss3.NewBucket(stack, jsii.String("MyBucket"), &awss3.BucketProps{
    Versioned: jsii.Bool(true),
})
```

### Generic structures<a name="go-generic-structures"></a>

In some places, the AWS CDK uses JavaScript arrays or untyped objects as input to a method\. \(See, for example, AWS CodeBuild's [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_codebuild.BuildSpec.html#static-fromwbrobjectvalue](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_codebuild.BuildSpec.html#static-fromwbrobjectvalue) method\.\) In Go, these objects are represented as slices and an empty interface, respectively\.

The CDK provides variadic helper functions such as `jsii.Strings` for building slices containing primitive types\. 

```
jsii.Strings("One", "Two", "Three")
```

### Developing custom constructs<a name="go-writing-constructs"></a>

In Go, it is usually more straightforward to write a new construct than to extend an existing one\. First, define a new struct type, anonymously embedding one or more existing types if extension\-like semantics are desired\. Write methods for any new functionality you're adding and the fields necessary to hold the data they need\. Define a props interface if your construct needs one\. Finally, write a factory function `NewMyConstruct()` to return an instance of your construct\.

If you are simply changing some default values on an existing construct or adding a simple behavior at instantiation, you don't need all that plumbing\. Instead, write a factory function that calls the factory function of the construct you're "extending\." In other CDK languages, for example, you might create a `TypedBucket` construct that enforces the type of objects in an Amazon S3 bucket by overriding the `s3.Bucket` type and, in your new type's initializer, adding a bucket policy that allows only specified filename extensions to be added to the bucket\. In Go, it is easier to simply write a `NewTypedBucket` that returns an `s3.Bucket` \(instantiated using `s3.NewBucket`\) to which you have added an appropriate bucket policy\. No new construct type is necessary because the functionality is already available in the standard bucket construct; the new "construct" just provides a simpler way to configure it\.

## Building, synthesizing, and deploying<a name="go-running"></a>

The AWS CDK automatically compiles your app before running it\. However, it can be useful to build your app manually to check for errors and to run tests\. You can do this by issuing `go build` at a command prompt while in your project's root directory\.

Run any tests you've written by running `go test` at a command prompt\.

The [stacks](stacks.md) defined in your AWS CDK app can be synthesized and deployed individually or together using the commands below\. Generally, you should be in your project's main directory when you issue them\.
+ `cdk synth`: Synthesizes a AWS CloudFormation template from one or more of the stacks in your AWS CDK app\.
+ `cdk deploy`: Deploys the resources defined by one or more of the stacks in your AWS CDK app to AWS\.

You can specify the names of multiple stacks to be synthesized or deployed in a single command\. If your app defines only one stack, you do not need to specify it\. 

```
cdk synth                 # app defines single stack
cdk deploy Happy Grumpy   # app defines two or more stacks; two are deployed
```

You may also use the wildcards \* \(any number of characters\) and ? \(any single character\) to identify stacks by pattern\. When using wildcards, enclose the pattern in quotes\. Otherwise, the shell may try to expand it to the names of files in the current directory before they are passed to the AWS CDK Toolkit\.

```
cdk synth "Stack?"    # Stack1, StackA, etc.
cdk deploy "*Stack"   # PipeStack, LambdaStack, etc.
```

**Tip**  
You don't need to explicitly synthesize stacks before deploying them; `cdk deploy` performs this step for you to make sure your latest code gets deployed\.

For full documentation of the `cdk` command, see [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.