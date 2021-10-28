# Migrating to AWS CDK v2<a name="work-with-cdk-v2"></a>

Version 2 of the AWS Cloud Development Kit \(AWS CDK\) is designed to make writing infrastructure as code in your preferred programming language even easier\.

As a developer preview, CDK v2 is not intended for production use and may be subject to breaking API changes before it is finalized\. We are eager to hear your feedback on how CDK v2 works for you, and your suggestions for making it better, via [GitHub Issues](https://github.com/aws/aws-cdk/issues/new/choose)\.

## Changes in AWS CDK v2<a name="work-with-cdk-v2-changes.title"></a>

The main changes in CDK v2 are as follows\.
+ AWS CDK v2 consolidates the AWS Construct Library into a single package, `aws-cdk-lib`\. Developers no longer need to install one or more individual packages for each AWS service or to keep all CDK library versions in sync\.
+ Experimental modules are not included in the main AWS Construct Library; they are instead distributed as individual packages named with an `alpha` suffix and a semantic version number that matches the first version of the AWS Construct Library mith which they are compatible, also with an `alpha` suffix\. Constructs move into the main AWS Construct Library after being designated stable, permitting the main Construct Library to adhere to strict semantic versioning going forward\. 

  Stability is specified at the service level\. For example, if we begin creating one or more L2 constructs for Amazon AppFlow, which at this writing has only L1 constructs, they would appear in a module named `@aws-cdk/aws-appflow-alpha` at first, then move to `aws-cdk-lib` when we feel the new constructs meet the fundamental needs of customers\.

  Once a module has been designated stable and incorporated into `aws-cdk-lib`, new APIs are added using the "BetaN" convention described next\. 

   L1 \(CfnXXXX\) constructs are always considered stable and so are included in `aws-cdk-lib`\.

  A new version of each experimental module is released with every release of the AWS CDK, but for the most part, they needn't be kept in sync\. You can upgrade `aws-cdk-lib` or the experimental module whenever you want\. The exception is that when two or more related experimental modules depend on each other, they must be the same version\.
+ For stable modules to which new functionality is being added, new APIs \(whether entirely new constructs or new methods or properties on an existing construct\) receive a `Beta1` suffix \(and then `Beta2`, `Beta3`, etc\. when breaking changes are needed\) while work is in progress\. A version of the API without the suffix is added when the API is designated stable\. All methods except the latest \(whether beta or final\) are deprecated\.

  For example, if we add a new method `grantPower()` to a construct, it initially appears as `grantPowerBeta1().` If breaking changes are needed \(for example, a new required parameter or property\), the next version of the method would be named `grantPowerBeta2()`, and so on\. When work is complete and the API is finalized, the method `grantPower()` \(with no suffix\) is added\.

  All the beta APIs remain in the Construct Library until the next major version \(3\.0\) release, and their signatures are guaranteed not to change\. They'll be deprecated, so you'll see warnings, but you can continue to use them without fear that a future AWS CDK release will break your application\.
+ The core `Construct` class has been extracted from the AWS CDK into a separate library, along with related types, to support efforts to apply this programming model to other domains\. If you are writing your own constructs or using related APIs, you must declare the `construct` module as a dependency and make minor changes to your imports\. If you are using advanced features, such as hooking into the CDK app lifecycle, more changes may be needed\. [See the RFC](https://github.com/aws/aws-cdk-rfcs/blob/master/text/0192-remove-constructs-compat.md#release-notes) for full details\.
+ Deprecated properties, methods, and types in AWS CDK v1\.x and its Construct Library have been removed from the CDK v2 API\. In most supported languages, these APIs produce warnings under v1\.x, so you may have already migrated to the replacement APIs\. A complete [list of deprecated APIs](https://github.com/aws/aws-cdk/blob/master/DEPRECATED_APIs.md) in CDK v1\.x is available on GitHub\.
+ Behavior that was gated by feature flags in AWS CDK v1\.x is enabled by default in CDK v2, and the old feature flags are no longer needed or, in most cases, supported\. A handful are still supported so that you can revert to v1 behavior in very specific circumstances; see [Updating feature flags](#v2-migrating-cdk-json)\.
+ CDK v2 requires that the environments you deploy into be boostrapped using the modern bootstrap stack; the legacy stack is no longer supported\. CDK v2 furthermore requires a new version of the modern stack\. Simply re\-bootstrap the affected environments to upgrade them\. It is not necessary to set any feature flags or environment variables to specify the modern bootstrap stack\.

**Important**  
The modern bootstrap template effectively grants the permissions implied by the `--cloudformation-execution-policies` to any AWS account in the `--trust` list, which by default will extend permissions to read and write to any resource in the bootstrapped account\. Make sure to [configure the bootstrapping stack](bootstrapping.md#bootstrapping-customizing) with policies and trusted accounts you are comfortable with\.

Aside from this topic, the AWS CDK Developer Guide describes CDK v1\.x\. Most of the information in the Guide still applies in CDK v2, or can be adapted with only minor changes\. A v2 Developer Guide will be available at General Availability \(GA\) of CDK v2\. A version of the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) is available for CDK v2\.

## Prerequisites<a name="v2-prerequisites"></a>

Most requirements for AWS CDK v2 are the same as for AWS CDK v1\.x\. See [Prerequisites](getting_started.md#getting_started_prerequisites)\. Additional requirements are listed here\.
+ For TypeScript developers, TypeScript 3\.8 or later is required\.
+ A new version of the CDK Toolkit is required for use with CDK v2\. To install it, issue `npm install -g aws-cdk@next`\.

## Migrating to AWS CDK v2<a name="v2-migrating"></a>

To migrate your app to AWS CDK v2, first update the feature flags in `cdk.json`\. Then update your app's dependencies and imports as necessary for the programming language it is written in\.

### Updating feature flags<a name="v2-migrating-cdk-json"></a>

Remove all feature flags from `cdk.json`\. You can add one or more of the flags listed below, set to `false`, if your app relies on these specific AWS CDK v1\.x behaviors\. Use the `cdk diff` command to inspect the changes to your synthesized template to see if any of these flags are needed\. 

`@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId`  
If your application uses multiple Amazon API Gateway API keys and associates them to usage plans

`@aws-cdk/aws-rds:lowercaseDbIdentifier`  
If your application uses Amazon RDS database instance or database clusters, and explicitly specifies the identifier for these

`@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021`  
 If your application uses the TLS\_V1\_2\_2019 security policy with Amazon CloudFront distributions\. CDK v2 uses security policy TLSv1\.2\_2021 by default\. 

`@aws-cdk/core:stackRelativeExports`  
If your application uses multiple stacks and you refer to resources from one stack in another, this determines whether absolute or relative path is used to construct AWS CloudFormation exports

The syntax for reverting these flags in `cdk.json` is shown here\.

```
{
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": false,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": false,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": false,
    "@aws-cdk/core:stackRelativeExports": false,
  }
}
```

### Updating dependencies and imports<a name="v2-migrating-dependencies"></a>

Update your app's dependencies in the appropriate configuration file, then install the new dependencies\. Finally, update the imports in your code\.

------
#### [ TypeScript ]

For AWS CDK applications, update `package.json` as follows\. Remove dependencies on v1\-style individual stable modules\. Note that `@aws-cdk/assert` must be updated to the same version as `aws-cdk-lib`\.

Experimental constructs are provided in separate, independently\-versioned packages with names that end in `alpha` and an alpha version number that corresponds to the first release of `aws-cdk-lib` wit which they are compatible\. Here we have pinned `aws-codestar` to v2\.0\.0\-alpha\.1\.

```
{
  "dependencies": {
    "aws-cdk-lib": "^2.0.0-rc.1",
    "@aws-cdk/aws-codestar-alpha": "2.0.0-alpha.1",
    "@aws-cdk/assert": "^2.0.0-rc.1",
    "constructs": "^10.0.0"
  }
}
```

For construct libraries, establish the lowest version of `aws-cdk-lib` you require for your application \(2\.0\.0\-rc\.1 here\) and update `package.json` as follows\.

Note that `aws-cdk-lib` appears both as a peer dependency and a dev dependency\.

```
{
  "peerDependencies": {
    "aws-cdk-lib": "^2.0.0-rc.1",
    "constructs": "^10.0.0"
  },
  "devDependencies": {
    "aws-cdk-lib": "2.0.0-rc.1",
    "@aws-cdk/assert": "^2.0.0-rc.1",
    "constructs": "^10.0.0",
    "typescript": "~3.9.0"
  }  
}
```

Install the new dependencies by running `npm install` or `yarn install`\.

Change your app's imports to import `Construct` from the new `constructs` module, core types such as `App` and `Stack` from the top level of `aws-cdk-lib`, and AWS Construct Library modules from namespaces under `aws-cdk-lib`\.

```
import { Construct } from 'constructs';
import { App, Stack } from 'aws-cdk-lib';                 // core constructs
import { aws_s3 as s3 } from 'aws-cdk-lib';               // stable module
import * as codestar from '@aws-cdk/aws-codestar-alpha';  // experimental module
```

------
#### [ JavaScript ]

Update `package.json` as follows\. Remove dependencies on v1\-style individual stable modules\. Note that `@aws-cdk/assert` must be updated to the same version as `aws-cdk-lib`\.

Experimental constructs are provided in separate, independently\-versioned packages with names that end in `alpha` and an alpha version number that corresponds to the first release of `aws-cdk-lib` wit which they are compatible\. Here we have pinned `aws-codestar` to v2\.0\.0\-alpha\.1\.

```
{
  "dependencies": {
    "aws-cdk-lib": "^2.0.0-rc.1",
    "@aws-cdk/aws-codestar-alpha": "2.0.0-alpha.1",
    "@aws-cdk/assert": "^2.0.0-rc.1",
    "constructs": "^10.0.0"
  }
}
```

Install the new dependencies by running `npm install` or `yarn install`\.

Change your app's imports to import `Construct` from the new `constructs` module, core types such as `App` and `Stack` from the top level of `aws-cdk-lib`, and AWS Construct Library modules from namespaces under `aws-cdk-lib`\.

```
const { Construct } = require('constructs');
const { App, Stack } = require('aws-cdk-lib');              // core constructs
const s3 = require('aws-cdk-lib').aws_s3;                   // stable module
const codestar = require('@aws-cdk/aws-codestar-alpha');    // experimental module
```

------
#### [ Python ]

Update the `install_requires` definition in `setup.py` as follows\. Remove dependencies on v1\-style individual stable modules\. Note that `@aws-cdk/assert` must be updated to the same version as `aws-cdk-lib`\.

Experimental constructs are provided in separate, independently\-versioned packages with names that end in `alpha` and an alpha version number that corresponds to the first release of `aws-cdk-lib` wit which they are compatible\. Here we have pinned `aws-codestar` to v2\.0\.0alpha1\.

```
install_requires=[
     "aws-cdk-lib>=2.0.0rc1",
     "constructs>=10.0.0",
     "aws-cdk.aws-codestar-alpha>=2.0.0alpha1",
     # ...
],
```

Install the new dependencies with `pip install -r requirements.txt`\.

**Tip**  
Uninstall any other versions of AWS CDK modules already installed in your app's virtual environment using `pip uninstall`\.

Change your app's imports to import `Construct` from the new `constructs` module, core types such as `App` and `Stack` from the top level of `aws_cdk`, and AWS Construct Library modules from namespaces under `aws_cdk`\.

```
from constructs import Construct
from aws_cdk import App, Stack                    # core constructs
from aws_cdk import aws_s3 as s3                  # stable module
import aws_cdk.aws_codestar_alpha as codestar     # experimental module

# ...

class MyConstruct(Construct):
    # ...
    
class MyStack(Stack):
    # ...
    
s3.Bucket(...)
```

------
#### [ Java ]

In `pom.xml`, remove all `software.amazon.awscdk` dependencies for stable modules and replace them with dependencies on `software.constructs` and `software.amazon.awscdk`\.

Experimental constructs are provided in separate, independently\-versioned packages with names that end in `alpha` and an alpha version number that corresponds to the first release of `aws-cdk-lib` wit which they are compatible\. Here we have pinned `aws-codestar` to v2\.0\.0\-alpha\.1\.

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>aws-cdk-lib</artifactId>
    <version>2.0.0-rc.1</version>
</dependency><dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>code-star-alpha</artifactId>
    <version>2.0.0-alpha.1</version>
</dependency>
<dependency>
    <groupId>software.constructs</groupId>
    <artifactId>constructs</artifactId>
    <version>10.0.0</version>
</dependency>
```

Install the new dependencies by running `mvn package`\.

Change your code to import `Construct` from the new `software.constructs` library, core classes like `Stack` and `App` from `software.amazon.awscdk`, and service constructs from `software.amazon.awscdk.services`\.

```
import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.App;
import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.codestar.alpha.GitHubRepository;
```

------
#### [ C\# ]

The most straightforward way to upgrade the dependencies of a C\# CDK application is to edit the `.csproj` file manually\. Remove all stable `Amazon.CDK.*` package references and replace them with references to the `Amazon.CDK.Lib` and `Constructs` packages\.

Experimental constructs are provided in separate, independently\-versioned packages with names that end in `alpha` and an alpha version number that corresponds to the first release of `aws-cdk-lib` wit which they are compatible\. Here we have pinned `aws-codestar` to v2\.0\.0\-alpha\.1\.

```
<PackageReference Include="Amazon.CDK.Lib" Version="2.0.0-rc.1" />
<PackageReference include="Amazon.CDK.AWS.Codestar.Alpha" Version="2.0.0-alpha.1" />
<PackageReference Include="Constructs" Version="10.0.0" />
```

Install the new dependencies by running `dotnet restore`\.

Change the imports in your source files as follows\.

```
using Constructs;         // for Construct class
using Amazon.CDK;                   // for core classes like App and Stack
using Amazon.CDK.AWS.S3;            // for stable constructs like Bucket
using Amazon.CDK.Codestar.Alpha;    // for experimental constructs
```

------

## Troubleshooting<a name="work-with-cdk-v2-trouble.title"></a>

**Unexpected infrastructure changes**  
Before deploying your app, use `cdk diff` to check for unexpected changes to its resources\. Such changes are typically not caused by upgrading to AWS CDK v2, but are the result of deprecated behavior that was previously hidden by feature flags\. This is a symptom of upgrading from a version of CDK older than about 1\.85\.x; you'd see the same changes upgrading to the latest v1\.x release\. You can usually resolve this by upgrading your app to the latest v1\.x release, revising your code as necessary, deploying, and then upgrading to v2\.

If your upgraded app ends up undeployable after the two\-stage upgrade, please [report the issue](https://github.com/aws/aws-cdk/issues/new/choose)\.

**Typescript `'from' expected` or `';' expected` error in imports**  
Upgrade to TypeScript 3\.8 or later\.