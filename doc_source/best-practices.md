# Best practices for developing and deploying cloud infrastructure with the AWS CDK<a name="best-practices"></a>

The AWS CDK allows developers or administrators to define their cloud infrastructure using a supported programming language\. CDK applications should be organized into logical units, such as API, database, and monitoring resources, and optionally have a pipeline for automated deployments\. The logical units should be implemented as constructs including the infrastructure \(e\.g\. Amazon S3 buckets, Amazon RDS databases, Amazon VPC network\), runtime code \(e\.g\. AWS Lambda functions\), and configuration code\. Stacks define the deployment model of these logical units\. For a more detailed introduction to the concepts behind the CDK, see [Getting started with the AWS CDK](getting_started.md)\.

The AWS CDK reflects careful consideration of the needs of our customers and internal teams and of the failure patterns that often arise during the deployment and ongoing maintenance of complex cloud applications\. We discovered that failures are often related to "out\-of\-band" changes to an application, such as configuration changes, that were not fully tested\. Therefore, we developed the AWS CDK around a model in which your entire application, not just business logic but also infrastructure and configuration, is defined in code\. That way, proposed changes can be carefully reviewed, comprehensively tested in environments resembling production to varying degrees, and fully rolled back if something goes wrong\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/all-in-one.jpg)

At deployment time, the AWS CDK synthesizes a cloud assembly that contains not only AWS CloudFormation templates describing your infrastructure in all target environments, but file assets containing your runtime code and their supporting files\. With the CDK, every commit in your application's main version control branch can represent a complete, consistent, deployable version of your application\. Your application can then be deployed automatically whenever a change is made\.

The philosophy behind the AWS CDK leads to our recommended best practices, which we have divided into four broad categories\.

**Tip**  
In addition to the guidance in this document, you should also consider [best practices for AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html) as well as for the individual AWS services you use, where they are obviously applicable to CDK\-defined infrastructure\.
+ [Organization best practices](#best-practices-organization)
+ [Coding best practices](#best-practices-code)
+ [Construct best practices](#best-practices-constructs)
+ [Application best practices](#best-practices-apps)

## Organization best practices<a name="best-practices-organization"></a>

In the beginning stages of AWS CDK adoption, it's important to consider how to set up your organization for success\. It's a best practice to have a team of experts responsible for training and guiding the rest of the company as they adopt the CDK The size of this team may vary, from one or two people at a small company to a full\-fledged Cloud Center of Excellence \(CCoE\) at a larger company\. This team is responsible for setting standards and policies for how cloud infrastructure will be done at your company, as well as for training and mentoring developers\.

The CCoE may provide guidance on what programming languages should be used for cloud infrastructure\. The details will vary from one organization to the next, but a good policy helps make sure developers can easily understand and maintain all cloud infrastructure throughout the company\.

The CCoE also creates a "landing zone" that defines your organizational units within AWS\. A landing zone is a pre\-configured, secure, scalable, multi\-account AWS environment based on best practice blueprints\. You can tie together the services that make up your landing zone with [AWS Control Tower](https://aws.amazon.com/controltower/), a high\-level service configures and manages your entire multi\-account system from a single user interface\.

Development teams should be able use their own accounts for testing and have the ability to deploy new resources in these accounts as needed\. Individual developers can treat these resources as extensions of their own development workstation\. Using [CDK Pipelines](cdk_pipeline.md), the AWS CDK applications can then be deployed via a CI/CD account to testing, integration, and production environments \(each isolated in its own AWS region and/or account\) by merging the developers' code into your organization's canonical repository\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/best-practice-deploy-to-multiple-accounts.png)

## Coding best practices<a name="best-practices-code"></a>

 This section presents best practices for organizing your AWS CDK code\. The diagram below shows the relationship between a team and that team's code repositories, packages, applications, and construct libraries\. 

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/code-organization.jpg)

### Start simple and add complexity only when you need it<a name="best-practices-code-kiss"></a>

The guiding principle for most of our best practices is to keep things simple as possible—but no simpler\. Add complexity only when your requirements dictate a more complicated solution\. With the AWS CDK, you can always refactor your code as necessary to support new requirements, so it doesn't make sense to architect for all possible scenarios up front\.

### Align with the AWS Well\-Architected framework<a name="best-practices-code-well-architected"></a>

The [AWS Well\-Architected](http://aws.amazon.com/architecture/well-architected/) framework defines a *component* as the code, configuration, and AWS resources that together deliver against a requirement\. A component is often the unit of technical ownership, and is decoupled from other components\. The term *workload* is used to identify a set of components that together deliver business value\. A workload is usually the level of detail that business and technology leaders communicate about\.

An AWS CDK application maps to a component as defined by the AWS Well\-Architected Framework\. AWS CDK apps are a mechanism to codify and deliver Well\-Architected cloud application best practices\. You can also create and share components as reusable code libraries through artifact repositories, such as AWS CodeArtifact\.

### Every application starts with a single package in a single repository<a name="best-practices-code-package"></a>

A single package is the entry point of your AWS CDK app\. This is where you define how and where the different logical units of your application are deployed, as well as the CI/CD pipeline to deploy the application\. The app's constructs define the logical units of your solution\.

Use additional packages for constructs that you use in more than one application\. \(Shared constructs should also have their own lifecycle and testing strategy\.\) Dependencies between packages in the same repository are managed by your repo's build tooling\. 

Though it is possible, it is not recommended to put multiple applications in the same repository, especially when using automated deployment pipelines, because this increases the "blast radius" of changes during deployment\. With multiple applications in a repository, not only do changes to one application trigger deployment of the others \(even if they have not changed\), but a break in one application prevents the other applications from being deployed\.

### Move code into repositories based on code lifecycle or team ownership<a name="best-practices-code-repo"></a>

When packages begin to be used in multiple applications, move them to their own repository, so they can be referenced by the build systems of the applications that use them, but updated on cadences independent of the lifecycles of those applications\. It may make sense to put all shared constructs in one repository at first\.

Also move packages to their own repo when different teams are working on them, to help enforce access control\.

To consume packages across repository boundaries, you now need a private package repository—similar to NPM, PyPi, or Maven Central, but internal to your organization, and a release process that builds, tests, and publishes the package to the private package repository\. [CodeArtifact](%url-aca-user) can host packages for most popular programming languages\.

Dependencies on packages in the package repository are managed by your language's package manager, for example NPM for TypeScript or JavaScript applications\. Your package manager helps to make sure builds are repeatable by recording the specific versions of every package your application depends on and allows you to upgrade those dependencies in a controlled manner\.

Shared packages need a different testing strategy: although for a single application it might be good enough to deploy the application to a testing environment and confirm that it still works, shared packages need to be tested independently of the consuming application, as if they were being released to the public\. \(Your organization might in fact choose to actually release some shared packages to the public\.\)

Keep in mind that a construct can be arbitrarily simple or complex\. A `Bucket` is a construct, but `CameraShopWebsite` could be a construct, too\.

### Infrastructure and runtime code live in the same package<a name="best-practices-code-all"></a>

The AWS CDK not only generates AWS CloudFormation templates for deploying infrastructure, it also bundles runtime assets like Lambda functions and Docker images and deploys them alongside your infrastructure\. So it's not only possible to combine the code that defines your infrastructure and the code that implements your runtime logic into a single construct— it's a best practice\. These two kinds of code don't need to live in separate repositories or even in separate packages\.

A construct that is self\-contained, in other words that completely describes a piece of functionality including its infrastructure and logic, makes it easy to evolve the two kinds of code together, test them in isolation, share and reuse the code across projects, and version all the code in sync\.

## Construct best practices<a name="best-practices-constructs"></a>

 This section contains best practices for developing constructs\. Constructs are reusable, composable modules that encapsulate resources, and the building blocks of AWS CDK apps\.

### Model your app through constructs, not stacks<a name="best-practices-constructs-model"></a>

When breaking down your application into logical units, represent each unit as a descendant of [https://docs.aws.amazon.com/cdk/api/latest/docs/constructs.Construct.html](https://docs.aws.amazon.com/cdk/api/latest/docs/constructs.Construct.html) and not of [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html)\. Stacks are a unit of deployment, and so tend to be oriented to specific applications\. By using constructs instead of stacks, you give yourself and your users the flexibility to build stacks in the way that makes the most sense for each deployment scenario\. For example, you could compose multiple constructs into a `DevStack` with some configuration for development environments and then have a different composition for your `ProdStack`\.

### Configure with properties and methods, not environment variables<a name="best-practices-constructs-config"></a>

Environment variable lookups inside constructs and stacks are a common anti\-pattern\. Both constructs and stacks should accept a properties object to allow for full configurability completely in code\. To do otherwise is to introduce a dependency on the machine that the code will run on, which becomes another bit of configuration information you have to keep track of and manage\.

In general, environment variable lookups should be limited to the top level of an AWS CDK app, and should be used to pass in information needed for running in a development environment; see [Environments](environments.md)\.

### Unit test your infrastructure<a name="best-practices-constructs-test"></a>

If you avoid network lookups during synthesis and model all your production stages in code \(best practices we cover later\), you can run a full suite of unit tests at build time, consistently, in all environments\. If any single commit always results in the same generated template, you can trust the unit tests that you write to confirm that the generated templates look how you expect them to\. See [Testing constructs](testing.md)\.

### Don't change the logical ID of stateful resources<a name="best-practices-constructs-logicalid"></a>

Changing the logical ID of a resource results in the resource being replaced with a new one at the next deployment\. For stateful resources like databases and buckets, or persistent infrastructure like an Amazon VPC, this is almost never what you want\. Be careful about any refactor of your AWS CDK code that could cause the ID to change, and write unit tests that assert that the logical IDs of your stateful resources remain static\. The logical ID is derived from the `id` you specify when you instantiate the construct, and the construct's position in the construct tree; see [Logical IDs](identifiers.md#identifiers_logical_ids)\.

### Constructs aren't enough for compliance<a name="best-practices-constructs-compliance"></a>

Many enterprise customers are writing their own wrappers for L2 constructs \(the "curated" constructs that represent individual AWS resources with built\-in sane defaults and best practices\) to enforce security best practices such as static encryption and specific IAM policies\. For example, you might create a `MyCompanyBucket` that you then use in your applications in place of the usual Amazon S3 `Bucket` construct\. This pattern is useful for surfacing security guidance early in the software development lifecycle, but it cannot be relied on as the sole means of enforcement\.

Instead, use AWS features such as [service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) and [permission boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) to enforce your security guardrails at the organization level\. Use [Aspects](aspects.md) or tools like [CloudFormation Guard](https://github.com/aws-cloudformation/cloudformation-guard) to make assertions about the security properties of infrastructure elements before deployment\. Use AWS CDK for what it does best\.

Finally, keep in mind that writing your own "L2\+" constructs like these may prevent your developers from taking advantage of the growing ecosystems of AWS CDK packages, such as [AWS Solutions Constructs](https://docs.aws.amazon.com/solutions/latest/constructs/welcome.html), as these are typically built upon standard AWS CDK constructs and won't be able to use your custom versions\.

## Application best practices<a name="best-practices-apps"></a>

In this section we discuss how best to write your AWS CDK applications, combining constructs to define how your AWS resources are connected\.

### Make decisions at synthesis time<a name="best-practices-apps-synth"></a>

Although AWS CloudFormation lets you make decisions at deployment time \(using `Conditions`, `{ Fn::If }`, and `Parameters`\), and the AWS CDK gives you some access to these mechanisms, we recommend against using them\. The types of values you can use, and the types of operations you can perform on them, are quite limited compared to those available in a general\-purpose programming language\.

Instead, try to make all decisions, such as which construct to instantiate, in your AWS CDK application, using your programming language's `if` statements and other features\. For example, a common CDK idiom, iterating over a list and instantiating a construct with values from each item in the list, simply isn't possible using AWS CloudFormation expressions\.

Treat AWS CloudFormation as an implementation detail that the AWS CDK uses for robust cloud deployments, not as a language target\. You're not writing AWS CloudFormation templates in TypeScript or Python, you're writing CDK code that happens to use CloudFormation for deployment\.

### Use generated resource names, not physical names<a name="best-practices-apps-names"></a>

Names are a precious resource\. Every name can only be used once, so if you hard\-code a table name or bucket name into your infrastructure and application, you can't deploy that piece of infrastructure twice in the same account\. \(The name we're talking about here is the name specified by, for example, the `bucketName` property on an Amazon S3 bucket construct\.\)

What's worse, you can't make changes to the resource that require it to be replaced\. If a property can only be set at resource creation, for example the `KeySchema` of an Amazon DynamoDB table, that property is immutable, and changing it requires a new resource\. But the new resource must have the same name in order to be a true replacement, and it can't while the existing resource is still using that name\.

A better approach is to specify as few names as possible\. If you leave out resource names, the AWS CDK will generate them for you, and it'll do so in a way that won't cause these problems\. You then, for example, pass the generated table name \(which you can reference as `table.tableName` in your AWS CDK application\) as an environment variable into your AWS Lambda function, or you generate a configuration file on your Amazon EC2 instance on startup, or you write the actual table name to AWS Systems Manager Parameter Store and your application reads it from there\.

If the place you need it is another AWS CDK stack, that's even easier\. Given one stack that defines the resource and another than needs to use it:
+ If the two stacks are in the same AWS CDK app, just pass a reference between the two stacks\. For example, save a reference to the resource's construct as an attribute of the defining stack \(`this.stack.uploadBucket = myBucket`\), then pass that attribute to the constructor of the stack that needs the resource\.
+ When the two stacks are in different AWS CDK apps, use a static `from` method to import an externally\-defined resource based on its ARN, name, or other attributes \(for example, `Table.fromarn()` for a DynamoDB table\)\. Use the `CfnOutput` construct to print the ARN or other required value in the output of `cdk deploy`, or look in the AWS console\. Or the second app can parse the CloudFormation template generated by the first app and retrieve that value from the Outputs section\.

### Define removal policies and log retention<a name="best-practices-apps-removal-logs"></a>

The AWS CDK does its best to keep you from losing data by defaulting to policies that retain everything you create\. For example, the default removal policy on resources that contain data \(such as Amazon S3 buckets and database tables\) is to never delete the resource when it is removed from the stack, but rather orphan the resource from the stack\. Similarly, the CDK's default is to retain all logs forever\. In production environments, these defaults can quickly result in the storage of large amounts of data you don't actually need, and a corresponding AWS bill\.

Consider carefully what you want these policies to actually be for each production resource and specify them accordingly\. Use [Aspects](aspects.md) to validate the removal and logging policies in your stack\.

### Separate your application into multiple stacks as dictated by deployment requirements<a name="best-practices-apps-separate"></a>

There is no hard and fast rule to how many stacks your application needs\. You'll usually end up basing the decision on your deployment patterns\. Keep in mind the following guidelines:
+ It's typically easier to keep as many resources in the same stack as possible, so keep them together unless you know you want them separated\.
+ Consider keeping stateful resources \(like databases\) in a separate stack from stateless resources\. You can then turn on termination protection on the stateful stack, and can freely destroy or create multiple copies of the stateless stack without risk of data loss\.
+ Stateful resources are more sensitive to construct renaming—renaming leads to resource replacement—so it makes sense not to nest them inside constructs that are likely to be moved around or renamed \(unless the state can be rebuilt if lost, like a cache\)\. This is another good reason to put stateful resources in their own stack\.

### Commit `cdk.context.json` to avoid non\-deterministic behavior<a name="best-practices-apps-context"></a>

Determinism is key to successful AWS CDK deployments\. A AWS CDK app should have essentially the same result whenever it is deployed to a given environment\.

Since your AWS CDK app is written in a a general\-purpose programming language, it can execute arbitrary code, use arbitrary libraries, and make arbitrary network calls\. For example, you could use an AWS SDK to retrieve some information from your AWS account while synthesizing your app\. Recognize that doing so will result in additional credential setup requirements, increased latency, and a chance, however small, of failure every time you run `cdk synth`\.

You should never modify your AWS account or resources during synthesis; synthesizing an app should not have side effects\. Changes to your infrastructure should happen only in the deployment phase, after the AWS CloudFormation template has been generated\. This way, if there's a problem, AWS CloudFormation will automatically roll back the change\. To make changes that can't be easily made within the AWS CDK framework, use [custom resources](https://docs.aws.amazon.com/cdk/api/latest/docs/custom-resources-readme.html) to execute arbitrary code at deployment time\.

Even strictly read\-only calls are not necessarily safe\. Consider what happens if the value returned by a network call changes\. What part of your infrastructure will that impact? What will happen to already\-deployed resources? Here are just two of the situations in which a sudden change in values might cause a problem\.
+ If you provision an Amazon VPC to all available Availability Zones in a specified region, and the number of AZs is two on deployment day, your IP space gets split in half\. If AWS launches a new Availability Zone the next day, the next deployment after that tries to split your IP space into thirds, requiring all subnets to be recreated\. This probably won't be possible because your Amazon EC2 instances are still running, and you'll have to clean this up manually\.
+ If you query for the latest Amazon Linux machine image and deploy an Amazon EC2 instance, and the next day a new image is released, a subsequent deployment picks up the new AMI and replaces all your instances\. This may not be what you expected to happen\.

These situations can be particularly pernicious because the AWS\-side change may occur after months or years of successful deployments\. Suddenly your deployments are failing "for no reason" and you long ago forgot what you did and why\.

Fortunately, the AWS CDK includes a mechanism called *context providers* to record a snapshot of non\-deterministic values, allowing future synthesis operations produce exactly the same template\. The only changes in the new template are the changes *you* made in your code\. When you use a construct's `.fromLookup()` method, the result of the call is cached in `cdk.context.json`, which you should commit to version control along with the rest of your code to ensure future executions of your CDK app use the same value\. The CDK Toolkit includes commands to manage the context cache, so you can refresh specific entries when you need to\. For more information, see [Runtime context](context.md)\.

If you need some value \(from AWS or elsewhere\) for which there is no native CDK context provider, we recommend writing a separate script to retrieve the value and write it to a file, then read that file in your CDK app\. Run the script only when you want to refresh the stored value, not as part of your regular build process\.

### Let the AWS CDK manage roles and security groups<a name="best-practices-apps-roles"></a>

One of the great features of the AWS CDK construct library is the `grant()` convenience methods that allow quick and simple creation of AWS Identity and Access Management roles granting access to one resource by another using minimally\-scoped permissions\. For example, consider a line like the following:

```
myBucket.grantRead(myLambda)
```

This single line results in a policy being added to the Lambda function's role \(which is also created for you\)\. That role and its policies are more than a dozen lines of CloudFormation that you don't have to write, and the AWS CDK grants only the minimal permissions required for the function to read from the bucket\.

If you require developers to always use predefined roles that were created by a security team, AWS CDK coding becomes much more complicated, and your teams lose a lot of flexibility in how they design their applications\. A better alternative is to use [service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) and [permission boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) to ensure that developers stay within the guardrails\.

### Model all production stages in code<a name="best-practices-apps-stages"></a>

 In traditional AWS CloudFormation scenarios, your goal is to produce a single artifact that is parameterized so that it can be deployed to various target environments after applying configuration values specific to those environments\. In the CDK, you can, and should, build that configuration right into your source code\. Create a stack for your production environment, and a separate one for each of your other stages, and put the configuration values for each right there in the code\. Use services like [Secrets Manager](https://aws.amazon.com/secrets-manager/) and [Systems Manager](https://aws.amazon.com/systems-manager/) Parameter Store for sensitive values that you don't want to check in to source control, using the names or ARNs of those resources\.

 When you synthesize your application, the cloud assembly created in the `cdk.out` folder contains a separate template for each environment\. Your entire build is deterministic: there are no out\-of\-band changes to your application, and any given commit always yields the exact same AWS CloudFormation template and accompanying assets, which makes unit testing much more reliable\. 

### Measure everything<a name="best-practices-apps-measure"></a>

Achieving the goal of full continuous deployment, with no human intervention, requires a high level of automation, and that automation isn't possible without extensive amounts of monitoring\. Create metrics, alarms, and dashboards to measure all aspects of your deployed resources\. And don't just measure simple things like CPU usage and disk space: also record your business metrics, and use those measurements to automate deployment decisions like rollbacks\. Most of the L2 constructs in AWS CDK have convenience methods to help you create metrics, such as the `metricUserErrors()` method on the [dynamodb\.Table](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-dynamodb.Table.html) class\.