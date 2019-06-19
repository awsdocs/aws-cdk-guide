--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Writing AWS CDK Constructs<a name="writing_constructs"></a>

This topic provides some tips for writing idiomatic new constructs for the AWS CDK\. These tips apply equally to constructs you write to include in the AWS Construct Library, purpose\-built constructs to achieve a well\-defined goal, or constructs that serve as building blocks for assembling your cloud applications\.

## General Design Principles<a name="writing_constructs_general"></a>
+ Favor composition over inheritance\. Most of the constructs should directly extend the `Construct` class instead of some other construct\. Use inheritance mainly to allow polymorphism\. Typically, you define a construct within your scope and expose any of its APIs and properties in the enclosing construct\.
+ Provide defaults for everything that a reasonable guess can be made for\. Ideally, `props` should be optional and `new MyAwesomeConstruct(this, "Foo")` should be enough to set up a reasonable variant of the construct\. This doesn't mean that the user shouldn't have the opportunity to customize\. Instead, it means that the specific parameter should be optional and set to a reasonable value if it's not supplied\. This might involve creating other resources as part of initializing this construct\. For example, all resources that require a role allow passing in a `Role` object \(specifically, a `RoleRef` object\)\. However, if the user doesn't supply a role, the resource should create an appropriate default `Role` object\.
+ Use contextual defaulting between properties\. The value of one property might affect sensible defaults for other properties\. For example:, supposethe `enableDnsSupport` property requires that the `dnsSupport` property is set\. The construct should only throw an error if `dnsSupport` is not set, and the user calls `enableDnsSupport`\. A user expects things to just work\.
+ Make the user think about intent, not implementation details\. For example, if establishing an association between two resources \(such as a `Topic` and a `Queue`\) requires multiple steps \(in this case, creating a `Subscription` but also setting appropriate IAM permissions\), make both things happen in a single call \(to `subscribeQueue()`\)\.
+ Don't rename concepts or terminology\. For example don't rename the Amazon Simple Queue Service Amazon SQS `dataKeyReusePeriod` to `keyRotation` because it will be hard for users to diagnose problems\. They won't be able to search for *sqs dataKeyReuse* and find topics on it\. You can introduce a concept if a counterpart doesn't exist in AWS CloudFormation, especially if it directly maps onto the mental model that users already have about a service\.
+ Optimize for the common case\. For example, although `AutoScalingGroup` accepts a `VPC`and deploys in the private subnet by default because that's the common case, it also gives the user the option to `placementOptions` for special cases\.
+ If a class can have multiple modes or behaviors, prefer values over polymorphism\. Try switching behavior on property values first\. Switch to multiple classes with a shared base class or interface only if there's value in having multiple classes\. This reinforces type safety, as perhaps one mode has different features or required parameters from another mode\.

## Implementation Details<a name="writing_constructs_implementation_details"></a>
+ Every construct consists of an exported class \(`MyConstruct`\) and an exported interface \(`MyConstructProps`\) that defines the parameters for instances of those classes\. The `props` argument is the third to the construct' constructor, after the mandatory `scope` and `id` arguments\. If all of the properties in the `props` object are optional, the parameter should be optional\.
+ Most of the logic happens in the constructor\. The constructor builds the state of the construct from the child objects and any default objects the construct creates\.
+ Validate as early as possible\. If a parameter doesn't make sense, throw an `Error` in the constructor\. Override the `validate()` method to validate mutations that can occur after construction time\. Validation has the following hierarchy:
  + Best – Incorrect code won't compile, because of type safety guarantees\.
  + Good – At runtime, check everything the type checker can't enforce and fail early \(error in the constructor\)\.
  + Okay – Validate everything that can't be checked at construction time at synth time \(error in `validate()`\)\.
  + Not great – Fail with an error in AWS CloudFormation \(bad because the AWS CloudFormation deploy operation can take a long time, and the error can take several minutes to surface\)\.
  + Very bad – Fail with a timeout during AWS CloudFormation deployment\. \(It can take up to an hour for resource stabilization to timeout\!\)
  + Worst – Don't fail the deployment at all, but fail at runtime\.
+ Avoid unneeded hierarchy in `props`\. Try to keep the `props` interface flat to help discoverability\.
+ Constructs are classes that have a set of invariants they maintain over their lifetime, such as which members are initialized, and relationships between properties as members are mutated\.
+ Constructs mostly have write\-only scalar properties that are passed in the constructor, but mutating functions for collections\. For example, there will be `construct.addElement()` or `construct.onEvent()` functions, but not `construct.setProperty()`\. It's okay to deviate from this convention when it makes sense for your own constructs\.
+ Don't expose `Tokens` to your consumers\. `Tokens` are an implementation mechanism for one of two purposes: representing AWS CloudFormation intrinsic values, or representing lazily evaluated values\. You can use them for implementation purposes, but use more specific types as part of your public API\.

  `Tokens` are typically used in the implementation of an AWS Construct Library to pass lazy values to other constructs\. For example, if you have an array that can be mutated during the lifetime of your class, you pass it to an AWS CloudFormation Resourceconstruct, such as: `new cdk.Token(() => this.myList)`\.
+ Be aware that you might not be able to usefully inspect all strings\. Any string passed into your construct may contain special markers that represent values that will only be known at deploy time \(for example, the ARN of a resource that will be created during deployment\)\. Those are *stringified Tokens* and they look like `"${TOKEN[Token.123]}"`\. You can't validate those against a regular expression, for example, as their real values are not known yet\. To determine whether your string contains a special marker, use `cdk.unresolved(string)`\.
+ Indicate units of measurement in property names that don't use a strong type\. For example, use **milli**, **sec**, **min**, **hr**, **Bytes**, **KiB**, **MiB**, and **GiB**\.
+ Be sure to define an `IMyResource` interface for your resources that defines the API area that other constructs will use\. Typical capabilities on this interface are querying for a resource ARN and adding resource permissions\.
+ Accept objects instead of ARNs or names\. When accepting other resources as parameters, declare your property as `resource: IMyResource` instead of `resourceArn: string`\. This makes snapping objects together feel natural to consumers, and also allows you to query or modify the incoming resource\. For example, the latter is particularly useful if something about IAM permissions needs to be set\.
+ If your construct wraps a single \(or most prominent\) other construct, give it an ID of either **"Resource"** or **"Default"**\. The main resource that an AWS construct represents should use the ID **"Resource"** For higher\-level wrapping resources, you will generally use **"Default"**\. Resources named **"Default"** inherit their scope's logical ID, while resources named **"Resource"** have a distinct logical ID, but the human\-readable part of it won't show the **"Resource"** part\.

## Implementation Language<a name="writing_constructs_implementation_language"></a>

To reuse construct libraries across programming languages, author them in TypeScript, unless you plan to limit your constructs to a single programming language\.

## Code Organization<a name="writing_constructs_code_organization"></a>

Your package should look like the following\.

```
your-package
├── package.json
├── README.md
├── lib
│   ├── index.ts
│   ├── some-resource.ts
│   └── some-other-resource.ts
└── test
      ├── integ.everything.lit.ts
   ├── test.some-resource.ts
      └── test.some-other-resource.ts
```
+  If your package represents the canonical AWS Construct Library for this service, name your package `@aws-cdk/aws-xxx`\. We recommend starting the name with `cdk-`\.
+ Put code under `lib/`, and tests under `test/`\.
+ The entry point should be `lib/index.ts` and should contain only `export`s for other files\.
+ You don't need to put every class in a separate file\. Try to think of a reader\-friendly organization of your source files\.
+ To make package\-private utility functions, put them in a file that isn't exported from `index.ts`\.
+ Free\-floating functions \(functions that are not part of a class definition\) cannot be accessed through JSII \(that is, from languages other than TypeScript and JavaScript\)\. Don't use them for public features of your construct library\.
+ Document all public APIs with doc comments \(`JSdoc` syntax\)\. Document defaults using the `@default` marker in doc comments\.

## Testing<a name="writing_constructs_testing"></a>
+ Add unit tests for every construct \(`test.xxx.ts`\), relating the construct's properties to the AWS CloudFormation template that is generated\. Use the `@aws-cdk/assert` library to make it easier to write assertions on the AWS CloudFormation output\.
+ Try to test one concern per unit test\. Even if you could test more than one feature of the construct per test, it's better to write multiple tests, one for each feature\. A test should have one reason to break\.
+ Add integration tests \(`integ.xxx.ts`\) that are AWS CDK apps that exercise the features of the construct, then load your shell with credentials and run npm run integ to exercise them\. You will also have to run this if the AWS CloudFormation output of the construct changes\.
+ If there are packages that you depend on only for testing, add them to `devDependencies` \(instead of regular `dependencies`\)\. You're still not allowed to create dependency cycles this way \(from the root, run scripts/find\-cycles\.sh to determine whether you have created any cycles\)\.
+ If possible, try to make your integration test literate \(`integ.xxx.lit.ts`\) and link to it from the `README`\.

## README<a name="writing_constructs_readme"></a>
+ The header should include maturity level\.
+ The header should start at `H2`, not `H1`\.
+ Near the top of the README, include some example code for the typical use case\.
+ If there are multiple common use cases, provide an example for each one and describe what is happening at a high level \(for example, which resources are created\)\.
+ Reference docs are not needed as those are generated from the doc comments in the constructs\.
+ Use literate \(`.lit.ts`\) integration tests in the `README` file\.

## Construct IDs<a name="writing_constructs_construct_ids"></a>

All child construct IDs are part of your public contract\. Those IDs are used to generate AWS CloudFormation logical names for resources\. If they change, AWS CloudFormation will replace the resource\. Technically, this means that if you change any ID of a child construct, you will have to major\-version\-bump your library\.