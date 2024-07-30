# AWS CDK apps<a name="apps"></a>

The AWS Cloud Development Kit \(AWS CDK\) application or *app* is a collection of one or more CDK [stacks](stacks.md)\. Stacks are a collection of one or more [constructs](constructs.md), which define AWS resources and properties\. Therefore, the overall grouping of your stacks and constructs are known as your CDK app\.

**Topics**
+ [Defining apps](#apps-define)
+ [The construct tree](#apps-tree)

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