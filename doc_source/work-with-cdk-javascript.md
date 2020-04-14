# Working with the AWS CDK in JavaScript<a name="work-with-cdk-javascript"></a>

JavaScript is a fully\-supported client language for the AWS CDK and is considered stable\. Working with the AWS CDK in JavaScript uses familiar tools, including [Node\.js](https://nodejs.org/) and the Node Package Manager \(`npm`\)\. You may also use [Yarn](https://yarnpkg.com/) if you prefer, though the examples in this Guide use NPM\. The modules comprising the AWS Construct Library are distributed via the NPM repository, [npmjs\.org](https://www.npmjs.com/)\.

You can use any editor or IDE; many AWS CDK developers use [Visual Studio Code](https://code.visualstudio.com/) \(or its open\-source equivalent [VSCodium](https://github.com/VSCodium/vscodium)\), which has good support for JavaScript\.

## Prerequisites<a name="javascript-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

JavaScript AWS CDK applications require no additional prerequisites beyond these\.

## Creating a Project<a name="javascript-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language javascript
```

Creating a project also installs the [https://docs.aws.amazon.com/cdk/api/latest/docs/core-readme.html](https://docs.aws.amazon.com/cdk/api/latest/docs/core-readme.html) module and its dependencies\.

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. 

## Managing AWS Construct Library Modules<a name="javascript-managemodules"></a>

Use the Node Package Manager \(`npm`\), included with Node\.js, to install and update AWS Construct Library modules for use by your apps, as well as other packages you need\. \(You may use `yarn` instead of `npm` if you prefer\.\) `npm` also installs the dependencies for those modules automatically\.

AWS Construct Library modules are named like `@aws-cdk/SERVICE-NAME`\. For example, the command below installs the modules for Amazon S3 and AWS Lambda\.

```
npm install @aws-cdk/aws-s3 @aws-cdk/aws-lambda
```

Your project's dependencies are maintained in `package.json`\. You can edit this file to lock some or all of your dependencies to a specific version or to allow them to be updated to newer versions under certain criteria\. To update your project's dependencies:

```
npm update
```

**Note**  
All AWS Construct Library modules used in your project must be the same version\.

## AWS CDK Idioms in JavaScript<a name="javascript-cdk-idioms"></a>

### Props<a name="javascript-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), a *name*, and *props*, a bundle of key/value pairs that the construct uses to configure the AWS resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

Using an IDE or editor that has good JavaScript autocomplete will help avoid misspelling property names\. If a construct is expecting an `encryptionKeys` property, and you spell it `encryptionkeys`, when instantiating the construct, you haven't passed the value you intended\. This can cause an error at synthesis time if the property is required, or cause the property to be silently ignored if it is optional\. In the latter case, you may get a default behavior you intended to override\. Take special care here\.

When subclassing an AWS Construct Library class \(or overriding a method that takes a props\-like argument\), you may want to accept additional properties for your own use\. These values will be ignored by the parent class or overridden method, because they are never accessed in that code, so you can generally pass on all the props you received\.

However, we do occasionally add properties to constructs\. If a property we add in a later version happens to have the same name as one you're accepting, passing it up the chain can cause unexpected behavior\. It's safer to pass a shallow copy of the props you received with your property removed or set to undefined\. For example:

```
super(scope, name, {...props, encryptionKeys: undefined});
```

Alternatively, name your properties so that it is clear that they belong to your construct\. This way, it is unlikely they will collide with properties in future AWS CDK releases\. If there are many of them, use a single appropriately\-named object to hold them\.

### Missing Values<a name="javascript-missing-values"></a>

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

A version of the ECMAScript standard currently in development specifies new operators that will simplify the handling of undefined values\. Using them can simplify your code, but you will need a new version of Node\.js to use them\. For more information, see the [optional chaining](https://github.com/tc39/proposal-optional-chaining/blob/master/README.md) and [nullish coalescing](https://github.com/tc39/proposal-nullish-coalescing/blob/master/README.md) proposals\.

## Synthesizing and Deploying<a name="javascript-running"></a>

The [stacks](stacks.md) defined in your AWS CDK app can be deployed individually or together using the commands below\. Generally, you should be in your project's main directory when you issue them\.
+ `cdk synth`: Synthesizes a AWS CloudFormation template from one or more of the stacks in your AWS CDK app\.
+ `cdk deploy`: Deploys the resources defined by one or more of the stacks in your AWS CDK app to AWS\.

You can specify the names of multiple stacks to be synthesized or deployed in a single command\. If your app defines only one stack, its name is assumed and you do not need to specify it\.

**Tip**  
You don't need to explicitly synthesize stacks before deploying them; `cdk deploy` performs this step for you\.

For full documentation of the `cdk` command, see [AWS CDK Toolkit \(`cdk`\)](tools.md#cli)\.

## Using TypeScript Examples with JavaScript<a name="javascript-using-typescript-examples"></a>

[TypeScript](https://www.typescriptlang.org/) is the language we use to develop the AWS CDK, and it was the first language supported for developing applications, so many available AWS CDK code examples are written in TypeScript\. These code examples can be a good resource for JavaScript developers; you just need to remove the TypeScript\-specific parts of the code\. The most commonly\-used features are type annotations and interface declarations\.

Type annotations may be provided for variables, class members, function parameters, and function return types\. For variables, parameters, and members, types are specified by following the identifier with a colon and the type\. Function return values follow the function signature and consist of a colon and the type\.

```
var encrypted: boolean = true;

class myStack extends core.Stack {
    bucket: s3.Bucket;
    // ...
}

function makeEnv(account: string, region: string) : object {
    // ...
}
```

To convert type\-annotated code to JavaScript, remove the colon and the type\. Class members must have some value in JavaScript; remove them entirely if they only have a type annotation in TypeScript\.

```
var encrypted = true;

class myStack extends core.Stack {
    // ...
}

function makeEnv(account, region) {
    // ...
}
```

In TypeScript, interfaces are used to give bundles of required and optional properties, and their types, a name\. You can then use the interface name as a type annotation\. TypeScript will make sure that the object you use as, for example, an argument to a function has the required properties of the right types\.

```
interface myFuncProps {
    code: lambda.Code,
    handler?: string
}
```

JavaScript does not have an interface feature, so once you've removed the type annotations, delete the interface declarations as well\.

Finally, TypeScript supports the access modifiers `public`, `protected`, and `private` for members of classes\. All class members in JavaScript are public\. Simply remove these modifiers wherever you see them\.

Knowing how to identify and remove these three TypeScript features goes a long way toward adapting short TypeScript snippets to JavaScript\. But it may be impractical to convert longer TypeScript examples in this fashion, since they are more likely to use TypeScript features you're not familiar with\. For these situations, we recommend [Babel](https://babeljs.io/) with the [TypeScript plug\-in](https://babeljs.io/docs/en/babel-plugin-transform-typescript)\. Babel won't complain if code uses an undefined variable, for example, as `tsc` would\. If it is syntactically valid, then with few exceptions, Babel can translate it to JavaScript\. This makes Babel particularly valuable for converting snippets that may not be runnable in on their own\.

## Migrating to TypeScript<a name="javascript-to-typescript"></a>

As their projects get larger and more complex, many JavaScript developers move to [TypeScript](https://www.typescriptlang.org/)\. TypeScript is a superset of JavaScript—all JavaScript code is valid TypeScript code, so no changes to your code are required—and it is also a supported AWS CDK language\. Type annotations and other TypeScript features are optional and can be added to your AWS CDK app as you find value in them\. TypeScript also gives you early access to new JavaScript features, such as optional chaining and nullish coalescing, before they're finalized—and without requiring that you upgrade Node\.js\.

TypeScript's "shape\-based" interfaces, which define bundles of required and optional properties \(and their types\) within an object, allow common mistakes to be caught while you're writing the code, and make it easier for your IDE to provide robust autocomplete and other real\-time coding advice\.

Coding in TypeScript does involve an additional step: compiling your app with the TypeScript compiler, `tsc`\. This step can happen automatically whenever you save your source code, or before you run your app\. For typical AWS CDK apps, compilation requires a few seconds at most\.

The easiest way to migrate an existing JavaScript AWS CDK app to TypeScript is to create a new TypeScript project using `cdk init app --language typescript`, then copy your source files \(and any other necessary files, such as assets like AWS Lambda function source code\) to the new project\. Rename your JavaScript files to end in `.ts` and begin developing in TypeScript\.