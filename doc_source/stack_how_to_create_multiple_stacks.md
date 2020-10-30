# Create an app with multiple stacks<a name="stack_how_to_create_multiple_stacks"></a>

Most of the other code examples in the *AWS CDK Developer Guide* involve only a single stack\. However, you can create apps containing any number of stacks\. Each stack results in its own AWS CloudFormation template\. Stacks are the *unit of deployment:* each stack in an app can be synthesized and deployed individually using the `cdk deploy` command\.

This topic illustrates how to extend the `Stack` class to accept new properties or arguments, how to use these properties to affect what resources the stack contains and their configuration, and how to instantiate multiple stacks from this class\. The example uses a Boolean property, named `encryptBucket` \(Python: `encrypt_bucket`\), to indicate whether an Amazon S3 bucket should be encrypted\. If so, the stack enables encryption using a key managed by AWS Key Management Service \(AWS KMS\)\. The app creates two instances of this stack, one with encryption and one without\.

## Before you begin<a name="cdk-how-to-create-multiple-stacks-prereqs"></a>

First, install Node\.js and the AWS CDK command line tools, if you haven't already\. See [Getting started with the AWS CDK](getting_started.md) for details\.

Next, create an AWS CDK project by entering the following commands at the command line\.

------
#### [ TypeScript ]

```
mkdir multistack
cd multistack
cdk init --language=typescript
```

------
#### [ JavaScript ]

```
mkdir multistack
cd multistack
cdk init --language=javascript
```

------
#### [ Python ]

```
mkdir multistack
cd multistack
cdk init --language=python
source .venv/bin/activate
pip install -r requirements.txt
```

------
#### [ Java ]

```
mkdir multistack
cd multistack
cdk init --language=java
```

You can import the resulting Maven project into your Java IDE\.

------
#### [ C\# ]

```
mkdir multistack
cd multistack
cdk init --language=csharp
```

You can open the file `src/Pipeline.sln` in Visual Studio\.

------

Finally, install the `core` and `s3` AWS Construct Library modules\. We use these modules in our app\.

------
#### [ TypeScript ]

```
npm install @aws-cdk/core @aws-cdk/aws-s3
```

------
#### [ JavaScript ]

```
npm install @aws-cdk/core @aws-cdk/aws-s3
```

------
#### [ Python ]

```
pip install aws_cdk.core aws_cdk.aws_s3
```

------
#### [ Java ]

Using the Maven integration in your IDE \(for example, in Eclipse, right\-click the project and choose **Maven** > **Add Dependency**\), add the following packages in the group `software.amazon.awscdk`\.

```
core
s3
```

------
#### [ C\# ]

```
nuget install Amazon.CDK
nuget install Amazon.CDK.AWS.S3
```

Or **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution** in Visual Studio

**Tip**  
If you don't see these packages in the **Browse** tab of the **Manage Packages for Solution** page, make sure the **Include prerelease** checkbox is ticked\.  
For a better experience, also add the `Amazon.Jsii.Analyzers` package to provide compile\-time checks for missing required properties\.

------

## Add optional parameter<a name="cdk-how-to-create-multiple-stacks-extend-stackprops"></a>

The `props` argument of the `Stack` constructor fulfills the interface `StackProps`\. Because we want our stack to accept an additional property to tell us whether to encrypt the Amazon S3 bucket, we should create an interface or class that includes that property\. This allows the compiler to make sure the property has a Boolean value and enables autocompletion for it in your IDE\.

So open the indicated source file in your IDE or editor and add the new interface, class, or argument\. The code should look like this after the changes\. The lines we added are shown in boldface\.

------
#### [ TypeScript ]

File: `lib/multistack-stack.ts`

```
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

interface MultiStackProps extends cdk.StackProps {
  encryptBucket?: boolean;
}

export class MultistackStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: MultiStackProps) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}
```

------
#### [ JavaScript ]

File: `lib/multistack-stack.js`

JavaScript doesn't have an interface feature; we don't need to add any code\.

```
const cdk = require('@aws-cdk/core');

class MultistackStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // The code that defines your stack goes here
  }
}

module.exports = { MultistackStack }
```

------
#### [ Python ]

File: `multistack/multistack_stack.py`

Python does not have an interface feature, so we'll extend our stack to accept the new property by adding a keyword argument\.

```
from aws_cdk import aws_s3 as s3

class MultistackStack(core.Stack):

    # The Stack class doesn't know about our encrypt_bucket parameter,
    # so accept it separately and pass along any other keyword arguments.
    def __init__(self, scope: core.Construct, id: str, *, encrypt_bucket=False,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # The code that defines your stack goes here
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackStack.java`

It's more complicated than we really want to get into to extend a props type in Java, so we'll simply write our stack's constructor to accept an optional Boolean parameter\. Since `props` is an optional argument, we'll write an additional constructor that allows you to skip it\. It will default to `false`\.

```
package com.myorg;

import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.core.Construct;

import software.amazon.awscdk.services.s3.Bucket;

public class MultistackStack extends Stack {
    // additional constructors to allow props and/or encryptBucket to be omitted
    public MultistackStack(final Construct scope, final String id, boolean encryptBucket) {
        this(scope, id, null, encryptBucket);
    }
    
    public MultistackStack(final Construct scope, final String id) {
        this(scope, id, null, false);
    }

    public MultistackStack(final Construct scope, final String id, final StackProps props,
            final boolean encryptBucket) {
        super(scope, id, props);

        // The code that defines your stack goes here
    }
}
```

------
#### [ C\# ]

File: `src/Multistack/MultistackStack.cs`

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;
namespace Multistack
{
    

    public class MultiStackProps : StackProps
    {
        public bool? EncryptBucket { get; set; }
    }


    public class MultistackStack : Stack
    {
        public MultistackStack(Construct scope, string id, MultiStackProps props) : base(scope, id, props)
        {
            // The code that defines your stack goes here
        }
    }
}
```

------

The new property is optional\. If `encryptBucket` \(Python: `encrypt_bucket`\) is not present, its value is `undefined`, or the local equivalent\. The bucket will be unencrypted by default\.

## Define the stack class<a name="cdk-how-to-create-multiple-stacks-define-stack"></a>

 Now let's define our stack class, using our new property\. Make the code look like the following\. The code you need to add or change is shown in boldface\. 

------
#### [ TypeScript ]

File: `lib/multistack-stack.ts`

```
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';

interface MultistackProps extends cdk.StackProps {
  encryptBucket?: boolean;
}

export class MultistackStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: MultistackProps) {
    super(scope, id, props);
    
    // Add a Boolean property "encryptBucket" to the stack constructor.
    // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
    // Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
    if (props && props.encryptBucket) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KMS_MANAGED, 
        removalPolicy: cdk.RemovalPolicy.DESTROY
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket", {
        removalPolicy: cdk.RemovalPolicy.DESTROY});
    }
  }
}
```

------
#### [ JavaScript ]

File: `lib/multistack-stack.js`

```
const cdk = require('@aws-cdk/core');
const s3 = require('@aws-cdk/aws-s3');

class MultistackStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);
    
    // Add a Boolean property "encryptBucket" to the stack constructor.
    // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
    // Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
    if ( props && props.encryptBucket) {
      new s3.Bucket(this, "MyGroovyBucket", {
        encryption: s3.BucketEncryption.KMS_MANAGED, 
        removalPolicy: cdk.RemovalPolicy.DESTROY
      });
    } else {
      new s3.Bucket(this, "MyGroovyBucket", {
        removalPolicy: cdk.RemovalPolicy.DESTROY});
    }
  }
}

module.exports = { MultistackStack }
```

------
#### [ Python ]

File: `multistack/multistack_stack.py`

```
from aws_cdk import core
from aws_cdk import aws_s3 as s3

class MultistackStack(core.Stack):

    # The Stack class doesn't know about our encrypt_bucket parameter,
    # so accept it separately and pass along any other keyword arguments.
    def __init__(self, scope: core.Construct, id: str, *, encrypt_bucket=False,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Add a Boolean property "encryptBucket" to the stack constructor.
        # If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
        # Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
        if encrypt_bucket:
            s3.Bucket(self, "MyGroovyBucket",
                      encryption=s3.BucketEncryption.KMS_MANAGED,
                      removal_policy=core.RemovalPolicy.DESTROY)
        else:
            s3.Bucket(self, "MyGroovyBucket",
                     removal_policy=core.RemovalPolicy.DESTROY)
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackStack.java`

```
package com.myorg;

import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.RemovalPolicy;

import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.s3.BucketEncryption;

public class MultistackStack extends Stack {
    // additional constructors to allow props and/or encryptBucket to be omitted
    public MultistackStack(final Construct scope, final String id,
            boolean encryptBucket) {
        this(scope, id, null, encryptBucket);
    }

    public MultistackStack(final Construct scope, final String id) {
        this(scope, id, null, false);
    }

    // main constructor
    public MultistackStack(final Construct scope, final String id,
            final StackProps props, final boolean encryptBucket) {
        super(scope, id, props);

        // Add a Boolean property "encryptBucket" to the stack constructor.
        // If true, creates an encrypted bucket. Otherwise, the bucket is
        // unencrypted. Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
        if (encryptBucket) {
            Bucket.Builder.create(this, "MyGroovyBucket")
                    .encryption(BucketEncryption.KMS_MANAGED)
                    .removalPolicy(RemovalPolicy.DESTROY).build();
        } else {
            Bucket.Builder.create(this, "MyGroovyBucket")
                    .removalPolicy(RemovalPolicy.DESTROY).build();
        }
    }
}
```

------
#### [ C\# ]

File: `src/Multistack/MultistackStack.cs`

```
using Amazon.CDK;
using Amazon.CDK.AWS.S3;

namespace Multistack
{
    
    public class MultiStackProps : StackProps
    {
        public bool? EncryptBucket { get; set; }
    }

    public class MultistackStack : Stack
    {
        public MultistackStack(Construct scope, string id, IMultiStackProps props = null) : base(scope, id, props)
        {
            // Add a Boolean property "EncryptBucket" to the stack constructor.
            // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
            // Encrypted bucket uses AWS KMS-managed keys (SSE-KMS).
            if (props?.EncryptBucket ?? false)
            {
                new Bucket(this, "MyGroovyBucket", new BucketProps
                {
                    Encryption = BucketEncryption.KMS_MANAGED,
                    RemovalPolicy = RemovalPolicy.DESTROY
                });
            }
            else
            {
                new Bucket(this, "MyGroovyBucket", new BucketProps
                {
                    RemovalPolicy = RemovalPolicy.DESTROY
                });
            }
        }
    }
}
```

------

## Create two stack instances<a name="stack_how_to_create_multiple_stacks-create-stacks"></a>

Now we'll add the code to instantiate two separate stacks\. As before, the lines of code shown in boldface are the ones you need to add\. Delete the existing `MultistackStack` definition\.

------
#### [ TypeScript ]

File: `bin/multistack.ts`

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from '@aws-cdk/core';
import { MultistackStack } from '../lib/multistack-stack';

const app = new cdk.App();

new MultistackStack(app, "MyWestCdkStack", {
    env: {region: "us-west-1"},
    encryptBucket: false
});
  
new MultistackStack(app, "MyEastCdkStack", {
    env: {region: "us-east-1"},
    encryptBucket: true
});
```

------
#### [ JavaScript ]

File: `bin/multistack.js`

```
#!/usr/bin/env node
const cdk = require('@aws-cdk/core');
const { MultistackStack } = require('../lib/multistack-stack');

const app = new cdk.App();

new MultistackStack(app, "MyWestCdkStack", {
    env: {region: "us-west-1"},
    encryptBucket: false
});
  
new MultistackStack(app, "MyEastCdkStack", {
    env: {region: "us-east-1"},
    encryptBucket: true
});
```

------
#### [ Python ]

File: `./app.py`

```
#!/usr/bin/env python3

from aws_cdk import core

from multistack.multistack_stack import MultistackStack

app = core.App()
MultistackStack(app, "MyWestCdkStack",
                env=core.Environment(region="us-west-1"),
                encrypt_bucket=False)

MultistackStack(app, "MyEastCdkStack",
                env=core.Environment(region="us-east-1"),
                encrypt_bucket=True)
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackApp.java`

```
package com.myorg;

import software.amazon.awscdk.core.App;
import software.amazon.awscdk.core.Environment;
import software.amazon.awscdk.core.StackProps;

public class MultistackApp {
    public static void main(final String argv[]) {
        App app = new App();

        new MultistackStack(app, "MyWestCdkStack", StackProps.builder()
                .env(Environment.builder()
                        .region("us-west-1")
                        .build())
                .build(), false);

        new MultistackStack(app, "MyEastCdkStack", StackProps.builder()
                .env(Environment.builder()
                        .region("us-east-1")
                        .build())
                .build(), true);

        app.synth();
    }
}
```

------
#### [ C\# ]

File: src/Multistack/Program\.cs

```
using Amazon.CDK;

namespace Multistack
{
    class Program
    {
        static void Main(string[] args)
        {
            var app = new App();

            new MultistackStack(app, "MyWestCdkStack", new MultiStackProps
            {
                Env = new Environment { Region = "us-west-1" },
                EncryptBucket = false
            });

            new MultistackStack(app, "MyEastCdkStack", new MultiStackProps
            {
                Env = new Environment { Region = "us-east-1" },
                EncryptBucket = true
            });

            app.Synth();
        }
    }
}
```

------

 This code uses the new `encryptBucket` \(Python: `encrypt_bucket`\) property on the `MultistackStack` class to instantiate the following:
+  One stack with an encrypted Amazon S3 bucket in the `us-east-1` AWS Region\. 
+  One stack with an unencrypted Amazon S3 bucket in the `us-west-1` AWS Region\. 

## Synthesize and deploy the stack<a name="cdk-how-to-create-multiple-stacks-synth-deploy"></a>

Now you can deploy stacks from the app\. First, synthesize a AWS CloudFormation template for `MyEastCdkStack`—the stack in `us-east-1`\. This is the stack with the encrypted S3 bucket\.

```
$ cdk synth MyEastCdkStack
```

The output should look similar to the following AWS CloudFormation template \(there might be slight differences\)\.

```
Resources:
  MyGroovyBucketFD9882AC:
    Type: AWS::S3::Bucket
    Properties:
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
    UpdateReplacePolicy: Retain
    DeletionPolicy: Retain
    Metadata:
      aws:cdk:path: MyEastCdkStack/MyGroovyBucket/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=1.10.0,@aws-cdk/aws-events=1.10.0,@aws-cdk/aws-iam=1.10.0,@aws-cdk/aws-kms=1.10.0,@aws-cdk/aws-s3=1.10.0,@aws-cdk/core=1.10.0,@aws-cdk/cx-api=1.10.0,@aws-cdk/region-info=1.10.0,jsii-runtime=node.js/v10.16.2
```

To deploy this stack to your AWS account, issue one of the following commands\. The first command uses your default AWS profile to obtain the credentials to deploy the stack\. The second uses a profile you specify: for *PROFILE\_NAME*, substitute the name of an AWS CLI profile that contains appropriate credentials for deploying to the `us-east-1` AWS Region\.

```
cdk deploy MyEastCdkStack
```

```
cdk deploy MyEastCdkStack --profile=PROFILE_NAME
```

## Clean up<a name="cdk-how-to-create-multiple-stacks-destroy-stack"></a>

To avoid charges for resources that you deployed, destroy the stack using the following command\.

```
cdk destroy MyEastCdkStack
```

The destroy operation fails if there is anything stored in the stack's bucket\. There shouldn't be if you've only followed the instructions in this topic\. But if you did put something in the bucket, you must delete the bucket's contents, but not the bucket itself, using the AWS Management Console or the AWS CLI before destroying the stack\.