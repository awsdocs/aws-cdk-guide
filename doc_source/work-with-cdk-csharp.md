# Working with the AWS CDK in C\#<a name="work-with-cdk-csharp"></a>

\.NET is a fully\-supported client platform for the AWS CDK and is considered stable\. Our \.NET code examples are in C\#\. It is possible to write AWS CDK applications in other \.NET languages, such as Visual Basic or F\#, but we are unable to provide support for these languages\.

You can develop AWS CDK applications in C\# using familiar tools including Visual Studio, the `dotnet` command, and the NuGet package manager\. The modules comprising the AWS Construct Library are distributed via [nuget\.org](https://www.nuget.org/packages?q=amazon.cdk.aws)\.

We suggest using [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) \(any edition\) and the Microsoft \.NET Framework on Windows to develop AWS CDK apps in C\#\. You may use other tools \(for example, Mono on Linux or Mac OS X\), but our ability to provide instruction and support for these environments may be limited\.

## Prerequisites<a name="csharp-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

C\# AWS CDK applications require a \.NET Standard 2\.1 compatible implementation\. Suitable implementations include:
+ \.NET Core v3\.1 or later
+ \.NET Framework v4\.6\.1 or later
+ Mono v5\.4 or later on Linux or Mac OS X; [download here](https://www.mono-project.com/download/stable/)

If you have an up\-to\-date Windows 10 installation, you already have a suitable installation of \.NET Framework\.

The \.NET Standard toolchain includes `dotnet`, a command\-line tool for building and running \.NET applications and managing NuGet packages\. Even if you are using Visual Studio, this command is useful for batch operations and for installing AWS Construct Library packages\.

## Creating a project<a name="csharp-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language csharp
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. 

The resulting project includes a reference to the `Amazon.CDK` NuGet package\. It and its dependencies are installed automatically by NuGet\.

## Managing AWS construct library modules<a name="csharp-managemodules"></a>

The \.NET ecosystem uses the NuGet package manager\. AWS Construct Library modules are named like `Amazon.CDK.AWS.SERVICE-NAME`\. For example, the NuGet package name for the Amazon S3 module is `Amazon.CDK.AWS.S3`\.

**Note**  
All AWS Construct Library modules used in your project must be the same version\.

NuGet has four standard, mostly\-equivalent interfaces; you can use the one that suits your needs and working style\. You can also use compatible tools, such as [Paket](https://fsprojects.github.io/Paket/) or [MyGet](https://fsprojects.github.io/Paket/)\.

### The Visual Studio NuGet GUI<a name="csharp-vs-nuget-gui"></a>

Visual Studio's NuGet tools are accessible from **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution**\. Use the **Browse** tab to find the AWS Construct Library packages you want to install\. You can choose the desired version, including pre\-release versions \(mark the **Include prerelease** checkbox\) and add them to any of the open projects\. 

**Note**  
All AWS Construct Library modules deemed "experimental" \(see [Versioning](reference.md#versioning)\) are flagged as pre\-release in NuGet\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/visual-studio-nuget.png)

Look in the **Updates** panel to install new versions of your packages\.

### The NuGet console<a name="csharp-vs-nuget-console"></a>

The NuGet console is a PowerShell\-based interface to NuGet that works in the context of a Visual Studio project\. You can open it in Visual Studio by choosing **Tools** > **NuGet Package Manager** > **Package Manager Console**\. For more information on using this tool, see [Install and Manage Packages with the Package Manager Console in Visual Studio](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-powershell)\.

### The `dotnet` command<a name="csharp-vs-dotnet-command"></a>

The `dotnet` command is the primary command\-line tool for working with Visual Studio C\# projects\. You can invoke it from any Windows command prompt\. Among its many capabilities, `dotnet` can add NuGet dependencies to a Visual Studio project\.

Assuming you're in the same directory as the Visual Studio project \(`.csproj`\) file, issue a command like the following to install a package\.

```
dotnet add package Amazon.CDK.AWS.S3
```

You may issue the command from another directory by including the path to the project file, or to the directory that contains it, after the `add` keyword\. The following example assumes that you are in your AWS CDK project's main directory\.

```
dotnet add src/PROJECT-DIR package Amazon.CDK.AWS.S3
```

To install a specific version of a package, include the `-v` flag and the desired version\. AWS Construct Library modules that are deemed "experimental" \(see [Versioning](reference.md#versioning)\) are flagged as pre\-release in NuGet, and must be installed using an explicit version number\.

```
dotnet add package Amazon.CDK.AWS.S3 -v VERSION-NUMBER
```

To update a package, issue the same `dotnet add` command you used to install it\. If you do not specify a version number, the latest version is installed\. For experimental modules, again, you must specify an explicit version number\.

For more information on managing packages using the `dotnet` command, see [Install and Manage Packages Using the dotnet CLI](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-dotnet-cli)\.

### The `nuget` command<a name="csharp-vs-nuget-command"></a>

The `nuget` command line tool can install and update NuGet packages\. However, it requires your Visual Studio project to be set up differently from the way `cdk init` sets up projects\. \(Technical details: `nuget` works with `Packages.config` projects, while `cdk init` creates a newer\-style `PackageReference` project\.\)

We do not recommend the use of the `nuget` tool with AWS CDK projects created by `cdk init`\. If you are using another type of project, and want to use `nuget`, see the [NuGet CLI Reference](https://docs.microsoft.com/en-us/nuget/reference/nuget-exe-cli-reference)\.

## AWS CDK idioms in C\#<a name="csharp-cdk-idioms"></a>

### Props<a name="csharp-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), a *name*, and *props*, a bundle of key/value pairs that the construct uses to configure the resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

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
     public MimeBucket(final Construct scope, final string id, final MimeBucketProps props=null) : base(scope, id, props) {
         // ...
     }
}

// instantiate our MyBucket class 
var bucket = new MyBucket(this, "MyBucket", new MimeBucketProps {
    Versioned = true,
    MimeType = "image/jpeg"
});
```

When calling the parent class's initializer or overridden method, you can generally pass the props you received\. The new type is compatible with its parent, and extra props you added are ignored\.

Keep in mind that future releases of the AWS CDK may coincidentally add a new property with a name you used for your own property\. This won't cause any technical issues using your construct or method \(since your property isn't passed "up the chain," the parent class or overridden method will simply use a default value\) but it may cause confusion for your construct's users\. You can avoid this potential problem by naming your properties so they clearly belong to your construct \(e\.g\. `BobEncryption` rather than just `encryption`, assuming you're Bob\)\. If there are many new properties, bundle them into an appropriately\-named class \(`BobBucketPoperties`?\) and pass them as a single property\.

### Generic structures<a name="csharp-generic-structures"></a>

In some places, the AWS CDK uses JavaScript arrays or untyped objects or as input to a method\. \(See, for example, AWS CodeBuild's [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-codebuild.BuildSpec.html#to-wbr-build-wbr-spec](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-codebuild.BuildSpec.html#to-wbr-build-wbr-spec) method\.\) In C\#, objects are represented as `System.Collections.Generic.Dictionary<String, Object>`\. In cases where the values are all strings, you can use `Dictionary<String, String>`\. JavaScript arrays are represented as `object[]` or `string[]` in C\#\.

### Missing values<a name="csharp-missing-values"></a>

In C\#, missing values in AWS CDK objects such as props are represented by `null`\. The null\-conditional member access operator `?.` and the null coalescing operator `??` are convenient for working with these values\.

```
// mimeType is null if props is null or if props.MimeType is null
string mimeType = props?.MimeType;

// mimeType defaults to text/plain. either props or props.MimeType can be null
string MimeType = props?.MimeType ?? "text/plain";
```

## Building, synthesizing, and deploying<a name="csharp-running"></a>

Before running, build \(compile\) the app by pressing F6 in Visual Studio or by issuing `dotnet build src` from the command line, where `src` is the directory in your project directory that contains the Visual Studio Solution \(`.sln`\) file\.

The build step reports any syntax or type errors in your code\. Once you can build your application without errors, you're ready to synthesize or deploy\.

The [stacks](stacks.md) defined in your AWS CDK app can be deployed individually or together using the commands below\. Generally, you should be in your project's main directory when you issue them\.
+ `cdk synth`: Synthesizes a AWS CloudFormation template from one or more of the stacks in your AWS CDK app\.
+ `cdk deploy`: Deploys the resources defined by one or more of the stacks in your AWS CDK app to AWS\.

You can specify the names of multiple stacks to be synthesized or deployed in a single command\. If your app defines only one stack, its name is assumed and you do not need to specify it\.

**Tip**  
You don't need to explicitly synthesize stacks before deploying them; `cdk deploy` performs this step for you\.

For full documentation of the `cdk` command, see [AWS CDK Toolkit \(`cdk`\)](tools.md#cli)\.