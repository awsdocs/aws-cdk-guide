# Assets<a name="assets"></a>

Assets are local files, directories, or Docker images that can be bundled into AWS CDK libraries and apps; for example, a directory that contains the handler code for an AWS Lambda function\. Assets can represent any artifact that the app needs to operate\.

You typically reference assets through APIs that are exposed by specific AWS constructs\. For example, when you define a [lambda\.Function](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html) construct, the [code](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html#code) property lets you pass an [asset](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Code.html#static-assetpath) \(directory\)\. `Function` uses assets to bundle the contents of the directory and use it for the function's code\. Similarly, [ecs\.ContainerImage\.fromAsset](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecs.ContainerImage.html#static-from-assetdirectory-props) uses a Docker image built from a local directory when defining an Amazon ECS task definition\.

## Assets in detail<a name="assets_details"></a>

When you refer to an asset in your app, the [cloud assembly](apps.md#apps_cloud_assembly) synthesized from your application includes metadata information with instructions for the AWS CDK CLI on where to find the asset on the local disk, and what type of bundling to perform based on the type of asset, such as a directory to compress \(zip\) or a Docker image to build\.

The AWS CDK generates a source hash for assets, which can be used at construction time to determine whether the contents of an asset have changed\.

By default, the AWS CDK creates a copy of the asset in the cloud assembly directory, which defaults to `cdk.out`, under the source hash\. This is so that the cloud assembly is self\-contained and moved over to a different host for deployment\. See [Cloud assemblies](apps.md#apps_cloud_assembly) for details\.

The AWS CDK also synthesizes AWS CloudFormation parameters that the AWS CDK CLI specifies during deployment\. The AWS CDK uses those parameters to refer to the deploy\-time values of the asset\.

When the AWS CDK deploys an app that references assets \(either directly by the app code or through a library\), the AWS CDK CLI first prepares and publishes them to Amazon S3 or Amazon ECR, and only then deploys the stack\. The AWS CDK specifies the locations of the published assets as AWS CloudFormation parameters to the relevant stacks, and uses that information to enable referencing these locations within an AWS CDK app\.

This section describes the low\-level APIs available in the framework\.

## Asset types<a name="assets_types"></a>

The AWS CDK supports the following types of assets:

Amazon S3 Assets  
These are local files and directories that the AWS CDK uploads to Amazon S3\.

Docker Image  
These are Docker images that the AWS CDK uploads to Amazon ECR\.

These asset types are explained in the following sections\.

### Amazon S3 assets<a name="assets_types_s3"></a>

You can define local files and directories as assets, and the AWS CDK packages and uploads them to Amazon S3 through the [aws\-s3\-assets](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-assets-readme.html) module\.

The following example defines a local directory asset and a file asset\.

------
#### [ TypeScript ]

```
import { Asset } from '@aws-cdk/aws-s3-assets';

// Archived and uploaded to Amazon S3 as a .zip file
const directoryAsset = new Asset(this, "SampleZippedDirAsset", {
  path: path.join(__dirname, "sample-asset-directory")
});

// Uploaded to Amazon S3 as-is
const fileAsset = new Asset(this, 'SampleSingleFileAsset', {
  path: path.join(__dirname, 'file-asset.txt')
});
```

------
#### [ JavaScript ]

```
const { Asset } = require('@aws-cdk/aws-s3-assets');

// Archived and uploaded to Amazon S3 as a .zip file
const directoryAsset = new Asset(this, "SampleZippedDirAsset", {
  path: path.join(__dirname, "sample-asset-directory")
});

// Uploaded to Amazon S3 as-is
const fileAsset = new Asset(this, 'SampleSingleFileAsset', {
  path: path.join(__dirname, 'file-asset.txt')
});
```

------
#### [ Python ]

```
import os.path
dirname = os.path.dirname(__file__)

from aws_cdk.aws_s3_assets import Asset

# Archived and uploaded to Amazon S3 as a .zip file
directory_asset = Asset(self, "SampleZippedDirAsset",
  path=os.path.join(dirname, "sample-asset-directory")
)

# Uploaded to Amazon S3 as-is
file_asset = Asset(self, 'SampleSingleFileAsset',
  path=os.path.join(dirname, 'file-asset.txt')
)
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.services.s3.assets.Asset;

// Directory where app was started
File startDir = new File(System.getProperty("user.dir"));

// Archived and uploaded to Amazon S3 as a .zip file
Asset directoryAsset = Asset.Builder.create(this, "SampleZippedDirAsset")
                .path(new File(startDir, "sample-asset-directory").toString()).build();

// Uploaded to Amazon S3 as-is
Asset fileAsset = Asset.Builder.create(this, "SampleSingleFileAsset")
                .path(new File(startDir, "file-asset.txt").toString()).build();
```

------
#### [ C\# ]

```
using System.IO;
using Amazon.CDK.AWS.S3.Assets;

// Archived and uploaded to Amazon S3 as a .zip file
var directoryAsset = new Asset(this, "SampleZippedDirAsset", new AssetProps
{
    Path = Path.Combine(Directory.GetCurrentDirectory(), "sample-asset-directory")
});

// Uploaded to Amazon S3 as-is
var fileAsset = new Asset(this, "SampleSingleFileAsset", new AssetProps
{
    Path = Path.Combine(Directory.GetCurrentDirectory(), "file-asset.txt")
});
```

------

In most cases, you don't need to directly use the APIs in the `aws-s3-assets` module\. Modules that support assets, such as `aws-lambda`, have convenience methods that enable you to use assets\. For Lambda functions, the [asset](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Code.html#static-assetpath) property enables you to specify a directory or a \.zip file in the local file system\.

#### Lambda function example<a name="assets_types_s3_lambda"></a>

A common use case is to create AWS Lambda functions with the handler code, which is the entry point for the function, as an Amazon S3 asset\.

The following example uses an Amazon S3 asset to define a Python handler in the local directory `handler` and creates a Lambda function with the local directory asset as the `code` property\. Below is the Python code for the handler\.

```
def lambda_handler(event, context):
  message = 'Hello World!'
  return {
    'message': message
  }
```

The code for the main AWS CDK app should look like the following\.

------
#### [ TypeScript ]

```
import * as cdk from '@aws-cdk/core';
import * as lambda from '@aws-cdk/aws-lambda';
import * as path from 'path';

export class HelloAssetStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    new lambda.Function(this, 'myLambdaFunction', {
      code: lambda.Code.fromAsset(path.join(__dirname, 'handler')),
      runtime: lambda.Runtime.PYTHON_3_6,
      handler: 'index.lambda_handler'
    });
  }
}
```

------
#### [ JavaScript ]

```
const cdk = require('@aws-cdk/core');
const lambda = require('@aws-cdk/aws-lambda');
const path = require('path');

class HelloAssetStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    new lambda.Function(this, 'myLambdaFunction', {
      code: lambda.Code.fromAsset(path.join(__dirname, 'handler')),
      runtime: lambda.Runtime.PYTHON_3_6,
      handler: 'index.lambda_handler'
    });
  }
}

module.exports = { HelloAssetStack }
```

------
#### [ Python ]

```
from aws_cdk.core import Stack, Construct
from aws_cdk import aws_lambda as lambda_

import os.path
dirname = os.path.dirname(__file__)

class HelloAssetStack(Stack):
    def __init__(self, scope: Construct, id: str, **kwargs):
        super().__init__(scope, id, **kwargs)

        lambda_.Function(self, 'myLambdaFunction',
            code=lambda_.Code.from_asset(os.path.join(dirname, 'handler')),
            runtime=lambda_.Runtime.PYTHON_3_6,
            handler="index.lambda_handler")
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;

public class HelloAssetStack extends Stack {

    public HelloAssetStack(final App scope, final String id) {
        this(scope, id, null);
    }

    public HelloAssetStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);

        File startDir = new File(System.getProperty("user.dir"));

        Function.Builder.create(this, "myLambdaFunction")
                .code(Code.fromAsset(new File(startDir, "handler").toString()))
                .runtime(Runtime.PYTHON_3_6)
                .handler("index.lambda_handler").build();        
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using System.IO;

public class HelloAssetStack : Stack
{
    public HelloAssetStack(Construct scope, string id, StackProps props) : base(scope, id, props)
    {
        new Function(this, "myLambdaFunction", new FunctionProps
        {
            Code = Code.FromAsset(Path.Combine(Directory.GetCurrentDirectory(), "handler")),
            Runtime = Runtime.PYTHON_3_6,
            Handler = "index.lambda_handler"
        });
    }
}
```

------

The `Function` method uses assets to bundle the contents of the directory and use it for the function's code\.

#### Deploy\-time attributes example<a name="assets_types_s3_deploy"></a>

Amazon S3 asset types also expose [deploy\-time attributes](resources.md#resources_attributes) that can be referenced in AWS CDK libraries and apps\. The AWS CDK CLI command cdk synth displays asset properties as AWS CloudFormation parameters\.

The following example uses deploy\-time attributes to pass the location of an image asset into a Lambda function as environment variables\.

------
#### [ TypeScript ]

```
import { Asset } from '@aws-cdk/aws-s3-assets';
import * as path from 'path';

const imageAsset = new Asset(this, "SampleAsset", {
  path: path.join(__dirname, "images/my-image.png")
});

new lambda.Function(this, "myLambdaFunction", {
  code: lambda.Code.asset(path.join(__dirname, "handler")),
  runtime: lambda.Runtime.PYTHON_3_6,
  handler: "index.lambda_handler",
  environment: {
    'S3_BUCKET_NAME': imageAsset.s3BucketName,
    'S3_OBJECT_KEY': imageAsset.s3ObjectKey,
    'S3_URL': imageAsset.s3Url
  }
});
```

------
#### [ JavaScript ]

```
const { Asset } = require('@aws-cdk/aws-s3-assets');
const path = require('path');

const imageAsset = new Asset(this, "SampleAsset", {
  path: path.join(__dirname, "images/my-image.png")
});

new lambda.Function(this, "myLambdaFunction", {
  code: lambda.Code.asset(path.join(__dirname, "handler")),
  runtime: lambda.Runtime.PYTHON_3_6,
  handler: "index.lambda_handler",
  environment: {
    'S3_BUCKET_NAME': imageAsset.s3BucketName,
    'S3_OBJECT_KEY': imageAsset.s3ObjectKey,
    'S3_URL': imageAsset.s3Url
  }
});
```

------
#### [ Python ]

```
import os.path

from aws_cdk import aws_lambda as lambda_
from aws_cdk.aws_s3_assets import Asset

dirname = os.path.dirname(__file__)

image_asset = Asset(self, "SampleAsset",
    path=os.path.join(dirname, "images/my-image.png"))

lambda_.Function(self, "myLambdaFunction",
    code=lambda_.Code.asset(os.path.join(dirname, "handler")),
    runtime=lambda_.Runtime.PYTHON_3_6,
    handler="index.lambda_handler",
    environment=dict(
        S3_BUCKET_NAME=image_asset.s3_bucket_name,
        S3_OBJECT_KEY=image_asset.s3_object_key,
        S3_URL=image_asset.s3_url))
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.s3.assets.Asset;

public class FunctionStack extends Stack {
    public FunctionStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);

        File startDir = new File(System.getProperty("user.dir"));

        Asset imageAsset = Asset.Builder.create(this, "SampleAsset")
                .path(new File(startDir, "images/my-image.png").toString()).build())

        Function.Builder.create(this, "myLambdaFunction")
                .code(Code.fromAsset(new File(startDir, "handler").toString()))
                .runtime(Runtime.PYTHON_3_6)
                .handler("index.lambda_handler")
                .environment(new HashMap<String, String>() {{
                    put("S3_BUCKET_NAME", imageAsset.getS3BucketName());
                    put("S3_OBJECT_KEY", imageAsset.getS3ObjectKey());
                    put("S3_URL", imageAsset.getS3Url());
                }}).build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using Amazon.CDK.AWS.S3.Assets;
using System.IO;
using System.Collections.Generic;

var imageAsset = new Asset(this, "SampleAsset", new AssetProps
{
    Path = Path.Combine(Directory.GetCurrentDirectory(), @"images\my-image.png")
});

new Function(this, "myLambdaFunction", new FunctionProps
{
    Code = Code.FromAsset(Path.Combine(Directory.GetCurrentDirectory(), "handler")),
    Runtime = Runtime.PYTHON_3_6,
    Handler = "index.lambda_handler",
    Environment = new Dictionary<string, string>
    {
        ["S3_BUCKET_NAME"] = imageAsset.S3BucketName,
        ["S3_OBJECT_KEY"] = imageAsset.S3ObjectKey,
        ["S3_URL"] = imageAsset.S3Url
    }
});
```

------

#### Permissions<a name="assets_types_s3_permissions"></a>

If you use Amazon S3 assets directly through the [aws\-s3\-assets](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-s3-assets-readme.html) module, IAM roles, users, or groups, and need to read assets in runtime, grant those assets IAM permissions through the [asset\.grantRead](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3-assets.Asset.html#grant-readgrantee) method\.

The following example grants an IAM group read permissions on a file asset\.

------
#### [ TypeScript ]

```
import { Asset } from '@aws-cdk/aws-s3-assets';
import * as path from 'path';

const asset = new Asset(this, 'MyFile', {
  path: path.join(__dirname, 'my-image.png')
});

const group = new iam.Group(this, 'MyUserGroup');
asset.grantRead(group);
```

------
#### [ JavaScript ]

```
const { Asset } = require('@aws-cdk/aws-s3-assets');
const path = require('path');

const asset = new Asset(this, 'MyFile', {
  path: path.join(__dirname, 'my-image.png')
});

const group = new iam.Group(this, 'MyUserGroup');
asset.grantRead(group);
```

------
#### [ Python ]

```
from aws_cdk.aws_s3_assets import Asset
from aws_cdk import aws_iam as iam

import os.path
dirname = os.path.dirname(__file__)

        asset = Asset(self, "MyFile",
            path=os.path.join(dirname, "my-image.png"))

        group = iam.Group(self, "MyUserGroup")
        asset.grantRead(group)
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.iam.Group;
import software.amazon.awscdk.services.s3.assets.Asset;

public class GrantStack extends Stack {
    public GrantStack(final App scope, final String id, final StackProps props) {
        super(scope, id, props);

        File startDir = new File(System.getProperty("user.dir"));

        Asset asset = Asset.Builder.create(this, "SampleAsset")
                .path(new File(startDir, "images/my-image.png").toString()).build();

        Group group = new Group(this, "MyUserGroup");
        asset.grantRead(group);   }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Amazon.CDK.AWS.IAM;
using Amazon.CDK.AWS.S3.Assets;
using System.IO;

var asset = new Asset(this, "MyFile", new AssetProps {
    Path = Path.Combine(Path.Combine(Directory.GetCurrentDirectory(), @"images\my-image.png"))
});

var group = new Group(this, "MyUserGroup");
asset.GrantRead(group);
```

------

### Docker image assets<a name="assets_types_docker"></a>

The AWS CDK supports bundling local Docker images as assets through the [aws\-ecr\-assets](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ecr-assets-readme.html) module\.

The following example defines a docker image that is built locally and pushed to Amazon ECR\. Images are built from a local Docker context directory \(with a Dockerfile\) and uploaded to Amazon ECR by the AWS CDK CLI or your app's CI/CD pipeline, and can be naturally referenced in your AWS CDK app\. 

------
#### [ TypeScript ]

```
import { DockerImageAsset } from '@aws-cdk/aws-ecr-assets';

const asset = new DockerImageAsset(this, 'MyBuildImage', {
  directory: path.join(__dirname, 'my-image')
});
```

------
#### [ JavaScript ]

```
const { DockerImageAsset } = require('@aws-cdk/aws-ecr-assets');

const asset = new DockerImageAsset(this, 'MyBuildImage', {
  directory: path.join(__dirname, 'my-image')
});
```

------
#### [ Python ]

```
from aws_cdk.aws_ecr_assets import DockerImageAsset

import os.path
dirname = os.path.dirname(__file__)

asset = DockerImageAsset(self, 'MyBuildImage',
    directory=os.path.join(dirname, 'my-image'))
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.ecr.assets.DockerImageAsset;

File startDir = new File(System.getProperty("user.dir"));

DockerImageAsset asset = DockerImageAsset.Builder.create(this, "MyBuildImage")
            .directory(new File(startDir, "my-image").toString()).build();
```

------
#### [ C\# ]

```
using System.IO;
using Amazon.CDK.AWS.Ecr.Assets;

var asset = new DockerImageAsset(this, "MyBuildImage", new DockerImageAssetProps
{
    Directory = Path.Combine(Path.Combine(Directory.GetCurrentDirectory(), "my-image"))
});
```

------

The `my-image` directory must include a Dockerfile\. The AWS CDK CLI builds a Docker image from `my-image`, pushes it to an Amazon ECR repository, and specifies the name of the repository as an AWS CloudFormation parameter to your stack\. Docker image asset types expose [deploy\-time attributes](resources.md#resources_attributes) that can be referenced in AWS CDK libraries and apps\. The AWS CDK CLI command cdk synth displays asset properties as AWS CloudFormation parameters\.

#### Amazon ECS task definition example<a name="assets_types_docker_ecs"></a>

A common use case is to create an Amazon ECS [TaskDefinition](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecs.TaskDefinition.html) to run docker containers\. The following example specifies the location of a Docker image asset that the AWS CDK builds locally and pushes to Amazon ECR\. 

------
#### [ TypeScript ]

```
import * as ecs from '@aws-cdk/aws-ecs';
import * as path from 'path';

const taskDefinition = new ecs.FargateTaskDefinition(this, "TaskDef", {
  memoryLimitMiB: 1024,
  cpu: 512
});

taskDefinition.addContainer("my-other-container", {
  image: ecs.ContainerImage.fromAsset(path.join(__dirname, "..", "demo-image"))
});
```

------
#### [ JavaScript ]

```
const ecs = require('@aws-cdk/aws-ecs');
const path = require('path');

const taskDefinition = new ecs.FargateTaskDefinition(this, "TaskDef", {
  memoryLimitMiB: 1024,
  cpu: 512
});

taskDefinition.addContainer("my-other-container", {
  image: ecs.ContainerImage.fromAsset(path.join(__dirname, "..", "demo-image"))
});
```

------
#### [ Python ]

```
import aws_cdk.aws_ecs as ecs

import os.path
dirname = os.path.dirname(__file__)

task_definition = ecs.FargateTaskDefinition(self, "TaskDef",
    memory_limit_mib=1024,
    cpu=512)

task_definition.add_container("my-other-container",
    image=ecs.ContainerImage.from_asset(
        os.path.join(dirname, "..", "demo-image")))
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.services.ecs.FargateTaskDefinition;
import software.amazon.awscdk.services.ecs.ContainerDefinitionOptions;
import software.amazon.awscdk.services.ecs.ContainerImage;

File startDir = new File(System.getProperty("user.dir"));

FargateTaskDefinition taskDefinition = FargateTaskDefinition.Builder.create(
        this, "TaskDef").memoryLimitMiB(1024).cpu(512).build();

taskDefinition.addContainer("my-other-container",
        ContainerDefinitionOptions.builder()
            .image(ContainerImage.fromAsset(new File(startDir,
                        "demo-image").toString())).build());
```

------
#### [ C\# ]

```
using System.IO;
using Amazon.CDK.AWS.ECS;

var taskDefinition = new FargateTaskDefinition(this, "TaskDef", new FargateTaskDefinitionProps
{
    MemoryLimitMiB = 1024,
    Cpu = 512
});

taskDefinition.AddContainer("my-other-container", new ContainerDefinitionOptions
{
    Image = ContainerImage.FromAsset(Path.Combine(Directory.GetCurrentDirectory(), "demo-image");
});
```

------

#### Deploy\-time attributes example<a name="assets_types_docker_deploy"></a>

The following example shows how to use the deploy\-time attributes `repository` and `imageUri` to create an Amazon ECS task definition with the AWS Fargate launch type\. Note that the Amazon ECR repo lookup requires the image's tag, not its URI, so we snip it from the end of the asset's URI\.

------
#### [ TypeScript ]

```
import * as ecs from '@aws-cdk/aws-ecs';
import * as path from 'path';
import { DockerImageAsset } from '@aws-cdk/aws-ecr-assets';

const asset = new DockerImageAsset(this, 'my-image', {
  directory: path.join(__dirname, "..", "demo-image")
});

const taskDefinition = new ecs.FargateTaskDefinition(this, "TaskDef", {
  memoryLimitMiB: 1024,
  cpu: 512
});

taskDefinition.addContainer("my-other-container", {
  image: ecs.ContainerImage.fromEcrRepository(asset.repository, asset.imageUri.split(":").pop())
});
```

------
#### [ JavaScript ]

```
const ecs = require('@aws-cdk/aws-ecs');
const path = require('path');
const { DockerImageAsset } = require('@aws-cdk/aws-ecr-assets');

const asset = new DockerImageAsset(this, 'my-image', {
  directory: path.join(__dirname, "..", "demo-image")
});

const taskDefinition = new ecs.FargateTaskDefinition(this, "TaskDef", {
  memoryLimitMiB: 1024,
  cpu: 512
});

taskDefinition.addContainer("my-other-container", {
  image: ecs.ContainerImage.fromEcrRepository(asset.repository, asset.imageUri.split(":").pop())
});
```

------
#### [ Python ]

```
import aws_cdk.aws_ecs as ecs
from aws_cdk.aws_ecr_assets import DockerImageAsset

import os.path
dirname = os.path.dirname(__file__)

asset = DockerImageAsset(self, 'my-image',
    directory=os.path.join(dirname, "..", "demo-image"))

task_definition = ecs.FargateTaskDefinition(self, "TaskDef",
    memory_limit_mib=1024, cpu=512)

task_definition.add_container("my-other-container",
    image=ecs.ContainerImage.fromEcrRepository(
        asset.repository, asset.image_uri.rpartition(":")[-1]))
```

------
#### [ Java ]

```
import java.io.File;

import software.amazon.awscdk.services.ecr.assets.DockerImageAsset;

import software.amazon.awscdk.services.ecs.FargateTaskDefinition;
import software.amazon.awscdk.services.ecs.ContainerDefinitionOptions;
import software.amazon.awscdk.services.ecs.ContainerImage;

File startDir = new File(System.getProperty("user.dir"));

DockerImageAsset asset = DockerImageAsset.Builder.create(this, "my-image")
            .directory(new File(startDir, "demo-image").toString()).build();

FargateTaskDefinition taskDefinition = FargateTaskDefinition.Builder.create(
        this, "TaskDef").memoryLimitMiB(1024).cpu(512).build();

// extract the tag from the asset's image URI for use in ECR repo lookup
String imageUri = asset.getImageUri();
String imageTag = imageUri.substring(imageUri.lastIndexOf(":") + 1);

taskDefinition.addContainer("my-other-container",
        ContainerDefinitionOptions.builder().image(ContainerImage.fromEcrRepository(
                asset.getRepository(), imageTag)).build());
```

------
#### [ C\# ]

```
using System.IO;
using Amazon.CDK.AWS.ECS;
using Amazon.CDK.AWS.Ecr.Assets;

var asset = new DockerImageAsset(this, "my-image", new DockerImageAssetProps {
    Directory = Path.Combine(Directory.GetCurrentDirectory(), "demo-image")
});

var taskDefinition = new FargateTaskDefinition(this, "TaskDef", new FargateTaskDefinitionProps
{
    MemoryLimitMiB = 1024,
    Cpu = 512
});

taskDefinition.AddContainer("my-other-container", new ContainerDefinitionOptions
{
    Image = ContainerImage.FromEcrRepository(asset.Repository, asset.ImageUri.Split(":").Last())
});
```

------

#### Build arguments example<a name="assets_types_docker_build"></a>

You can provide customized build arguments for the Docker build step through the `buildArgs` \(Python: `build_args`\) property option when the AWS CDK CLI builds the image during deployment\.

------
#### [ TypeScript ]

```
const asset = new DockerImageAsset(this, 'MyBuildImage', {
  directory: path.join(__dirname, 'my-image'),
  buildArgs: {
    HTTP_PROXY: 'http://10.20.30.2:1234'
  }
});
```

------
#### [ JavaScript ]

```
const asset = new DockerImageAsset(this, 'MyBuildImage', {
  directory: path.join(__dirname, 'my-image'),
  buildArgs: {
    HTTP_PROXY: 'http://10.20.30.2:1234'
  }
});
```

------
#### [ Python ]

```
asset = DockerImageAsset(self, "MyBulidImage",
    directory=os.path.join(dirname, "my-image"),
    build_args=dict(HTTP_PROXY="http://10.20.30.2:1234"))
```

------
#### [ Java ]

```
DockerImageAsset asset = DockerImageAsset.Builder.create(this, "my-image"),
            .directory(new File(startDir, "my-image").toString())
            .buildArgs(new HashMap<String, String>() {{
                put("HTTP_PROXY", "http://10.20.30.2:1234");
            }}).build();
```

------
#### [ C\# ]

```
var asset = new DockerImageAsset(this, "MyBuildImage", new DockerImageAssetProps {
    Directory = Path.Combine(Directory.GetCurrentDirectory(), "my-image"),
    BuildArgs = new Dictionary<string, string>
    {
        ["HTTP_PROXY"] = "http://10.20.30.2:1234"
    }
});
```

------

#### Permissions<a name="assets_types_docker_permissions"></a>

If you use a module that supports Docker image assets, such as [aws\-ecs](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ecs-readme.html), the AWS CDK manages permissions for you when you use assets directly or through [ContainerImage\.fromEcrRepository](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecs.ContainerImage.html#static-from-ecr-repositoryrepository-tag) \(Python: `from_ecr_repository`\)\. If you use Docker image assets directly, you need to ensure that the consuming principal has permissions to pull the image\. 

In most cases, you should use [asset\.repository\.grantPull](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecr.Repository.html#grant-pullgrantee) method \(Python: `grant_pull`\. This modifies the IAM policy of the principal to enable it to pull images from this repository\. If the principal that is pulling the image is not in the same account or is an AWS service, such as AWS CodeBuild, that does not assume a role in your account, you must grant pull permissions on the resource policy and not on the principal's policy\. Use the [asset\.repository\.addToResourcePolicy](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ecr.Repository.html#add-to-resource-policystatement) method \(Python: `add_to_resource_policy`\) to grant the appropriate principal permissions\. 

## AWS CloudFormation resource metadata<a name="assets_cfn"></a>

**Note**  
This section is relevant only for construct authors\. In certain situations, tools need to know that a certain CFN resource is using a local asset\. For example, you can use the AWS SAM CLI to invoke Lambda functions locally for debugging purposes\. See [SAM CLI](sam.md) for details\.

To enable such use cases, external tools consult a set of metadata entries on AWS CloudFormation resources:
+ `aws:asset:path` – Points to the local path of the asset\.
+ `aws:asset:property` – The name of the resource property where the asset is used\.

Using these two metadata entries, tools can identify that assets are used by a certain resource, and enable advanced local experiences\.

To add these metadata entries to a resource, use the `asset.addResourceMetadata` \(Python: `add_resource_metadata`\)  method\.  