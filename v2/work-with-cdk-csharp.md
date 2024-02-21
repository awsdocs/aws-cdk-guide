# Working with the AWS CDK in C\#<a name="work-with-cdk-csharp"></a>

\.NET is a fully\-supported client language for the AWS CDK and is considered stable\. C\# is the main \.NET language for which we provide examples and support\. You can choose to write AWS CDK applications in other \.NET languages, such as Visual Basic or F\#, but AWS offers limited support for using these languages with the CDK\.

You can develop AWS CDK applications in C\# using familiar tools including Visual Studio, Visual Studio Code, the `dotnet` command, and the NuGet package manager\. The modules comprising the AWS Construct Library are distributed via [nuget\.org](https://www.nuget.org/packages?q=amazon.cdk.aws)\.

We suggest using [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) \(any edition\) on Windows to develop AWS CDK apps in C\#\.

**Topics**
+ [Get started with C\#](#csharp-prerequisites)
+ [Creating a project](#csharp-newproject)
+ [Managing AWS Construct Library modules](#csharp-managemodules)
+ [AWS CDK idioms in C\#](#csharp-cdk-idioms)
+ [Building, synthesizing, and deploying](#csharp-running)

## Get started with C\#<a name="csharp-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [Getting started with the AWS CDK](getting_started.md)\.

C\# AWS CDK applications require \.NET Core v3\.1 or later, [available here](https://dotnet.microsoft.com/download/dotnet-core/3.1)\.

**Note**  
Third\-party language deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

The \.NET toolchain includes `dotnet`, a command\-line tool for building and running \.NET applications and managing NuGet packages\. Even if you work mainly in Visual Studio, this command can be useful for batch operations and for installing AWS Construct Library packages\.

## Creating a project<a name="csharp-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\. Use the `--language` option and specify `csharp`:

```
mkdir my-project
cd my-project
cdk init app --language csharp
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. Hyphens in the folder name are converted to underscores\. However, the name should otherwise follow the form of a C\# identifier; for example, it should not start with a number or contain spaces\. 

The resulting project includes a reference to the `Amazon.CDK.Lib` NuGet package\. It and its dependencies are installed automatically by NuGet\.

## Managing AWS Construct Library modules<a name="csharp-managemodules"></a>

The \.NET ecosystem uses the NuGet package manager\. The main CDK package, which contains the core classes and all stable service constructs, is `Amazon.CDK.Lib`\. Experimental modules, where new functionality is under active development, are named like `Amazon.CDK.AWS.SERVICE-NAME.Alpha`, where the service name is a short name without an AWS or Amazon prefix\. For example, the NuGet package name for the AWS IoT module is `Amazon.CDK.AWS.IoT.Alpha`\. If you can't find a package you want, [search Nuget\.org](https://www.nuget.org/packages?q=amazon.cdk.aws)\.

**Note**  
The [\.NET edition of the CDK API Reference](https://docs.aws.amazon.com/cdk/api/latest/dotnet/api/index.html) also shows the package names\.

Some services' AWS Construct Library support is in more than one module\. For example, AWS IoT has a second module named `Amazon.CDK.AWS.IoT.Actions.Alpha`\.

The AWS CDK's main module, which you'll need in most AWS CDK apps, is imported in C\# code as `Amazon.CDK`\. Modules for the various services in the AWS Construct Library live under `Amazon.CDK.AWS`\. For example, the Amazon S3 module's namespace is `Amazon.CDK.AWS.S3`\.

We recommend writing C\# `using` directives for the CDK core constructs and for each AWS service you use in each of your C\# source files\. You may find it convenient to use an alias for a namespace or type to help resolve name conflicts\. You can always use a type's fully\-qualfiied name \(including its namespace\) without a `using` statement\.

NuGet has four standard, mostly\-equivalent interfaces; you can use the one that suits your needs and working style\. You can also use compatible tools, such as [Paket](https://fsprojects.github.io/Paket/) or [MyGet](https://www.myget.org/)\.

### The Visual Studio NuGet GUI<a name="csharp-vs-nuget-gui"></a>

Visual Studio's NuGet tools are accessible from **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**\. Use the **Browse** tab to find the AWS Construct Library packages you want to install\. You can choose the desired version, including pre\-release versions of your modules and add them to any of the open projects\. 

**Note**  
All AWS Construct Library modules deemed "experimental" \(see [Versioning](reference.md#versioning)\) are flagged as pre\-release in NuGet and have an `alpha` name suffix\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/v2/guide/images/visual-studio-nuget.png)

Look on the **Updates** page to install new versions of your packages\.

### The NuGet console<a name="csharp-vs-nuget-console"></a>

The NuGet console is a PowerShell\-based interface to NuGet that works in the context of a Visual Studio project\. You can open it in Visual Studio by choosing **Tools** > **NuGet Package Manager** > **Package Manager Console**\. For more information about using this tool, see [Install and Manage Packages with the Package Manager Console in Visual Studio](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-powershell)\.

### The `dotnet` command<a name="csharp-vs-dotnet-command"></a>

The `dotnet` command is the primary command\-line tool for working with Visual Studio C\# projects\. You can invoke it from any Windows command prompt\. Among its many capabilities, `dotnet` can add NuGet dependencies to a Visual Studio project\.

Assuming you're in the same directory as the Visual Studio project \(`.csproj`\) file, issue a command like the following to install a package\. Note that since the main CDK library is included when you create a project, you should ever only need to explictly install experimental modules\. Experimental modules require you to specify an explicit version number\.

```
dotnet add package Amazon.CDK.AWS.IoT.Alpha -v VERSION-NUMBER
```

You may issue the command from another directory by including the path to the project file, or to the directory that contains it, after the `add` keyword\. The following example assumes that you are in your AWS CDK project's main directory\.

```
dotnet add src/PROJECT-DIR package Amazon.CDK.AWS.IoT.Alpha -v VERSION-NUMBER
```

To install a specific version of a package, include the `-v` flag and the desired version\.

To update a package, issue the same `dotnet add` command you used to install it\. For experimental modules, again, you must specify an explicit version number\.

For more information about managing packages using the `dotnet` command, see [Install and Manage Packages Using the dotnet CLI](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-dotnet-cli)\.

### The `nuget` command<a name="csharp-vs-nuget-command"></a>

The `nuget` command line tool can install and update NuGet packages\. However, it requires your Visual Studio project to be set up differently from the way `cdk init` sets up projects\. \(Technical details: `nuget` works with `Packages.config` projects, while `cdk init` creates a newer\-style `PackageReference` project\.\)

We do not recommend the use of the `nuget` tool with AWS CDK projects created by `cdk init`\. If you are using another type of project, and want to use `nuget`, see the [NuGet CLI Reference](https://docs.microsoft.com/en-us/nuget/reference/nuget-exe-cli-reference)\.

## AWS CDK idioms in C\#<a name="csharp-cdk-idioms"></a>

### Props<a name="csharp-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), an *id*, and *props*, a bundle of key/value pairs that the construct uses to configure the resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

In C\#, props are expressed using a props type\. In idiomatic C\# fashion, we can use an object initializer to set the various properties\. Here we're creating an Amazon S3 bucket using the `Bucket` construct; its corresponding props type is `BucketProps`\.

```
var bucket = new Bucket(this, "MyBucket", new BucketProps {
    Versioned = true
});
```

**Tip**  
Add the package `Amazon.JSII.Analyzers` to your project to get required\-values checking in your props definitions inside Visual Studio\.

When extending a class or overriding a method, you may want to accept additional props for your own purposes that are not understood by the parent class\. To do this, subclass the appropriate props type and add the new attributes\.

```
// extend BucketProps for use with MimeBucket
class MimeBucketProps : BucketProps {
    public string MimeType { get; set; }
}

// hypothetical bucket that enforces MIME type of objects inside it
class MimeBucket : Bucket {
     public MimeBucket( readonly Construct scope, readonly string id, readonly MimeBucketProps props=null) : base(scope, id, props) {
         // ...
     }
}

// instantiate our MimeBucket class 
var bucket = new MimeBucket(this, "MyBucket", new MimeBucketProps {
    Versioned = true,
    MimeType = "image/jpeg"
});
```

When calling the parent class's initializer or overridden method, you can generally pass the props you received\. The new type is compatible with its parent, and extra props you added are ignored\.

A future release of the AWS CDK could coincidentally add a new property with a name you used for your own property\. This won't cause any technical issues using your construct or method \(since your property isn't passed "up the chain," the parent class or overridden method will simply use a default value\) but it may cause confusion for your construct's users\. You can avoid this potential problem by naming your properties so they clearly belong to your construct\. If there are many new properties, bundle them into an appropriately\-named class and pass them as a single property\.

### Generic structures<a name="csharp-generic-structures"></a>

In some APIs, the AWS CDK uses JavaScript arrays or untyped objects as input to a method\. \(See, for example, AWS CodeBuild's [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_codebuild.BuildSpec.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_codebuild.BuildSpec.html) method\.\) In C\#, these objects are represented as `System.Collections.Generic.Dictionary<String, Object>`\. In cases where the values are all strings, you can use `Dictionary<String, String>`\. JavaScript arrays are represented as `object[]` or `string[]` array types in C\#\.

**Tip**  
You might define short aliases to make it easier to work with these specific dictionary types\.  

```
using StringDict = System.Collections.Generic.Dictionary<string, string>;
using ObjectDict = System.Collections.Generic.Dictionary<string, object>;
```

### Missing values<a name="csharp-missing-values"></a>

In C\#, missing values in AWS CDK objects such as props are represented by `null`\. The null\-conditional member access operator `?.` and the null coalescing operator `??` are convenient for working with these values\.

```
// mimeType is null if props is null or if props.MimeType is null
string mimeType = props?.MimeType;

// mimeType defaults to text/plain. either props or props.MimeType can be null
string MimeType = props?.MimeType ?? "text/plain";
```

## Building, synthesizing, and deploying<a name="csharp-running"></a>

The AWS CDK automatically compiles your app before running it\. However, it can be useful to build your app manually to check for errors and run tests\. You can do this by pressing F6 in Visual Studio or by issuing `dotnet build src` from the command line, where `src` is the directory in your project directory that contains the Visual Studio Solution \(`.sln`\) file\.

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