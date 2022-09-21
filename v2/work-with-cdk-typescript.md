# Working with the AWS CDK in TypeScript<a name="work-with-cdk-typescript"></a>

TypeScript is a fully\-supported client language for the AWS CDK and is considered stable\. Working with the AWS CDK in TypeScript uses familiar tools, including Microsoft's TypeScript compiler \(`tsc`\), [Node\.js](https://nodejs.org/) and the Node Package Manager \(`npm`\)\. You may also use [Yarn](https://yarnpkg.com/) if you prefer, though the examples in this Guide use NPM\. The modules comprising the AWS Construct Library are distributed via the NPM repository, [npmjs\.org](https://www.npmjs.com/)\.

You can use any editor or IDE; many AWS CDK developers use [Visual Studio Code](https://code.visualstudio.com/) \(or its open\-source equivalent [VSCodium](https://vscodium.com/)\), which has excellent support for TypeScript\.

## Prerequisites<a name="typescript-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

You also need TypeScript itself \(version 3\.8 or later\)\. If you don't already have it, you can install it using `npm`\.

```
npm install -g typescript
```

**Note**  
If you get a permission error, and have administrator access on your system, try `sudo npm install -g typescript`\.

Keep TypeScript up to date with a regular `npm update -g typescript`\.

**Note**  
Third\-party Language Deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

## Creating a project<a name="typescript-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language typescript
```

Creating a project also installs the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html) module and its dependencies\.

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. Hyphens in the folder name are converted to underscores\. However, the name should otherwise follow the form of a TypeScript identifier; for example, it should not start with a number or contain spaces\. 

## Using local `tsc` and `cdk`<a name="typescript-local"></a>

For the most part, this guide assumes you install TypeScript and the CDK Toolkit globally \(`npm install -g typescript aws-cdk`\), and the provided command examples \(such as `cdk synth`\) follow this assumption\. This approach makes it easy to keep both components up to date, and since both take a strict approach to backward compatibility, there is generally little risk in always using the latest versions\.

Some teams prefer to specify all dependencies within each project, including tools like the TypeScript compiler and the CDK Toolkit\. This practice lets you pin these components to specific versions and ensure that all developers on your team \(and your CI/CD environment\) use exactly those versions\. This eliminates a possible source of change, helping to make builds and deployments more consistent and repeatable\.

The CDK includes dependencies for both TypeScript and the CDK Toolkit in the TypeScript project template's `package.json`, so if you want to use this approach, you don't need to make any changes to your project\. All you need to do is use slightly different commands for building your app and for issuing `cdk` commands\. 

| Operation | Use global tools | Use local tools | 
| --- |--- |--- |
| Initialize project | `cdk init --language typescript` | `npx aws-cdk init --language typescript` | 
| --- |--- |--- |
| Build | `tsc` | `npm run build` | 
| --- |--- |--- |
| Run CDK Toolkit command | `cdk ...` | `npm run cdk ...` or `npx aws-cdk ...` | 
| --- |--- |--- |

`npx aws-cdk` runs the version of the CDK Toolkit installed locally in the current project, if one exists, falling back to the global installation, if any\. If no global installation exists, `npx` downloads a temporary copy of the CDK Toolkit and runs that\. You may specify an arbitrary version of the CDK Toolkit using the `@` syntax: `npx aws-cdk@2.0 --version` prints `2.0.0`\. 

**Tip**  
Set up an alias so you can use the `cdk` command with a local CDK Toolkit installation\.  

```
alias cdk="npx aws-cdk"
```

```
doskey cdk=npx aws-cdk $*
```

## Managing AWS Construct Library modules<a name="typescript-managemodules"></a>

Use the Node Package Manager \(`npm`\) to install and update AWS Construct Library modules for use by your apps, as well as other packages you need\. \(You may use `yarn` instead of `npm` if you prefer\.\) `npm` also installs the dependencies for those modules automatically\.

Most AWS CDK constructs are in the main CDK package, named `aws-cdk-lib`, which is a default dependency in new projects created by cdk init\. "Experimental" AWS Construct Library modules, where higher\-level constructs are still under development, are named like `@aws-cdk/SERVICE-NAME-alpha`\. The service name has an *aws\-* prefix\. If you're unsure of a module's name, [search for it on NPM](https://www.npmjs.com/search?q=%40aws-cdk)\.

**Note**  
The [CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) also shows the package names\.

For example, the command below installs the experimental module for AWS CodeStar\.

```
npm install @aws-cdk/aws-codestar-alpha
```

Some services' Construct Library support is in more than one namespace\. For example, besides `aws-route53`, there are three additional Amazon Route 53 namespaces, `aws-route53-targets`, `aws-route53-patterns`, and `aws-route53resolver`\.

Your project's dependencies are maintained in `package.json`\. You can edit this file to lock some or all of your dependencies to a specific version or to allow them to be updated to newer versions under certain criteria\. To update your project's NPM dependencies to the latest permitted version according to the rules you specified in `package.json`:

```
npm update
```

In TypeScript, you import modules into your code under the same name you use to install them using NPM\. We recommend the following practices when importing AWS CDK classes and AWS Construct Library modules in your applications\. Following these guidelines will help make your code consistent with other AWS CDK applications as well as easier to understand\.
+ Use ES6\-style `import` directives, not `require()`\.
+ Generally, import individual classes from `aws-cdk-lib`\.

  ```
  import { App, Stack } from 'aws-cdk-lib';
  ```
+ If you need many classes from `aws-cdk-lib`, you may use a namespace alias of `cdk` instead of importing the individual classes\. Avoid doing both\.

  ```
  import * as cdk from 'aws-cdk-lib';
  ```
+ Generally, import AWS service constructs using short namespace aliases\.

  ```
  import { aws_s3 as s3 } from 'aws-cdk-lib';
  ```

## AWS CDK idioms in TypeScript<a name="typescript-cdk-idioms"></a>

### Props<a name="typescript-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), an *id*, and *props*, a bundle of key/value pairs that the construct uses to configure the AWS resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

In TypeScript, the shape of `props` is defined using an interface that tells you the required and optional arguments and their types\. Such an interface is defined for each kind of `props` argument, usually specific to a single construct or method\. For example, the [Bucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) construct \(in the `aws-cdk-lib/aws-s3 module`\) specifies a `props` argument conforming to the [BucketProps](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.BucketProps.html) interface\. 

If a property is itself an object, for example the [websiteRedirect](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.BucketProps.html#websiteredirect) property of `BucketProps`, that object will have its own interface to which its shape must conform, in this case [RedirectTarget](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.RedirectTarget.html)\.

If you are subclassing an AWS Construct Library class \(or overriding a method that takes a props\-like argument\), you can inherit from the existing interface to create a new one that specifies any new props your code requires\. When calling the parent class or base method, generally you can pass the entire props argument you received, since any attributes provided in the object but not specified in the interface will be ignored\.

A future release of the AWS CDK could coincidentally add a new property with a name you used for your own property\. Passing the value you receive up the inheritance chain can then cause unexpected behavior\. It's safer to pass a shallow copy of the props you received with your property removed or set to `undefined`\. For example:

```
super(scope, name, {...props, encryptionKeys: undefined});
```

Alternatively, name your properties so that it is clear that they belong to your construct\. This way, it is unlikely they will collide with properties in future AWS CDK releases\. If there are many of them, use a single appropriately\-named object to hold them\.

### Missing values<a name="typescript-missing-values"></a>

Missing values in an object \(such as props\) have the value `undefined` in TypeScript\. Version 3\.7 of the language introduced operators that simplify working with these values, making it easier to specify defaults and "short\-circuit" chaining when an undefined value is reached\. For more information about these features, see the [TypeScript 3\.7 Release Notes](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html), specifically the first two features, Optional Chaining and Nullish Coalescing\.

## Building, synthesizing, and deploying<a name="typescript-running"></a>

Generally, you should be in the project's root directory when building and running your application\.

Node\.js cannot run TypeScript directly; instead, your application is converted to JavaScript using the TypeScript compiler, `tsc`\. The resulting JavaScript code is then executed\.

The AWS CDK automatically does this whenever it needs to run your app\. However, it can be useful to compile manually to check for errors and to run tests\. To compile your TypeScript app manually, issue `npm run build`\. You may also issue `npm run watch` to enter watch mode, in which the TypeScript compiler automatically rebuilds your app whenever you save changes to a source file\.

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