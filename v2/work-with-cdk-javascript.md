# Working with the AWS CDK in JavaScript<a name="work-with-cdk-javascript"></a>

JavaScript is a fully\-supported client language for the AWS CDK and is considered stable\. Working with the AWS CDK in JavaScript uses familiar tools, including [Node\.js](https://nodejs.org/) and the Node Package Manager \(`npm`\)\. You may also use [Yarn](https://yarnpkg.com/) if you prefer, though the examples in this Guide use NPM\. The modules comprising the AWS Construct Library are distributed via the NPM repository, [npmjs\.org](https://www.npmjs.com/)\.

You can use any editor or IDE; many AWS CDK developers use [Visual Studio Code](https://code.visualstudio.com/) \(or its open\-source equivalent [VSCodium](https://vscodium.com/)\), which has good support for JavaScript\.

## Prerequisites<a name="javascript-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

JavaScript AWS CDK applications require no additional prerequisites beyond these\.

**Note**  
Third\-party Language Deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

## Creating a project<a name="javascript-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language javascript
```

Creating a project also installs the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html) module and its dependencies\.

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. Hyphens in the folder name are converted to underscores\. However, the name should otherwise follow the form of a JavaScript identifier; for example, it should not start with a number or contain spaces\. 

## Using local `cdk`<a name="typescript-local"></a>

For the most part, this guide assumes you install the CDK Toolkit globally \(`npm install -g aws-cdk`\), and the provided command examples \(such as `cdk synth`\) follow this assumption\. This approach makes it easy to keep the CDK Toolkit up to date, and since the CDK takes a strict approach to backward compatibility, there is generally little risk in always using the latest version\.

Some teams prefer to specify all dependencies within each project, including tools like the CDK Toolkit\. This practice lets you pin such components to specific versions and ensure that all developers on your team \(and your CI/CD environment\) use exactly those versions\. This eliminates a possible source of change, helping to make builds and deployments more consistent and repeatable\.

The CDK includes a dependency for the CDK Toolkit in the JavaScript project template's `package.json`, so if you want to use this approach, you don't need to make any changes to your project\. All you need to do is use slightly different commands for building your app and for issuing `cdk` commands\. 

| Operation | Use global CDK Toolkit | Use local CDK Toolkit | 
| --- |--- |--- |
| Initialize project | `cdk init --language javascript` | `npx aws-cdk init --language javascript` | 
| --- |--- |--- |
| Run CDK Toolkit command | `cdk ...` | `npm run cdk ...` or `npx aws-cdk ...` | 
| --- |--- |--- |

`npx aws-cdk` runs the version of the CDK Toolkit installed locally in the current project, if one exists, falling back to the global installation, if any\. If no global installation exists, `npx` downloads a temporary copy of the CDK Toolkit and runs that\. You may specify an arbitrary version of the CDK Toolkit using the `@` syntax: `npx aws-cdk@1.120 --version` prints `1.120.0`\. 

**Tip**  
Set up an alias so you can use the `cdk` command with a local CDK Toolkit installation\.  

```
alias cdk="npx aws-cdk"
```

```
doskey cdk=npx aws-cdk $*
```

## Managing AWS Construct Library modules<a name="javascript-managemodules"></a>

Use the Node Package Manager \(`npm`\) to install and update AWS Construct Library modules for use by your apps, as well as other packages you need\. \(You may use `yarn` instead of `npm` if you prefer\.\) `npm` also installs the dependencies for those modules automatically\.

Most AWS CDK constructs are in the main CDK package, named `aws-cdk-lib`, which is a default dependency in new projects created by cdk init\. "Experimental" AWS Construct Library modules, where higher\-level constructs are still under development, are named like `aws-cdk-lib/SERVICE-NAME-alpha`\. The service name has an *aws\-* prefix\. If you're unsure of a module's name, [search for it on NPM](https://www.npmjs.com/search?q=%40aws-cdk)\.

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

In JavaScript, you import modules into your code under the same name you use to install them using NPM\. We recommend the following practices when importing AWS CDK classes and AWS Construct Library modules in your applications\. Following these guidelines will help make your code consistent with other AWS CDK applications as well as easier to understand\.
+ Use `require()`, not ES6\-style `import` directives\. Older versions of Node\.js do not support ES6 imports, so using the older syntax is more widely compatible\. \(If you really want to use ES6 imports, use [esm](https://www.npmjs.com/package/esm) to ensure your project is compatible with all supported versions of Node\.js\.\)
+ Generally, import individual classes from `aws-cdk-lib`\.

  ```
  const { App, Stack } = require('aws-cdk-lib');
  ```
+ If you need many classes from `aws-cdk-lib`, you may use a namespace alias of `cdk` instead of importing the individual classes\. Avoid doing both\.

  ```
  const cdk = require('aws-cdk-lib');
  ```
+ Generally, import AWS Construct Libraries using short namespace aliases\.

  ```
  const { s3 } = require('aws-cdk-lib/aws-s3');
  ```

## AWS CDK idioms in JavaScript<a name="javascript-cdk-idioms"></a>

### Props<a name="javascript-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), an *id*, and *props*, a bundle of key/value pairs that the construct uses to configure the AWS resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

Using an IDE or editor that has good JavaScript autocomplete will help avoid misspelling property names\. If a construct is expecting an `encryptionKeys` property, and you spell it `encryptionkeys`, when instantiating the construct, you haven't passed the value you intended\. This can cause an error at synthesis time if the property is required, or cause the property to be silently ignored if it is optional\. In the latter case, you may get a default behavior you intended to override\. Take special care here\.

When subclassing an AWS Construct Library class \(or overriding a method that takes a props\-like argument\), you may want to accept additional properties for your own use\. These values will be ignored by the parent class or overridden method, because they are never accessed in that code, so you can generally pass on all the props you received\.

A future release of the AWS CDK could coincidentally add a new property with a name you used for your own property\. Passing the value you receive up the inheritance chain can then cause unexpected behavior\. It's safer to pass a shallow copy of the props you received with your property removed or set to `undefined`\. For example:

```
super(scope, name, {...props, encryptionKeys: undefined});
```

Alternatively, name your properties so that it is clear that they belong to your construct\. This way, it is unlikely they will collide with properties in future AWS CDK releases\. If there are many of them, use a single appropriately\-named object to hold them\.

### Missing values<a name="javascript-missing-values"></a>

Missing values in an object \(such as `props`\) have the value `undefined` in JavaScript\. The usual techniques apply for dealing with these\. For example, a common idiom for accessing a property of a value that may be undefined is as follows:

```
// a may be undefined, but if it is not, it may have an attribute b
// c is undefined if a is undefined, OR if a doesn't have an attribute b
let c = a && a.b;
```

However, if `a` could have some other "falsy" value besides `undefined`, it is better to make the test more explicit\. Here, we'll take advantage of the fact that `null` and `undefined` are equal to test for them both at once:

```
let c = a == null ? a : a.b;
```

**Tip**  
Node\.js 14\.0 and later support new operators that can simplify the handling of undefined values\. For more information, see the [optional chaining](https://github.com/tc39/proposal-optional-chaining/blob/master/README.md) and [nullish coalescing](https://github.com/tc39/proposal-nullish-coalescing/blob/master/README.md) proposals\.

## Synthesizing and deploying<a name="javascript-running"></a>

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

## Using TypeScript examples with JavaScript<a name="javascript-using-typescript-examples"></a>

[TypeScript](https://www.typescriptlang.org/) is the language we use to develop the AWS CDK, and it was the first language supported for developing applications, so many available AWS CDK code examples are written in TypeScript\. These code examples can be a good resource for JavaScript developers; you just need to remove the TypeScript\-specific parts of the code\.

TypeScript snippets often use the newer ECMAScript `import` and `export` keywords to import objects from other modules and to declare the objects to be made available outside the current module\. Node\.js has just begun supporting these keywords in its latest releases\. Depending on the version of Node\.js you're using \(or wish to support\), you might rewrite imports and exports to use the older syntax\.

Imports can be replaced with calls to the `require()` function\.

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import { Bucket, BucketPolicy } from 'aws-cdk-lib/aws-s3';
```

------
#### [ JavaScript ]

```
const cdk = require('aws-cdk-lib');
const { Bucket, BucketPolicy } = require('aws-cdk-lib/aws-s3');
```

------

Exports can be assigned to the `module.exports` object\.

------
#### [ TypeScript ]

```
export class Stack1 extends cdk.Stack {
  // ...
}

export class Stack2 extends cdk.Stack {
  // ...
}
```

------
#### [ JavaScript ]

```
class Stack1 extends cdk.Stack {
  // ...
}

class Stack2 extends cdk.Stack {
  // ...
}

module.exports = { Stack1, Stack2 }
```

------

**Note**  
An alternative to using the old\-style imports and exports is to use the [https://www.npmjs.com/package/esm](https://www.npmjs.com/package/esm) module\.

Once you've got the imports and exports sorted, you can dig into the actual code\. You may run into these commonly\-used TypeScript features:
+ Type annotations
+ Interface definitions
+ Type conversions/casts
+ Access modifiers

Type annotations may be provided for variables, class members, function parameters, and function return types\. For variables, parameters, and members, types are specified by following the identifier with a colon and the type\. Function return values follow the function signature and consist of a colon and the type\.

To convert type\-annotated code to JavaScript, remove the colon and the type\. Class members must have some value in JavaScript; set them to `undefined` if they only have a type annotation in TypeScript\.

------
#### [ TypeScript ]

```
var encrypted: boolean = true;

class myStack extends cdk.Stack {
    bucket: s3.Bucket;
    // ...
}

function makeEnv(account: string, region: string) : object {
    // ...
}
```

------
#### [ JavaScript ]

```
var encrypted = true;

class myStack extends cdk.Stack {
    bucket = undefined;
    // ...
}

function makeEnv(account, region) {
    // ...
}
```

------

In TypeScript, interfaces are used to give bundles of required and optional properties, and their types, a name\. You can then use the interface name as a type annotation\. TypeScript will make sure that the object you use as, for example, an argument to a function has the required properties of the right types\.

```
interface myFuncProps {
    code: lambda.Code,
    handler?: string
}
```

JavaScript does not have an interface feature, so once you've removed the type annotations, delete the interface declarations entirely\.

When a function or method returns a general\-purpose type \(such as `object`\), but you want to treat that value as a more specific child type to access properties or methods that are not part of the more general type's interface, TypeScript lets you *cast* the value using `as` followed by a type or interface name\. JavaScript doesn't support \(or need\) this, so simply remove `as` and the following identifier\. A less\-common cast syntax is to use a type name in brackets, `<LikeThis>`; these casts, too, must be removed\.

Finally, TypeScript supports the access modifiers `public`, `protected`, and `private` for members of classes\. All class members in JavaScript are public\. Simply remove these modifiers wherever you see them\.

Knowing how to identify and remove these TypeScript features goes a long way toward adapting short TypeScript snippets to JavaScript\. But it may be impractical to convert longer TypeScript examples in this fashion, since they are more likely to use other TypeScript features\. For these situations, we recommend [Sucrase](https://github.com/alangpierce/sucrase)\. Sucrase won't complain if code uses an undefined variable, for example, as `tsc` would\. If it is syntactically valid, then with few exceptions, Sucrase can translate it to JavaScript\. This makes it particularly valuable for converting snippets that may not be runnable on their own\.

## Migrating to TypeScript<a name="javascript-to-typescript"></a>

Many JavaScript developers move to [TypeScript](https://www.typescriptlang.org/) as their projects get larger and more complex\. TypeScript is a superset of JavaScript—all JavaScript code is valid TypeScript code, so no changes to your code are required—and it is also a supported AWS CDK language\. Type annotations and other TypeScript features are optional and can be added to your AWS CDK app as you find value in them\. TypeScript also gives you early access to new JavaScript features, such as optional chaining and nullish coalescing, before they're finalized—and without requiring that you upgrade Node\.js\.

TypeScript's "shape\-based" interfaces, which define bundles of required and optional properties \(and their types\) within an object, allow common mistakes to be caught while you're writing the code, and make it easier for your IDE to provide robust autocomplete and other real\-time coding advice\.

Coding in TypeScript does involve an additional step: compiling your app with the TypeScript compiler, `tsc`\. For typical AWS CDK apps, compilation requires a few seconds at most\.

The easiest way to migrate an existing JavaScript AWS CDK app to TypeScript is to create a new TypeScript project using `cdk init app --language typescript`, then copy your source files \(and any other necessary files, such as assets like AWS Lambda function source code\) to the new project\. Rename your JavaScript files to end in `.ts` and begin developing in TypeScript\.