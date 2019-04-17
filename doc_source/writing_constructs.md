--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Writing AWS CDK Constructs<a name="writing_constructs"></a>

This topic provides some tips for writing idiomatic new constructs for the CDK\. These tips apply equally to constructs written for inclusion in the AWS Construct Library, purpose\-built constructs to achieve a well\-defined goal, or constructs that serve as building blocks for assembling your cloud applications\.

## General Design Principles<a name="writing_constructs_general"></a>
+ Favor composition over inheritance\. Most of the constructs should directly extend the `Construct` class instead of some other construct\. Use inheritance mainly to allow polymorphism\. Typically, you define a construct within your scope and expose any of its APIs and properties in the enclosing construct\.
+ Provide defaults for everything that a reasonable guess can be made for\. Ideally, `props` should be optional and `new MyAwesomeConstruct(this, "Foo")` should be enough to set up a reasonable variant of the construct\. This doesn't mean that the user should not have the opportunity to customize\! Instead, it means that the specific parameter should be optional and set to a reasonable value if it's not supplied\. This might involve creating other resources as part of initializing this construct\. For example, all resources that require a role allow passing in a `Role` object \(specifically, a `RoleRef` object\), but if the user doesn't supply one, an appropriate `Role` object is defined in place\.
+ Use contextual defaulting between properties\. The value of one property might affect sensible defaults for other properties\. For example: `enableDnsHostnames` and `enableDnsSupport`\. `dnsHostnames` require `dnsSupport`, so only throw an error if the user has explicitly disabled DNS Support, but tried to enable DNS ostnames\. A user expects things to just work\.
+ Make the user think about intent, not implementation detail\. For example, if establishing an association between two resources \(such as a `Topic` and a `Queue`\) requires multiple steps \(in this case, creating a `Subscription` but also setting appropriate IAM permissions\), make both things happen in a single call \(to `subscribeQueue()`\)\.
+ Don't rename concepts or terminology\. For example don't rename the Amazon SQS `dataKeyReusePeriod` to `keyRotation` because it will be hard for people to diagnose problems\. They won't be able to search for *sqs dataKeyReuse* and find topics on it\. It's permissible to introduce a concept if a counterpart doesn't exist in AWS CloudFormation, especially if it directly maps onto the mental model that users already have about a service\.
+ Optimize for the common case\. For example, `AutoScalingGroup` accepts a `VPC` and deploys in the private subnet by default because that's the common case, but has an option to `placementOptions` for special cases\.
+ If a class can have multiple modes or behaviors, prefer values over polymorphism\. Try switching behavior on property values first\. Switch to multiple classes with a shared base class or interface only if there's value to be had from having multiple classes \(type safety, maybe one mode has different featuresor required parameters\)\.

## Implementation Details<a name="writing_constructs_implementation_details"></a>
+ Every construct consists of an exported class \(`MyConstruct`\) and an exported interface \(`MyConstructProps`\) that defines the parameters for these classes\. The `props` argument is the third to the construct \(after the mandatory `scope` and `id` arguments\), and the entire parameter should be optional if all of the properties on the `props` object are optional\.
+ Most of the logic happens in the constructor\. The constructor builds up the state of the construct \(what children it has, which ones are always there and which are optional, and so on\)\.
+ Validate as early as possible\. Throw an `Error` in the constructor if the parameters don't make sense\. Override the `validate()` method to validate mutations that can occur after construction time\. Validation has the following hierarchy:
  + Best – Incorrect code won't compile, because of type safety guarantees\.
  + Good – Runtime check everything the type checker can't enforce and fail early \(error in the constructor\)\.
  + Okay – Validate everything that can't be checked at construction time at synth time \(error in `validate()`\)\.
  + Not great – Fail with an error in AWS CloudFormation \(bad because the AWS CloudFormation deploy operation can take a long time, and the error can take several minutes to surface\)\.
  + Very bad – Fail with a timeout during AWS CloudFormation deployment\. \(It can take up to an hour for resource stabilization to timeout\!\)
  + Worst – Don't fail the deployment at all, but fail at runtime\.
+ Avoid unneeded hierarchy in `props`\. Try to keep the `props` interface flat to help discoverability\.
+ Constructs are classes that have a set of invariants they maintain over their lifetime \(such as which members are initialized, and relationships between properties as members that are mutated\)\.
+ Constructs mostly have write\-only scalar properties that are passed in the constructor, but mutating functions for collections \(for example, there will be `construct.addElement()` or `construct.onEvent()` functions, but not `construct.setProperty()`\)\. It's perfectly fine to deviate from this convention when it makes sense for your own constructs\.
+ Don't expose `Tokens` to your consumers\. Tokens are an implementation mechanism for one of two purposes: representing AWS CloudFormation intrinsic values, or representing lazily evaluated values\. You can use them for implementation purposes, but use more specific types as part of your public API\.
+ `Tokens` are \(mostly\) used only in the implementation of an AWS Construct Library to pass lazy values to other constructs\. For example, if you have an array that can be mutated during the lifetime of your class, you pass it to an AWS CloudFormation Resourceconstruct like so: `new cdk.Token(() => this.myList)`\.
+ Be aware that you might not be able to usefully inspect all strings\. Any string passed into your construct may contain special markers that represent values that will only be known at deploy time \(for example, the ARN of a resource that will be created during deployment\)\. Those are *stringified Tokens* and they look like `"${TOKEN[Token.123]}"`\. You will not be able to validate those against a regular expression, for example, as their real values are not known yet\. To determine whether your string contains a special marker, use `cdk.unresolved(string)`\.
+ Indicate units of measurement in property names that don't use a strong type\. For example, use **milli**, **sec**, **min**, **hr**, **Bytes**, **KiB**, **MiB**, and **GiB**\.
+ Be sure to define an `IMyResource` interface for your resources that defines the API area that other constructs will use\. Typical capabilities on this interface are querying for a resource ARN and adding resource permissions\.
+ Accept objects instead of ARNs or names\. When accepting other resources as parameters, declare your property as `resource: IMyResource` instead of `resourceArn: string`\. This makes snapping objects together feel natural to consumers, and allows you to query or modify the incoming resource as well\. For example, the latter is particularly useful if something about IAM permissions needs to be set\.
+ Implement `export()` and `import()` functions for your resource\. These make it possible to interoperate with resources that are not defined in the same CDK app \(they might be manually created, created using raw AWS CloudFormation resources, or created in a completely unrelated CDK app\)\.
+ If your construct wraps a single \(or most prominent\) other construct, give it an ID of either **"Resource"** or **"Default"**\. The main resource that an AWS construct represents should use the ID **"Resource"** For higher\-level wrapping resources, you will generally use **"Default"** \(resources named **"Default"** will inherit their scope's logical ID, while resources named **"Resource"** will have a distinct logical ID, but the human\-readable part of it won't show the **"Resource"** part\)\.

## Implementation Language<a name="writing_constructs_implementation_language"></a>

For construct libraries to be reusable across programming languages, they need to be authored in a language that can compile to a `jsii` assembly\.

Author in TypeScript unless you plan to isolate your constructs to a single programming language\.

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
+  If it represents the canonical AWS Construct Library for this service, name your package is named `@aws-cdk/aws-xxx`\. We recommend starting with `cdk-`, but you are otherwise free to choose the name\.
+ Put code under `lib/`, and tests under `test/`\.
+ The entry point should be `lib/index.ts` and should only contain `export`s for other files\.
+ You don't need to put every class in a separate file\. Try to think of a reader\-friendly organization of your source files\.
+ To make package\-private utility functions, put them in a file that isn't exported from `index.ts`\.
+ Free\-floating functions \(functions that are not part of a class definition\) cannot be accessed through jsii \(that is, from languages other than TypeScript and JavaScript\)\. Don't use them for public features of your construct library\.
+ Document all public APIs with doc comments \(`JSdoc` syntax\)\. Document defaults using the **@default** marker in doc comments\.

## Testing<a name="writing_constructs_testing"></a>
+ Add unit tests for every construct \(`test.xxx.ts`\), relating the construct's properties to the AWS CloudFormation that is generated\. Use the `@aws-cdk/assert` library to make it easier to write assertions on the AWS CloudFormation output\.
+ Try to test one concern per unit test\. Even if you could test more than one feature of the construct per test, it's better to write multiple tests, one for each feature\. A test should have one reason to break\.
+ Add integration tests \(`integ.xxx.ts`\) that are CDK apps that exercise the features of the construct, then load your shell with credentials and run npm run integ to exercise them\. You will also have to run this if the AWS CloudFormation output of the construct changes\.
+ If there are packages that you depend on only for testing, add them to `devDependencies` \(instead of regular `dependencies`\)\. You're still not allowed to create dependency cycles this way \(from the root, run scripts/find\-cycles\.sh to determine whether you have created any cycles\)\.
+ If possible, try to make your integ test literate \(`integ.xxx.lit.ts`\) and link to it from the `README`\.

## README<a name="writing_constructs_readme"></a>
+ Header should include maturity level\.
+ Header should start at `H2`, not `H1`\.
+ Include some example code for the simple use case near the very top\.
+ If there are multiple common use cases, provide an example for each one and describe what happens under the hood at a high level \(for example, which resources are created\)\.
+ Reference docs are not needed\.
+ Use literate \(`.lit.ts`\) integration tests in the `README` file\.

## Construct IDs<a name="writing_constructs_construct_ids"></a>

All child construct IDs are part of your public contract\. Those IDs are used to generate AWS CloudFormation logical names for resources\. If they change, AWS CloudFormation will replace the resource\. Technically, this means that if you change any ID of a child construct you will have to major\-version\-bump your library\.