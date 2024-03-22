# AWS CDK apps<a name="apps"></a>

The AWS Cloud Development Kit \(AWS CDK\) application or *app* is a collection of one or more CDK [stacks](stacks.md)\. Stacks are a collection of one or more [constructs](constructs.md), which define AWS resources and properties\. Therefore, the overall grouping of your stacks and constructs are known as your CDK app\.

**Topics**
+ [Defining apps](#apps-define)
+ [The construct tree](#apps-tree)
+ [The app lifecycle](#lifecycle)

## Defining apps<a name="apps-define"></a>

You create an app by defining an app instance in the application file of your [project](projects.md)\. To do this, you import and use the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html) construct from the AWS Construct Library\. The `App` construct doesn't require any initialization arguments\. It is the only construct that can be used as the root\.

The `[App](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.App.html)` and `[Stack](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html)` classes from the AWS Construct Library are unique constructs\. Compared to other constructs, they don't configure AWS resources on their own\. Instead, they are used to provide context for your other constructs\. All constructs that represent AWS resources must be defined, directly or indirectly, within the scope of a `Stack` construct\. `Stack` constructs are defined within the scope of an `App` construct\.

Apps are then synthesized to create AWS CloudFormation templates for your stacks\. The following is an example:

------
#### [ TypeScript ]

```
const app = new App();
new MyFirstStack(app, 'hello-cdk');
app.synth();
```

------
#### [ JavaScript ]

```
const app = new App();
new MyFirstStack(app, 'hello-cdk');
app.synth();
```

------
#### [ Python ]

```
app = App()
MyFirstStack(app, "hello-cdk")
app.synth()
```

------
#### [ Java ]

```
App app = new App();
new MyFirstStack(app, "hello-cdk");
app.synth();
```

------
#### [ C\# ]

```
var app = new App();
new MyFirstStack(app, "hello-cdk");
app.Synth();
```

------
#### [ Go ]

```
app := awscdk.NewApp(nil)
            
MyFirstStack(app, "MyFirstStack", &MyFirstStackProps{
  awscdk.StackProps{
    Env: env(),
  },
})

app.Synth(nil)
```

------

Stacks within a single app can easily refer to each other's resources and properties\. The AWS CDK infers dependencies between stacks so that they can be deployed in the correct order\. You can deploy any or all of the stacks within an app with a single `cdk deploy` command\.

## The construct tree<a name="apps-tree"></a>

Constructs are defined inside of other constructs using the `scope` argument that is passed to every construct, with the `App` class as the root\. In this way, an AWS CDK app defines a hierarchy of constructs known as the *construct tree*\.

The root of this tree is your app, which is an instance of the `App` class\. Within the app, you instantiate one or more stacks\. Within stacks, you instantiate constructs, which may themselves instantiate resources or other constructs, and so on down the tree\.

Constructs are *always* explicitly defined within the scope of another construct, which creates relationships between constructs\. Almost always, you should pass `this` \(in Python, `self`\) as the scope, indicating that the new construct is a child of the current construct\. The intended pattern is that you derive your construct from [https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html), then instantiate the constructs it uses in its constructor\.

Passing the scope explicitly allows each construct to add itself to the tree, with this behavior entirely contained within the [`Construct` base class](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Construct.html)\. It works the same way in every language supported by the AWS CDK and does not require additional customization\.

**Important**  
Technically, it's possible to pass some scope other than `this` when instantiating a construct\. You can add constructs anywhere in the tree, or even in another stack in the same app\. For example, you could write a mixin\-style function that adds constructs to a scope passed in as an argument\. The practical difficulty here is that you can't easily ensure that the IDs you choose for your constructs are unique within someone else's scope\. The practice also makes your code more difficult to understand, maintain, and reuse\. It is almost always better to find a way to express your intent without resorting to abusing the `scope` argument\.

The AWS CDK uses the IDs of all constructs in the path from the tree's root to each child construct to generate the unique IDs required by AWS CloudFormation\. This approach means that construct IDs only need to be unique within their scope, rather than within the entire stack as in native AWS CloudFormation\. However, if you move a construct to a different scope, its generated stack\-unique ID changes, and AWS CloudFormation won't consider it the same resource\.

The construct tree is separate from the constructs that you define in your AWS CDK code\. However, it's accessible through any construct's `node` attribute, which is a reference to the node that represents that construct in the tree\. Each node is a [https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Node.html](https://docs.aws.amazon.com/cdk/api/v2/docs/constructs.Node.html) instance, the attributes of which provide access to the tree's root and to the node's parent scopes and children\.

1. `node.children` – The direct children of the construct\.

1. `node.id` – The identifier of the construct within its scope\.

1. `node.path` – The full path of the construct including the IDs of all of its parents\.

1. `node.root` – The root of the construct tree \(the app\)\.

1. `node.scope` – The scope \(parent\) of the construct, or undefined if the node is the root\.

1. `node.scopes` – All parents of the construct, up to the root\.

1. `node.uniqueId` – The unique alphanumeric identifier for this construct within the tree \(by default, generated from `node.path` and a hash\)\.

The construct tree defines an implicit order in which constructs are synthesized to resources in the final AWS CloudFormation template\. Where one resource must be created before another, AWS CloudFormation or the AWS Construct Library generally infers the dependency\. They then make sure that the resources are created in the right order\.

You can also add an explicit dependency between two nodes by using `node.addDependency()`\. For more information, see [Dependencies](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib-readme.html#dependencies) in the *AWS CDK API Reference*\.

The AWS CDK provides a simple way to visit every node in the construct tree and perform an operation on each one\. For more information, see [Aspects](aspects.md)\.

## The app lifecycle<a name="lifecycle"></a>

When you deploy your CDK app, the following phases take place\. This is known as the *app lifecycle*:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/v2/guide/images/Lifecycle.png)

An AWS CDK app goes through the following phases in its lifecycle\.
+ **Construction \(or Initialization\)** – Your code instantiates all of the defined constructs and then links them together\. In this stage, all of the constructs \(app, stacks, and their child constructs\) are instantiated and the constructor chain is executed\. Most of your app code is executed in this stage\.
+ **Preparation** – All constructs that have implemented the `prepare` method participate in a final round of modifications, to set up their final state\. The preparation phase happens automatically\. As a user, you don't see any feedback from this phase\. It's rare to need to use the "prepare" hook, and generally not recommended\. Be very careful when mutating the construct tree during this phase, because the order of operations could impact behavior\.
+ **Validation** – All constructs that have implemented the `validate` method can validate themselves to ensure that they're in a state that will correctly deploy\. You will get notified of any validation failures that happen during this phase\. Generally, we recommend performing validation as soon as possible \(usually as soon as you get some input\) and throwing exceptions as early as possible\. Performing validation early improves reliability as stack traces will be more accurate, and ensures that your code can continue to execute safely\.
+ **Synthesis** – This is the final stage of the execution of your AWS CDK app\. It's triggered by a call to `app.synth()`, and it traverses the construct tree and invokes the `synthesize` method on all constructs\. Constructs that implement `synthesize` can participate in synthesis and emit deployment artifacts to the resulting cloud assembly\. These artifacts include AWS CloudFormation templates, AWS Lambda application bundles, file and Docker image assets, and other deployment artifacts\. [Cloud assemblies](#apps_cloud_assembly) describes the output of this phase\. In most cases, you won't need to implement the `synthesize` method\.
+ **Deployment** – In this phase, the AWS CDK CLI takes the deployment artifacts cloud assembly produced by the synthesis phase and deploys it to an AWS environment\. It uploads assets to Amazon S3 and Amazon ECR, or wherever they need to go\. Then, it starts an AWS CloudFormation deployment to deploy the application and create the resources\.

By the time the AWS CloudFormation deployment phase starts, your AWS CDK app has already finished and exited\. This has the following implications:
+ The AWS CDK app can't respond to events that happen during deployment, such as a resource being created or the whole deployment finishing\. To run code during the deployment phase, you must inject it into the AWS CloudFormation template as a [custom resource](cfn_layer.md#develop-customize-custom)\. For more information about adding a custom resource to your app, see the [AWS CloudFormation module](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cloudformation-readme.html), or the [custom\-resource](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/custom-resource/) example\.
+ The AWS CDK app might have to work with values that can't be known at the time it runs\. For example, if the AWS CDK app defines an Amazon S3 bucket with an automatically generated name, and you retrieve the `bucket.bucketName` \(Python: `bucket_name`\) attribute, that value is not the name of the deployed bucket\. Instead, you get a `Token` value\. To determine whether a particular value is available, call `cdk.isUnresolved(value)` \(Python: `is_unresolved`\)\. See [Tokens](tokens.md) for details\.

### Cloud assemblies<a name="apps_cloud_assembly"></a>

The call to `app.synth()` is what tells the AWS CDK to synthesize a cloud assembly from an app\. Typically you don't interact directly with cloud assemblies\. They are files that include everything needed to deploy your app to a cloud environment\. For example, it includes an AWS CloudFormation template for each stack in your app\. It also includes a copy of any file assets or Docker images that you reference in your app\.

See the [cloud assembly specification](https://github.com/aws/aws-cdk/blob/master/design/cloud-assembly.md) for details on how cloud assemblies are formatted\.

To interact with the cloud assembly that your AWS CDK app creates, you typically use the AWS CDK CLI\. However, any tool that can read the cloud assembly format can be used to deploy your app\.

### Running your app<a name="app-run"></a>

The CDK CLI needs to know how to execute your AWS CDK app\. If you created the project from a template using the `cdk init` command, your app's `cdk.json` file includes an `app` key\. This key specifies the necessary command for the language that the app is written in\. If your language requires compilation, the command line performs this step before running the app, so you can't forget to do it\.

------
#### [ TypeScript ]

```
{
  "app": "npx ts-node --prefer-ts-exts bin/my-app.ts"
}
```

------
#### [ JavaScript ]

```
{
  "app": "node bin/my-app.js"
}
```

------
#### [ Python ]

```
{
    "app": "python app.py"
}
```

------
#### [ Java ]

```
{
  "app": "mvn -e -q compile exec:java"
}
```

------
#### [ C\# ]

```
{
  "app": "dotnet run -p src/MyApp/MyApp.csproj"
}
```

------
#### [ Go ]

```
{
  "app": "go mod download && go run my-app.go"
}
```

------

If you didn't create your project using the CDK CLI, or if you want to override the command line given in `cdk.json`, you can use the \-\-app option when issuing the `cdk` command\.

```
$ cdk --app 'executable' cdk-command ...
```

The *executable* part of the command indicates the command that should be run to execute your CDK application\. Use quotation marks as shown, since such commands contain spaces\. The *cdk\-command* is a subcommand like synth or deploy that tells the CDK CLI what you want to do with your app\. Follow this with any additional options needed for that subcommand\.

The AWS CDK CLI can also interact directly with an already\-synthesized cloud assembly\. To do that, pass the directory in which the cloud assembly is stored in \-\-app\. The following example lists the stacks defined in the cloud assembly stored under `./my-cloud-assembly`\.

```
$ cdk --app ./my-cloud-assembly ls
```