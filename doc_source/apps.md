# Apps<a name="apps"></a>

As described in [Constructs](constructs.md), to provision infrastructure resources, all constructs that represent AWS resources must be defined, directly or indirectly, within the scope of a [Stack](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/stack.html) construct\.

The following example declares a stack class named `MyFirstStack` that includes a single Amazon S3 bucket\. However, this only declares a stack\. You still need to define \(also known as to instantiate\) it in some scope to deploy it\.

```
class MyFirstStack extends Stack {
  constructor(scope: Construct, id: string, props: StackProps) {
    super(scope, id, props);

    new s3.Bucket(this, 'MyFirstBucket');
  }
}
```

## The App Construct<a name="apps_construct"></a>

To define the previous stack within some scope, use the [App](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk/app.html) construct\. The following example defines a `MyFirstStack` and produces the AWS CloudFormation template that the stack defined\.

```
const app = new App();
new MyFirstStack(app, 'hello-cdk');
app.synth();
```

The `App` construct doesn't require any initialization arguments, because it's the only construct that can be used as a root for the construct tree\. You can now use the `App` instance as a scope for defining a single instance of your stack\.

You can also extend the App construct as follows\.

```
class MyApp extends App {
  constructor() {
    new MyFirstStack(this, 'hello-cdk');
  }
}

new MyApp().synth();
```

Both options are equivalent\.

## App Lifecycle<a name="lifecycle"></a>

The following diagram shows the phases that the AWS CDK goes through when you call the cdk deploy\. This command deploys the resources that your app defines\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/Lifecycle.png)

An AWS CDK app goes through the following phases in its lifecycle\.

1\. Construction \(or Initialization\)  
 Your code instantiates all of the defined constructs and then links them together\. In this stage, all of the constructs \(app, stacks, and their child constructs\) are instantiated and the constructor chain is executed\. Most of your app code is executed in this stage\.

2\. Preparation  
All constructs that have implemented the `prepare` method participate in a final round of modifications, to set up their final state\. The preparation phase happens automatically\. As a user, you don't see any feedback from this phase\. It's rare to need to use the “prepare” hook, and generally not recommended\. You should be very careful when mutating the construct tree during this phase, because the order of operations could impact behavior\.

3\. Validation  
All constructs that have implemented the `validate` method can validate themselves to ensure that they're in a state that will correctly deploy\. You will get notified of any validation failures that happen during this phase\. Generally, we recommend that you perform validation as soon as possible \(usually as soon as you get some input\) and throw exceptions as early as possible\. Performing validation early improves diagnosability as stack traces will be more accurate, and ensures that your code can continue to execute safely\.

4\. Synthesis  
This is the final stage of the execution of your AWS CDK app\. It's triggered by a call to `app.synth()`, and it traverses the construct tree and invokes the `synthesize` method on all constructs\. Constructs that implement `synthesize` can participate in synthesis and emit deployment artifacts to the resulting cloud assembly\. These constructs include AWS CloudFormation templates, AWS Lambda application bundles, file and Docker image assets, and other deployment artifacts\. [Cloud Assemblies](#apps_cloud_assembly) describes the output of this phase\. In most cases, you won’t need to implement the `synthesize` method

5\. Deployment  
In this phase, the AWS CDK CLI takes the deployment artifacts cloud assembly produced by the synthesis phase and deploys it to an AWS environment\. It uploads assets to Amazon S3 and Amazon ECR, or wherever they need to go, and then starts an AWS CloudFormation deployment to deploy the application and create the resources\.

By the time the AWS CloudFormation deployment phase \(step 5\) starts, your AWS CDK app has already finished and exited\. This has the following implications:
+ The AWS CDK app can't respond to events that happen during deployment, such as a resource being created or the whole deployment finishing\. To run code during the deployment phase, you have to inject it into the AWS CloudFormation template as a [custom resource](cfn_layer.md#cfn_layer_custom)\. For more information about adding a custom resource to your app, see the [AWS CloudFormation module](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-cloudformation-readme.html), or the [custom\-resource](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/custom-resource/) example\.
+ The AWS CDK app might have to work with values that can't be known at the time it runs\. For example, if the AWS CDK app defines an Amazon S3 bucket with an automatically generated name, and you retrieve the `bucket.bucketName` attribute, that value is not the name of the deployed bucket\. Instead, you get a `Token` value\. To determine whether a particular value is available, call `cdk.isToken(value)`\. See [Tokens](tokens.md) for details\.

## Cloud Assemblies<a name="apps_cloud_assembly"></a>

The call to `app.synth()` is what tells the AWS CDK to synthesize a cloud assembly from an app\. Typically you don't interact directly with cloud assemblies\. They are files that include everything needed to deploy your app to a cloud environment\. For example, it includes an AWS CloudFormation template for each stack in your app, and a copy of any file assets or Docker images that you reference in your app\.

To interact with the cloud assembly that your AWS CDK app creates, you typically use the AWS CDK CLI\. But any tool that can read the cloud assembly format can be used to deploy your app\.

To work with the CDK CLI, you need to let it know how to execute an AWS CDK app\.

```
cdk --app executable cdk-command
```

The \-\-appoption instructs the CLI to run your AWS CDK app, and its contents depend on the programming language you use\. Eventually it should be a program that the operating system can run\. You can also create the `cdk.json`file and add information to it so that you need to call only `cdk cdk-command`\. For example, for JavaScript apps, the `cdk.json` file might look like the following, where `node bin/my-app.js` executes a Node\.js program\.

```
{
  "app": "node bin/my-app.js"
}
```

**Note**  
Use the `cdk init` command to create a language\-specific project, with a `cdk.json` file containing the correct configuration for the programming language you specify\.

The *cdk\-command* part of the AWS CDK CLI command represents what you want the AWS CDK to do with the app\.

The CLI can also interact directly with an already synthesized cloud assembly\. To do that, just pass the directory in which the cloud assembly is stored in `--app`\. The following example lists the stacks defined in the cloud assembly stored under `./my-cloud-assembly`\.

```
cdk --app ./my-cloud-assembly ls
```
