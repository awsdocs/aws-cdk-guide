# Migrating to AWS CDK v2<a name="work-with-cdk-v2"></a>

Version 2 of the AWS Cloud Development Kit \(AWS CDK\) is designed to make writing infrastructure as code in your preferred programming language even easier\.

As a developer preview, CDK v2 is not intended for production use and may be subject to breaking API changes before it is finalized\. We are eager to hear your feedback on how CDK v2 works for you, and your suggestions for making it better, via [GitHub Issues](https://github.com/aws/aws-cdk/issues/new/choose)\.

## Changes in AWS CDK v2<a name="work-with-cdk-v2-changes.title"></a>

The main changes in CDK v2 are:
+ AWS CDK v2 consolidates the AWS Construct Library into a single package; developers no longer need to install one or more individual packages for each AWS service\. This change also eliminates the requirement to keep all CDK library versions in sync\.
+ Experimental L2/L3 constructs are no longer included in the AWS Construct Library; they will instead be distributed separately with an appropriate semantic version number\. Constructs will move into the main AWS Construct Library only after being designated stable, permitting the Construct Library to adhere to strict semantic versioning going forward\. We are still finalizing the details of how experimental constructs will be distributed for CDK v2, so initially, CDK v2 does not provide experimental constructs\.
+ The core `Construct` class has been extracted from the AWS CDK into a separate library, along with related types, to support efforts to apply the CDK programming model to other domains\. If you are writing your own constructs or using related APIs, you must declare the `construct` module as a dependency and make minor changes to your imports\. If you are using advanced features, such as hooking into the CDK app lifecycle, more changes are needed\. [See the RFC](https://github.com/aws/aws-cdk-rfcs/blob/master/text/0192-remove-constructs-compat.md#release-notes) for full details\.
+ Deprecated properties, methods, and types in AWS CDK v1\.x and its Construct Library have been removed from the CDK v2 API\. In most supported languages, these APIs produce warnings under v1\.x, so you may have already migrated to the replacement APIs\. A complete [list of deprecated APIs](https://github.com/aws/aws-cdk/blob/master/DEPRECATED_APIs.md) in CDK v1\.x is available on GitHub\.
+ Behavior that was gated by feature flags in AWS CDK v1\.x is enabled by default in CDK v2, and the old feature flags are no longer needed or, in most cases, supported\.
+ CDK v2 requires that the environments you deploy into be boostrapped using the modern bootstrap stack; the legacy stack is no longer supported\. CDK v2 furthermore requires a new version of the modern stack\. Simply re\-bootstrap the affected environments to upgrade them\. It is not necessary to set any feature flags or environment variables to specify the modern bootstrap stack\.

Aside from this topic, the AWS CDK Developer Guide describes CDK v1\.x\. Most of the information in the Guide still applies in CDK v2, or can be adapted with only minor changes\. A v2 Developer Guide will be available at General Availability \(GA\) of CDK v2\. A version of the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) is available for CDK v2\.

## Prerequisites<a name="v2-prerequisites"></a>

Most requirements for AWS CDK v2 are the same as for AWS CDK v1\.x\. See [Prerequisites](getting_started.md#getting_started_prerequisites)\. Additional requirements are listed here\.
+ For TypeScript developers, TypeScript 3\.8 or later is required\.
+ A new version of the CDK Toolkit is required for use with CDK v2\. To install it, issue `npm install -g aws-cdk@next`\.

## Migrating to AWS CDK v2<a name="v2-migrating"></a>

To migrate your app to AWS CDK v2, first update the feature flags in `cdk.json`\. Then update your app's dependencies and imports as necessary for the programming language it is written in\.

### Updating `cdk.json`<a name="w289aac13c11b5"></a>

Remove all feature flags from `cdk.json`\. You can add one or more of the three flags listed below, set to `false`, if your app relies on these specific AWS CDK v1\.x behaviors\. Use the `cdk diff` command to inspect the changes to your synthesized template to see if any of these are needed\. 

`@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId`  
If your application uses multiple Amazon API Gateway API keys and associates them to usage plans

`@aws-cdk/aws-rds:lowercaseDbIdentifier`  
If your application uses Amazon RDS database instance or database clusters, and explicitly specifies the identifier for these

`@aws-cdk/core:stackRelativeExports`  
If your application uses multiple stacks and you refer to resources from one stack in another, this determines whether absolute or relative path is used to construct AWS CloudFormation exports

The syntax for reverting these flags in `cdk.json` is shown here\.

```
{
  "context": {
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": false,
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": false,
    "@aws-cdk/core:stackRelativeExports": false,
  }
}
```

### Updating dependencies and imports<a name="v2-migrating-dependencies"></a>

Update your app's dependencies in the appropriate configuration file, then install the new dependencies\. Finally, update the imports in your code\.

------
#### [ TypeScript ]

For AWS CDK applications, update `package.json` as follows\. Note that `@aws-cdk/assert` must be updated to the same version as `aws-cdk-lib`\.

```
{
  "dependencies": {
    "aws-cdk-lib": "^2.0.0-rc.1",
    "@aws-cdk/assert": "^2.0.0-rc.1",
    "constructs": "^10.0.0"
  }
}
```

For construct libraries, establish the lowest version of `aws-cdk-lib` you require for your application \(2\.0\.0\-rc\.1 here\) and update `package.json` as follows\. Note that `aws-cdk-lib` appears both as a peer dependency and a dev dependency\.

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
import { App, Stack } from 'aws-cdk-lib';
import { aws_s3 as s3 } from 'aws-cdk-lib';
```

------
#### [ JavaScript ]

Update `package.json` as follows\. Note that `@aws-cdk/assert` must be updated to the same version as `aws-cdk-lib`\.

```
{
  "dependencies": {
    "aws-cdk-lib": "^2.0.0-rc.1",
    "@aws-cdk/assert": "^2.0.0-rc.1",
    "constructs": "^10.0.0"
  }
}
```

Install the new dependencies by running `npm install` or `yarn install`\.

Change your app's imports to import `Construct` from the new `constructs` module, core types such as `App` and `Stack` from the top level of `aws-cdk-lib`, and AWS Construct Library modules from namespaces under `aws-cdk-lib`\.

```
const { Construct } = require('constructs');
const { App, Stack } = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib').aws_s3;
```

------
#### [ Python ]

Update the `install_requires` definition in `setup.py` to look like this\.

```
install_requires=[
     "aws-cdk-lib>=2.0.0rc1",
     "constructs>=10.0.0",
     # ...
]
```

Install the new dependencies with `pip install -r requirements.txt`\.

**Tip**  
Uninstall any other versions of AWS CDK modules already installed in your app's virtual environment using `pip uninstall`\.

Change your app's imports to import `Construct` from the new `constructs` module, core types such as `App` and `Stack` from the top level of `aws_cdk`, and AWS Construct Library modules from namespaces under `aws_cdk`\.

```
from constructs import Construct
from aws_cdk import App, Stack
from aws_cdk import aws_s3 as s3

# ...

class MyConstruct(Construct):
    # ...
    
class MyStack(Stack):
    # ...
    
s3.Bucket(...)
```

Alternatively, you can import `constructs` and `aws_cdk`, then refer to classes through their namespace\.

```
import constructs
import aws_cdk as cdk
from aws_cdk import aws_s3 as s3

# ...

class MyConstruct(constructs.Construct):
    # ...
    
class MyStack(cdk.Stack):
    # ...
    
s3.Bucket(...)
```

------
#### [ Java ]

In `pom.xml`, remove all `software.amazon.awscdk` dependencies and replace them with these dependencies on `software.constructs` and `software.amazon.awscdk`\.

```
<dependency>
    <groupId>software.amazon.awscdk</groupId>
    <artifactId>aws-cdk-lib</artifactId>
    <version>2.0.0-rc.1</version>
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
```

------
#### [ C\# ]

The most straightforward way to upgrade the dependencies of a C\# CDK application is to edit the `.csproj` file manually\. Remove all `Amazon.CDK.*` package references and replace them with references to the `Amazon.CDK.Lib` and `Constructs` packages\.

```
<PackageReference Include="Amazon.CDK.Lib" Version="2.0.0-rc.1" />
<PackageReference Include="Constructs" Version="10.0.0" />
```

Install the new dependencies by running `dotnet restore`\.

Change the imports in your source files as follows\.

```
using Constructs;         // for Construct class
using Amazon.CDK;         // for core classes like App and Stack
using Amazon.CDK.AWS.S3;  // for service constructs like Bucket
```

------

## Troubleshooting<a name="work-with-cdk-v2-trouble.title"></a>

**Unexpected infrastructure changes**  
Before deploying your app, use `cdk diff` to check for unexpected changes to its resources\. Such changes are typically not caused by upgrading to AWS CDK v2, but are the result of deprecated behavior that was previously hidden by feature flags\. This is a symptom of upgrading from a version of CDK older than about 1\.85\.x; you'd have the same problem upgrading to the latest v1\.x release\. You can usually resolve this by upgrading your app to the latest v1\.x release, revising your code as necessary, deploying, and then upgrading to v2\.

If your upgraded app ends up undeployable after the two\-stage upgrade, please [report the issue](https://github.com/aws/aws-cdk/issues/new/choose)\.

**Typescript `'from' expected` or `';' expected` error in imports**  
Upgrade to TypeScript 3\.8 or later\.