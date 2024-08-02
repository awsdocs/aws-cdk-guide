# Example: Create a CDK app with multiple stacks<a name="stack_how_to_create_multiple_stacks"></a>

You can create an AWS Cloud Development Kit \(AWS CDK\) application containing multiple [stacks](stacks.md)\. When you deploy the AWS CDK app, each stack becomes its own AWS CloudFormation template\. You can also synthesize and deploy each stack individually using the AWS CDK CLI `cdk deploy` command\.

In this example, we cover the following:
+ How to extend the `Stack` class to accept new properties or arguments\.
+ How to use properties to determine which resources the stack contains and their configuration\.
+ How to instantiate multiple stacks from this class\.

The example in this topic uses a Boolean property, named `encryptBucket` \(Python: `encrypt_bucket`\)\. It indicates whether an Amazon S3 bucket should be encrypted\. If so, the stack enables encryption using a key managed by AWS Key Management Service \(AWS KMS\)\. The app creates two instances of this stack, one with encryption and one without\.

## Prerequisites<a name="cdk-how-to-create-multiple-stacks-prereqs"></a>

This example assumes that all [getting started](getting_started.md) steps have been completed\.

## Create a CDK project<a name="cdk-how-to-create-multiple-stacks-create"></a>

First, we create a CDK project using the CDK CLI:

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
source .venv/bin/activate # On Windows, run '.\venv\Scripts\activate' instead
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

## Add an optional parameter<a name="cdk-how-to-create-multiple-stacks-extend-stackprops"></a>

The `props` argument of the `Stack` constructor fulfills the interface `StackProps`\. In this example, we want the stack to accept an additional property to tell us whether to encrypt the Amazon S3 bucket\. To do this, we create an interface or class that includes the property\. This allows the compiler to make sure that the property has a Boolean value and enables auto\-completion for it in your IDE\.

We open our *stack file* in our IDE or editor and add the new interface, class, or argument\. New lines are highlighted in bold:

------
#### [ TypeScript ]

File: `lib/multistack-stack.ts`

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

interface MultiStackProps extends cdk.StackProps {
  encryptBucket?: boolean;
}

export class MultistackStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: MultiStackProps) {
    super(scope, id, props);

    // The code that defines our stack goes here
  }
}
```

------
#### [ JavaScript ]

File: `lib/multistack-stack.js`

JavaScript doesn't have an interface feature; we don't need to add any code\.

```
const cdk = require('aws-cdk-stack');

class MultistackStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // The code that defines our stack goes here
  }
}

module.exports = { MultistackStack }
```

------
#### [ Python ]

File: `multistack/multistack_stack.py`

Python does not have an interface feature, so we'll extend our stack to accept the new property by adding a keyword argument\.

```
import aws_cdk as cdk
from constructs import Construct

class MultistackStack(cdk.Stack):

    # The Stack class doesn't know about our encrypt_bucket parameter,
    # so accept it separately and pass along any other keyword arguments.
    def __init__(self, scope: Construct, id: str, *, encrypt_bucket=False,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # The code that defines our stack goes here
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackStack.java`

It's more complicated than we really want to get into to extend a props type in Java\. Instead, write the stack's constructor to accept an optional Boolean parameter\. Because `props` is an optional argument, we'll write an additional constructor that lets you skip it\. It will default to `false`\.

```
package com.myorg;

import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.constructs.Construct;

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

        // The code that defines our stack goes here
    }
}
```

------
#### [ C\# ]

File: `src/Multistack/MultistackStack.cs`

```
using Amazon.CDK;
using constructs;

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
            // The code that defines our stack goes here
        }
    }
}
```

------

The new property is optional\. If `encryptBucket` \(Python: `encrypt_bucket`\) is not present, its value is `undefined`, or the local equivalent\. The bucket will be unencrypted by default\.

## Define the stack class<a name="cdk-how-to-create-multiple-stacks-define-stack"></a>

 Next, we define our stack class, using our new property\. New code is highlighted in bold:

------
#### [ TypeScript ]

File: `lib/multistack-stack.ts`

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from constructs;
import * as s3 from 'aws-cdk-lib/aws-s3';

interface MultistackProps extends cdk.StackProps {
  encryptBucket?: boolean;
}

export class MultistackStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: MultistackProps) {
    super(scope, id, props);
    
    // Add a Boolean property "encryptBucket" to the stack constructor.
    // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
    // Encrypted bucket uses KMS-managed keys (SSE-KMS).
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
const cdk = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');

class MultistackStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);
    
    // Add a Boolean property "encryptBucket" to the stack constructor.
    // If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
    // Encrypted bucket uses KMS-managed keys (SSE-KMS).
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
import aws_cdk as cdk
from constructs import Construct
from aws_cdk import aws_s3 as s3

class MultistackStack(cdk.Stack):

    # The Stack class doesn't know about our encrypt_bucket parameter,
    # so accept it separately and pass along any other keyword arguments.
    def __init__(self, scope: Construct, id: str, *, encrypt_bucket=False,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Add a Boolean property "encryptBucket" to the stack constructor.
        # If true, creates an encrypted bucket. Otherwise, the bucket is unencrypted.
        # Encrypted bucket uses KMS-managed keys (SSE-KMS).
        if encrypt_bucket:
            s3.Bucket(self, "MyGroovyBucket",
                      encryption=s3.BucketEncryption.KMS_MANAGED,
                      removal_policy=cdk.RemovalPolicy.DESTROY)
        else:
            s3.Bucket(self, "MyGroovyBucket",
                     removal_policy=cdk.RemovalPolicy.DESTROY)
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackStack.java`

```
package com.myorg;

import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.constructs.Construct;
import software.amazon.awscdk.RemovalPolicy;

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
        // unencrypted. Encrypted bucket uses KMS-managed keys (SSE-KMS).
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
            // Encrypted bucket uses KMS-managed keys (SSE-KMS).
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

In our *application file*, we add the code to instantiate two separate stacks\. We delete the existing `MultistackStack` definition and define our two stacks\. New code is highlight in bold:

------
#### [ TypeScript ]

File: `bin/multistack.ts`

```
#!/usr/bin/env node
import 'source-map-support/register';
import * as cdk from 'aws-cdk-lib';
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
          
app.synth();
```

------
#### [ JavaScript ]

File: `bin/multistack.js`

```
#!/usr/bin/env node
const cdk = require('aws-cdk-lib');
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
          
app.synth();
```

------
#### [ Python ]

File: `./app.py`

```
#!/usr/bin/env python3

import aws_cdk as cdk

from multistack.multistack_stack import MultistackStack

app = cdk.App()
MultistackStack(app, "MyWestCdkStack",
                env=cdk.Environment(region="us-west-1"),
                encrypt_bucket=False)

MultistackStack(app, "MyEastCdkStack",
                env=cdk.Environment(region="us-east-1"),
                encrypt_bucket=True)
            
app.synth()
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MultistackApp.java`

```
package com.myorg;

import software.amazon.awscdk.App;
import software.amazon.awscdk.Environment;
import software.amazon.awscdk.StackProps;

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

Next, we can deploy stacks from the app\. First, we synthesize an AWS CloudFormation template for `MyEastCdkStack`\. This is the stack in `us-east-1` with the encrypted Amazon S3 bucket\.

```
$ cdk synth MyEastCdkStack
```

To deploy this stack to our AWS environment, we can issue one of the following commands\. The first command uses our default AWS profile to obtain the credentials to deploy the stack\. The second uses a profile that we specify\. For *PROFILE\_NAME*, we can substitute the name of an AWS CLI profile that contains appropriate credentials for deploying to the `us-east-1` AWS Region\.

```
$ cdk deploy MyEastCdkStack
```

```
$ cdk deploy MyEastCdkStack --profile=PROFILE_NAME
```

## Clean up<a name="cdk-how-to-create-multiple-stacks-destroy-stack"></a>

To avoid charges for resources that we deployed, we destroy the stack using the following command:

```
cdk destroy MyEastCdkStack
```

The destroy operation fails if there is anything stored in the stack's bucket\. There shouldn't be, since we only created the bucket\. If we did put something in the bucket, we must delete the bucket contents before destroying the stack\. We can use the AWS Management Console or the AWS CLI to delete the bucket contents\.