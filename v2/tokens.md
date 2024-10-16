# Tokens and the AWS CDK<a name="tokens"></a>

In the AWS Cloud Development Kit \(AWS CDK\), *tokens* are placeholders for values that aren’t known when defining constructs or synthesizing stacks\. These values will be fully resolved at deployment, when your actual infrastructure is created\. When developing AWS CDK applications, you will work with tokens to manage these values across your application\.

## Token example<a name="tokens-example"></a>

The following is an example of a CDK stack that defines a construct for an Amazon Simple Storage Service \(Amazon S3\) bucket\. Since the name of our bucket is not yet known, the value for `bucketName` is stored as a token:

------
#### [ TypeScript ]

```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';

export class CdkDemoAppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');
    
    // Store value of the S3 bucket name
    const myBucketName = myBucket.bucketName;
    
    // Print the current value for the S3 bucket name at synthesis
    console.log("myBucketName: " + bucketName);
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Duration } = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');

    // Store value of the S3 bucket name
    const myBucketName = myBucket.bucketName;

    // Print the current value for the S3 bucket name at synthesis
    console.log("myBucketName: " + myBucketName);
  }
}

module.exports = { CdkDemoAppStack }
```

------
#### [ Python ]

```
from aws_cdk import (
    Stack
)
from constructs import Construct
from aws_cdk import aws_s3 as s3

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define an S3 bucket
        my_bucket = s3.Bucket(self, "myBucket")

        # Store the value of the S3 bucket name
        my_bucket_name = my_bucket.bucket_name

        # Print the current value for the S3 bucket name at synthesis
        print(f"myBucketName: {my_bucket_name}")
```

------
#### [ Java ]

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.services.s3.Bucket;

import java.util.Map;

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define an S3 bucket
        Bucket myBucket = Bucket.Builder.create(this, "myBucket")
            .build();
        
        // Store the token for the bucket name
        String myBucketName = myBucket.getBucketName();

        // Print the token at synthesis
        System.out.println("myBucketName: " + myBucketName);
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Constructs;
using Amazon.CDK.AWS.S3;

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define an S3 bucket
            var myBucket = new Bucket(this, "myBucket");

            // Store the token for the bucket name
            var myBucketName = myBucket.BucketName;

            // Print the token at synthesis
            System.Console.WriteLine($"myBucketName: {myBucketName}");
        }
    }
}
```

------
#### [ Go ]

```
package main

import (
	"fmt"

	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awss3"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type CdkDemoAppStackProps struct {
	awscdk.StackProps
}

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define an S3 bucket
	myBucket := awss3.NewBucket(stack, jsii.String("myBucket"), &awss3.BucketProps{})

	// Store the token for the bucket name
	myBucketName := myBucket.BucketName()

	// Print the token at synthesis
	fmt.Println("myBucketName: ", *myBucketName)

	return stack
}

// ...
```

------

When we run `cdk synth` to synthesize our stack, the value for `myBucketName` will be displayed in the token format of `${Token[TOKEN.1234]}`\. This token format is a result of how the AWS CDK encodes tokens\. In this example, the token is encoded as a string:

```
$ cdk synth --quiet
myBucketName: ${Token[TOKEN.21]}
```

Since the value for our bucket name is not known at synthesis, the token is rendered as `myBucket<unique-hash>`\. Our AWS CloudFormation template uses the `Ref` intrinsic function to reference its value, which will be known at deployment:

```
Resources:
  myBucket5AF9C99B:
    # ...
Outputs:
  bucketNameOutput:
    Description: The name of the S3 bucket
    Value:
      Ref: myBucket5AF9C99B
```

For more information on how the unique hash is generated, see [Generated logical IDs in your AWS CloudFormation template](configure-synth.md#how-synth-default-logical-ids)\.

## Passing tokens<a name="tokens-passing"></a>

Tokens can be passed around as if they were the actual value they represent\. The following is an example that passes the token for our bucket name to a construct for an AWS Lambda function:

------
#### [ TypeScript ]



```
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as lambda from 'aws-cdk-lib/aws-lambda';

export class CdkDemoAppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');
    
    // ...

    // Define a Lambda function
    const myFunction = new lambda.Function(this, "myFunction", {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: "index.handler",
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
      functionName: myBucketName + "Function", // Pass token for the S3 bucket name
      environment: {
        BUCKET_NAME: myBucketName, // Pass token for the S3 bucket name
      }
    });
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Duration } = require('aws-cdk-lib');
const s3 = require('aws-cdk-lib/aws-s3');
const lambda = require('aws-cdk-lib/aws-lambda');

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');

    // ...

    // Define a Lambda function
    const myFunction = new lambda.Function(this, 'myFunction', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
      functionName: myBucketName + 'Function', // Pass token for the S3 bucket name
      environment: {
        BUCKET_NAME: myBucketName, // Pass token for the S3 bucket name
      }
    });
  }
}

module.exports = { CdkDemoAppStack }
```

------
#### [ Python ]

```
from aws_cdk import (
    Stack
)
from constructs import Construct
from aws_cdk import aws_s3 as s3
from aws_cdk import aws_lambda as _lambda

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define an S3 bucket
        my_bucket = s3.Bucket(self, "myBucket")

        # ...
        
        # Define a Lambda function
        my_function = _lambda.Function(self, "myFunction",
            runtime=_lambda.Runtime.NODEJS_20_X,
            handler="index.handler",
            code=_lambda.Code.from_inline("""
                exports.handler = async function(event) {
                  return {
                    statusCode: 200,
                    body: JSON.stringify('Hello World!'),
                  };
                };
            """),
            function_name=f"{my_bucket_name}Function",  # Pass token for the S3 bucket name
            environment={
                "BUCKET_NAME": my_bucket_name  # Pass token for the S3 bucket name
            }
        )
```

------
#### [ Java ]

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.services.s3.Bucket;
import software.amazon.awscdk.services.lambda.Code;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;

import java.util.Map;

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define an S3 bucket
        Bucket myBucket = Bucket.Builder.create(this, "myBucket")
            .build();
        
        // ...

        // Define a Lambda function
        Function myFunction = Function.Builder.create(this, "myFunction")
            .runtime(Runtime.NODEJS_20_X)
            .handler("index.handler")
            .code(Code.fromInline(
                "exports.handler = async function(event) {" +
                "return {" +
                "statusCode: 200," +
                "body: JSON.stringify('Hello World!')," +
                "};" +
                "};"
            ))
            .functionName(myBucketName + "Function") // Pass the token for the s3 bucket to the function construct
            .environment(Map.of("BUCKET_NAME", myBucketName))  // Pass the bucket name as environment variable
            .build();
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Constructs;
using Amazon.CDK.AWS.S3;
using Amazon.CDK.AWS.Lambda;
using System;
using System.Collections.Generic;

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define an S3 bucket
            var myBucket = new Bucket(this, "myBucket");

            // ...

            // Define a Lambda function
            var myFunction = new Function(this, "myFunction", new FunctionProps
            {
                 Runtime = Runtime.NODEJS_20_X,
                 Handler = "index.handler",
                 Code = Code.FromInline(@"
                     exports.handler = async function(event) {
                       return {
                         statusCode: 200,
                         body: JSON.stringify('Hello World!'),
                       };
                     };
                 "),
                 // Pass the token for the S3 bucket name 
                 Environment = new Dictionary<string, string>
                 {
                     { "BUCKET_NAME", myBucketName }
                 },
                 FunctionName = $"{myBucketName}Function" // Pass the token for the s3 bucket to the function construct
            });
        }
    }
}
```

------
#### [ Go ]

```
package main

import (
	"fmt"

	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
	"github.com/aws/aws-cdk-go/awscdk/v2/awss3"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type CdkDemoAppStackProps struct {
	awscdk.StackProps
}

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define an S3 bucket
	myBucket := awss3.NewBucket(stack, jsii.String("myBucket"), &awss3.BucketProps{})

	// ...

	// Define a Lambda function
	myFunction := awslambda.NewFunction(stack, jsii.String("myFunction"), &awslambda.FunctionProps{
		Runtime: awslambda.Runtime_NODEJS_20_X(),
		Handler: jsii.String("index.handler"),
		Code: awslambda.Code_FromInline(jsii.String(`
			exports.handler = async function(event) {
				return {
					statusCode: 200,
					body: JSON.stringify('Hello World!'),
				};
			};
		`)),
		FunctionName: jsii.String(fmt.Sprintf("%sFunction", *myBucketName)), // Pass the token for the S3 bucket to the function name
		Environment: &map[string]*string{
			"BUCKET_NAME": myBucketName,
		},
	})

	return stack
}
// ...
```

------

When we synthesize our template, the `Ref` and `Fn::Join` intrinsic functions are used to specify the values, which will be known at deployment:

```
Resources:
  myBucket5AF9C99B:
    Type: AWS::S3::Bucket
    # ...
  myFunction884E1557:
    Type: AWS::Lambda::Function
    Properties:
      # ...
      Environment:
        Variables:
          BUCKET_NAME:
            Ref: myBucket5AF9C99B
      FunctionName:
        Fn::Join:
          - ""
          - - Ref: myBucket5AF9C99B
            - Function
      # ...
```

## How token encodings work<a name="tokens-work"></a>

Tokens are objects that implement the `[IResolvable](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.IResolvable.html)` interface, which contains a single `resolve` method\. During synthesis, the AWS CDK calls this method to produce the final value for tokens in your CloudFormation template\.

**Note**  
You'll rarely work directly with the `IResolvable` interface\. You will most likely only see string\-encoded versions of tokens\.

### Token encoding types<a name="tokens-work-types"></a>

Tokens participate in the synthesis process to produce arbitrary values of any type\. Other functions typically only accept arguments of basic types, such as `string` or `number`\. To use tokens in these cases, you can encode them into one of three types by using static methods on the `[cdk\.Token](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html)` class\.
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrstringvalue-options](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrstringvalue-options) to generate a string encoding \(or call `.toString()` on the token object\)\.
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrlistvalue-options](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrlistvalue-options) to generate a list encoding\.
+ [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrnumbervalue](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-aswbrnumbervalue) to generate a numeric encoding\.

These take an arbitrary value, which can be an `IResolvable`, and encode them into a primitive value of the indicated type\.

**Important**  
Because any one of the previous types can potentially be an encoded token, be careful when you parse or try to read their contents\. For example, if you attempt to parse a string to extract a value from it, and the string is an encoded token, your parsing fails\. Similarly, if you try to query the length of an array or perform math operations with a number, you must first verify that they aren't encoded tokens\.

## How to check for tokens in your app<a name="tokens-check"></a>

To check whether a value has an unresolved token in it, call the `[Token\.isUnresolved](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Token.html#static-iswbrunresolvedobj)` \(Python: `is_unresolved`\) method\. The following is an example that checks if the value for our Amazon S3 bucket name is a token\. If its not a token, we then validate the length of the bucket name:

------
#### [ TypeScript ]



```
// ...

export class CdkDemoAppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');
    // ...

    // Check if bucket name is a token. If not, check if length is less than 10 characters
    if (cdk.Token.isUnresolved(myBucketName)) {
      console.log("Token identified.");
    } else if (!cdk.Token.isUnresolved(myBucketName) && myBucketName.length > 10) {
      throw new Error('Maximum length for name is 10 characters.');
    };

    // ...
```

------
#### [ JavaScript ]

```
const { Stack, Duration, Token, CfnOutput } = require('aws-cdk-lib');
// ...

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define an S3 bucket
    const myBucket = new s3.Bucket(this, 'myBucket');

    // ...

    // Check if bucket name is a token. If not, check if length is less than 10 characters
    if (Token.isUnresolved(myBucketName)) {
      console.log("Token identified.");
    } else if (!Token.isUnresolved(myBucketName) && myBucketName.length > 10) {
      throw new Error('Maximum length for name is 10 characters.');
    };

    // ...
```

------
#### [ Python ]



```
from aws_cdk import (
    Stack,
    Token
)
# ...

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define an S3 bucket
        my_bucket = s3.Bucket(self, "myBucket")

        # ...

        # Check if bucket name is a token. If not, check if length is less than 10 characters
        if Token.is_unresolved(my_bucket_name):
            print("Token identified.")
        elif not Token.is_unresolved(my_bucket_name) and len(my_bucket_name) < 10:
            raise ValueError("Maximum length for name is 10 characters.")
        
        # ...
```

------
#### [ Java ]

```
// ...
import software.amazon.awscdk.Token;
// ...

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define an S3 bucket
        Bucket myBucket = Bucket.Builder.create(this, "myBucket")
            .build();
        
        // ...

        // Check if the bucket name is a token. If not, check if length is less than 10 characters
        if (Token.isUnresolved(myBucketName)) {
            System.out.println("Token identified.");
        } else if (!Token.isUnresolved(myBucketName) && myBucketName.length() > 10) {
            throw new IllegalArgumentException("Maximum length for name is 10 characters.");
        }

        // ...
    }
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Constructs;
using Amazon.CDK.AWS.S3;
using Amazon.CDK.AWS.Lambda;
using System;
using System.Collections.Generic;

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define an S3 bucket
            var myBucket = new Bucket(this, "myBucket");

            // ...

            // Check if bucket name is a token. If not, check if length is less than 10 characters
            if (Token.IsUnresolved(myBucketName))
            {
                System.Console.WriteLine("Token identified.");
            }
            else if (!Token.IsUnresolved(myBucketName) && myBucketName.Length > 10)
            {
                throw new System.Exception("Maximum length for name is 10 characters.");
            }

            // ...
```

------
#### [ Go ]

```
// ...

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define an S3 bucket
	myBucket := awss3.NewBucket(stack, jsii.String("myBucket"), &awss3.BucketProps{})

	// ...

	// Check if the bucket name is unresolved (a token)
	if tokenUnresolved := awscdk.Token_IsUnresolved(myBucketName); tokenUnresolved != nil && *tokenUnresolved {
		fmt.Println("Token identified.")
	} else if tokenUnresolved != nil && !*tokenUnresolved && len(*myBucketName) > 10 {
		panic("Maximum length for name is 10 characters.")
	}

	// ...
```

------

When we run `cdk synth`, `myBucketName` is identified as a token:

```
$ cdk synth --quiet
Token identified.
```

**Note**  
You can use token encodings to escape the type system\. For example, you could string\-encode a token that produces a number value at synthesis time\. If you use these functions, it's your responsibility to make sure that your template resolves to a usable state after synthesis\.

## Working with string\-encoded tokens<a name="tokens_string"></a>

String\-encoded tokens look like the following\.

```
${TOKEN[Bucket.Name.1234]}
```

They can be passed around like regular strings, and can be concatenated, as shown in the following example\.

------
#### [ TypeScript ]

```
const functionName = bucket.bucketName + 'Function';
```

------
#### [ JavaScript ]

```
const functionName = bucket.bucketName + 'Function';
```

------
#### [ Python ]

```
function_name = bucket.bucket_name + "Function"
```

------
#### [ Java ]

```
String functionName = bucket.getBucketName().concat("Function");
```

------
#### [ C\# ]

```
string functionName = bucket.BucketName + "Function";
```

------
#### [ Go ]

```
functionName := *bucket.BucketName() + "Function"
```

------

You can also use string interpolation, if your language supports it, as shown in the following example\.

------
#### [ TypeScript ]

```
const functionName = `${bucket.bucketName}Function`;
```

------
#### [ JavaScript ]

```
const functionName = `${bucket.bucketName}Function`;
```

------
#### [ Python ]

```
function_name = f"{bucket.bucket_name}Function"
```

------
#### [ Java ]

```
String functionName = String.format("%sFunction". bucket.getBucketName());
```

------
#### [ C\# ]

```
string functionName = $"${bucket.bucketName}Function";
```

------
#### [ Go ]

Use `fmt.Sprintf` for similar functionality:

```
functionName := fmt.Sprintf("%sFunction", *bucket.BucketName())
```

------

Avoid manipulating the string in other ways\. For example, taking a substring of a string is likely to break the string token\.

## Working with list\-encoded tokens<a name="tokens_list"></a>

List\-encoded tokens look like the following:

```
["#{TOKEN[Stack.NotificationArns.1234]}"]
```

The only safe thing to do with these lists is pass them directly to other constructs\. Tokens in string list form cannot be concatenated, nor can an element be taken from the token\. The only safe way to manipulate them is by using AWS CloudFormation intrinsic functions like [Fn\.select](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-select.html)\.

## Working with number\-encoded tokens<a name="tokens_number"></a>

Number\-encoded tokens are a set of tiny negative floating\-point numbers that look like the following\.

```
-1.8881545897087626e+289
```

As with list tokens, you cannot modify the number value, as doing so is likely to break the number token\.

The following is an example of a construct that contains a token encoded as a number:

------
#### [ TypeScript ]

```
import { Stack, Duration, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as rds from 'aws-cdk-lib/aws-rds';
import * as ec2 from 'aws-cdk-lib/aws-ec2';

export class CdkDemoAppStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Define a new VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 3,  // Maximum number of availability zones to use
    });

    // Define an RDS database cluster
    const dbCluster = new rds.DatabaseCluster(this, 'MyRDSCluster', {
      engine: rds.DatabaseClusterEngine.AURORA,
      instanceProps: {
        vpc,
      },
    });

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // Print the value for our token at synthesis
    console.log("portToken: " + portToken);
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Duration } = require('aws-cdk-lib');
const lambda = require('aws-cdk-lib/aws-lambda');
const rds = require('aws-cdk-lib/aws-rds');
const ec2 = require('aws-cdk-lib/aws-ec2');

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define a new VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 3,  // Maximum number of availability zones to use
    });

    // Define an RDS database cluster
    const dbCluster = new rds.DatabaseCluster(this, 'MyRDSCluster', {
      engine: rds.DatabaseClusterEngine.AURORA,
      instanceProps: {
        vpc,
      },
    });

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // Print the value for our token at synthesis
    console.log("portToken: " + portToken);
  }
}

module.exports = { CdkDemoAppStack }
```

------
#### [ Python ]

```
from aws_cdk import (
    Duration,
    Stack,
)
from aws_cdk import aws_rds as rds
from aws_cdk import aws_ec2 as ec2
from constructs import Construct

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define a new VPC
        vpc = ec2.Vpc(self, 'MyVpc',
            max_azs=3  # Maximum number of availability zones to use
        )

        # Define an RDS database cluster
        db_cluster = rds.DatabaseCluster(self, 'MyRDSCluster',
            engine=rds.DatabaseClusterEngine.AURORA,
            instance_props=rds.InstanceProps(
                vpc=vpc
            )
        )

        # Get the port token (this is a token encoded as a number)
        port_token = db_cluster.cluster_endpoint.port

        # Print the value for our token at synthesis
        print(f"portToken: {port_token}")
```

------
#### [ Java ]

```
package com.myorg;

import software.constructs.Construct;
import software.amazon.awscdk.Stack;
import software.amazon.awscdk.StackProps;
import software.amazon.awscdk.services.ec2.Vpc;
import software.amazon.awscdk.services.rds.DatabaseCluster;
import software.amazon.awscdk.services.rds.DatabaseClusterEngine;
import software.amazon.awscdk.services.rds.InstanceProps;

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define a new VPC
        Vpc vpc = Vpc.Builder.create(this, "MyVpc")
            .maxAzs(3) // Maximum number of availability zones to use
            .build();

        // Define an RDS database cluster
        DatabaseCluster dbCluster = DatabaseCluster.Builder.create(this, "MyRDSCluster")
            .engine(DatabaseClusterEngine.AURORA)
            .instanceProps(InstanceProps.builder()
                .vpc(vpc)
                .build())
            .build();
        
        // Get the port token (this is a token encoded as a number)
        Number portToken = dbCluster.getClusterEndpoint().getPort();

        // Print the value for our token at synthesis
        System.out.println("portToken: " + portToken);
    }   
}
```

------
#### [ C\# ]

```
using Amazon.CDK;
using Constructs;
using Amazon.CDK.AWS.EC2;
using Amazon.CDK.AWS.RDS;
using System;
using System.Collections.Generic;

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define a new VPC
            var vpc = new Vpc(this, "MyVpc", new VpcProps
            {
                MaxAzs = 3  // Maximum number of availability zones to use
            });

            // Define an RDS database cluster
            var dbCluster = new DatabaseCluster(this, "MyRDSCluster", new DatabaseClusterProps
            {
                Engine = DatabaseClusterEngine.AURORA,  // Remove parentheses
                InstanceProps = new Amazon.CDK.AWS.RDS.InstanceProps // Specify RDS InstanceProps
                {
                    Vpc = vpc
                }
            });

            // Get the port token (this is a token encoded as a number)
            var portToken = dbCluster.ClusterEndpoint.Port;

            // Print the value for our token at synthesis
            System.Console.WriteLine($"portToken: {portToken}");
        }
    }
}
```

------
#### [ Go ]

```
package main

import (
	"fmt"

	"github.com/aws/aws-cdk-go/awscdk/v2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsec2"
	"github.com/aws/aws-cdk-go/awscdk/v2/awsrds"
	"github.com/aws/constructs-go/constructs/v10"
	"github.com/aws/jsii-runtime-go"
)

type CdkDemoAppStackProps struct {
	awscdk.StackProps
}

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define a new VPC
	vpc := awsec2.NewVpc(stack, jsii.String("MyVpc"), &awsec2.VpcProps{
		MaxAzs: jsii.Number(3), // Maximum number of availability zones to use
	})

	// Define an RDS database cluster
	dbCluster := awsrds.NewDatabaseCluster(stack, jsii.String("MyRDSCluster"), &awsrds.DatabaseClusterProps{
		Engine: awsrds.DatabaseClusterEngine_AURORA(),
		InstanceProps: &awsrds.InstanceProps{
			Vpc: vpc,
		},
	})

	// Get the port token (this is a token encoded as a number)
	portToken := dbCluster.ClusterEndpoint().Port()

	// Print the value for our token at synthesis
	fmt.Println("portToken: ", portToken)

	return stack
}

// ...
```

------

When we run `cdk synth`, the value for `portToken` is displayed as a number\-encoded token:

```
$ cdk synth --quiet
portToken: -1.8881545897087968e+289
```

### Pass number\-encoded tokens<a name="tokens-number-pass"></a>

When you pass number\-encoded tokens to other constructs, it may make sense to convert them to strings first\. For example, if you want to use the value of a number\-encoded string as part of a concatenated string, converting it helps with readability\.

In the following example, `portToken` is a number\-encoded token that we want to pass to our Lambda function as part of `connectionString`:

------
#### [ TypeScript ]

```
import { Stack, Duration, CfnOutput, StackProps } from 'aws-cdk-lib';
// ...
import * as lambda from 'aws-cdk-lib/aws-lambda';

export class CdkDemoAppStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Define a new VPC
    // ...

    // Define an RDS database cluster
    // ...

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // ...
    
    // Example connection string with the port token as a number
    const connectionString = `jdbc:mysql://mydb.cluster.amazonaws.com:${portToken}/mydatabase`;

    // Use the connection string as an environment variable in a Lambda function
    const myFunction = new lambda.Function(this, 'MyLambdaFunction', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
      environment: {
        DATABASE_CONNECTION_STRING: connectionString,  // Using the port token as part of the string
      },
    });

    // Output the value of our connection string at synthesis
    console.log("connectionString: " + connectionString);

    // Output the connection string
    new CfnOutput(this, 'ConnectionString', {
      value: connectionString,
    });
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Duration, CfnOutput } = require('aws-cdk-lib');
// ...
const lambda = require('aws-cdk-lib/aws-lambda');

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define a new VPC
    // ...

    // Define an RDS database cluster
    // ...

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // ...

    // Example connection string with the port token as a number
    const connectionString = `jdbc:mysql://mydb.cluster.amazonaws.com:${portToken}/mydatabase`;

    // Use the connection string as an environment variable in a Lambda function
    const myFunction = new lambda.Function(this, 'MyLambdaFunction', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromInline(`
        exports.handler = async function(event) {
          return {
            statusCode: 200,
            body: JSON.stringify('Hello World!'),
          };
        };
      `),
      environment: {
        DATABASE_CONNECTION_STRING: connectionString,  // Using the port token as part of the string
      },
    });

    // Output the value of our connection string at synthesis
    console.log("connectionString: " + connectionString);

    // Output the connection string
    new CfnOutput(this, 'ConnectionString', {
      value: connectionString,
    });
  }
}

module.exports = { CdkDemoAppStack }
```

------
#### [ Python ]

```
from aws_cdk import (
    Duration,
    Stack,
    CfnOutput,
)
from aws_cdk import aws_lambda as _lambda
# ...

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define a new VPC
        # ...

        # Define an RDS database cluster
        # ...

        # Get the port token (this is a token encoded as a number)
        port_token = db_cluster.cluster_endpoint.port

        # ...

        # Example connection string with the port token as a number
        connection_string = f"jdbc:mysql://mydb.cluster.amazonaws.com:{port_token}/mydatabase"

        # Use the connection string as an environment variable in a Lambda function
        my_function = _lambda.Function(self, 'MyLambdaFunction',
            runtime=_lambda.Runtime.NODEJS_20_X,
            handler='index.handler',
            code=_lambda.Code.from_inline("""
                exports.handler = async function(event) {
                    return {
                        statusCode: 200,
                        body: JSON.stringify('Hello World!'),
                    };
                };
            """),
            environment={
                'DATABASE_CONNECTION_STRING': connection_string  # Using the port token as part of the string
            }
        )

        # Output the value of our connection string at synthesis
        print(f"connectionString: {connection_string}")

        # Output the connection string
        CfnOutput(self, 'ConnectionString',
            value=connection_string
        )
```

------
#### [ Java ]



```
// ...
import software.amazon.awscdk.CfnOutput;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.lambda.Code;

import java.util.Map;

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define a new VPC
        // ...

        // Define an RDS database cluster
        // ...
        
        // Get the port token (this is a token encoded as a number)
        Number portToken = dbCluster.getClusterEndpoint().getPort();

        // ...

        // Example connection string with the port token as a number
        String connectionString = "jdbc:mysql://mydb.cluster.amazonaws.com:" + portToken + "/mydatabase";

        // Use the connection string as an environment variable in a Lambda function
        Function myFunction = Function.Builder.create(this, "MyLambdaFunction")
            .runtime(Runtime.NODEJS_20_X)
            .handler("index.handler")
            .code(Code.fromInline(
                "exports.handler = async function(event) {\n" +
                "  return {\n" +
                "    statusCode: 200,\n" +
                "    body: JSON.stringify('Hello World!'),\n" +
                "  };\n" +
                "};"))
            .environment(Map.of(
                "DATABASE_CONNECTION_STRING", connectionString // Using the port token as part of the string
            ))
            .build();

        // Output the value of our connection string at synthesis
        System.out.println("connectionString: " + connectionString);

        // Output the connection string
        CfnOutput.Builder.create(this, "ConnectionString")
            .value(connectionString)
            .build();
    }   
}
```

------
#### [ C\# ]

```
// ...
using Amazon.CDK.AWS.Lambda;

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define a new VPC
            // ...

            // Define an RDS database cluster
            var dbCluster = new DatabaseCluster(this, "MyRDSCluster", new DatabaseClusterProps
            // ...

            // Get the port token (this is a token encoded as a number)
            var portToken = dbCluster.ClusterEndpoint.Port;

            // ...

            // Example connection string with the port token as a number
            var connectionString = $"jdbc:mysql://mydb.cluster.amazonaws.com:{portToken}/mydatabase";

            // Use the connection string as an environment variable in a Lambda function
            var myFunction = new Function(this, "MyLambdaFunction", new FunctionProps
            {
                Runtime = Runtime.NODEJS_20_X,
                Handler = "index.handler",
                Code = Code.FromInline(@"
                    exports.handler = async function(event) {
                        return {
                            statusCode: 200,
                            body: JSON.stringify('Hello World!'),
                        };
                    };
                "),
                Environment = new Dictionary<string, string>
                {
                    { "DATABASE_CONNECTION_STRING", connectionString }  // Using the port token as part of the string
                }
            });

            // Output the value of our connection string at synthesis
            Console.WriteLine($"connectionString: {connectionString}");

            // Output the connection string
            new CfnOutput(this, "ConnectionString", new CfnOutputProps
            {
                Value = connectionString
            });
        }
    }
}
```

------
#### [ Go ]

```
// ...
	"github.com/aws/aws-cdk-go/awscdk/v2/awslambda"
)

type CdkDemoAppStackProps struct {
	awscdk.StackProps
}

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define a new VPC
	// ...

	// Define an RDS database cluster
	// ...

	// Get the port token (this is a token encoded as a number)
	portToken := dbCluster.ClusterEndpoint().Port()

	// ...

	// Example connection string with the port token as a number
	 connectionString := fmt.Sprintf("jdbc:mysql://mydb.cluster.amazonaws.com:%s/mydatabase", portToken)

	// Use the connection string as an environment variable in a Lambda function
	myFunction := awslambda.NewFunction(stack, jsii.String("MyLambdaFunction"), &awslambda.FunctionProps{
		Runtime: awslambda.Runtime_NODEJS_20_X(),
		Handler: jsii.String("index.handler"),
		Code: awslambda.Code_FromInline(jsii.String(`
			exports.handler = async function(event) {
				return {
					statusCode: 200,
					body: JSON.stringify('Hello World!'),
				};
			};
		`)),
		Environment: &map[string]*string{
			"DATABASE_CONNECTION_STRING": jsii.String(connectionString), // Using the port token as part of the string
		},
	})

	// Output the value of our connection string at synthesis
	fmt.Println("connectionString: ", connectionString)

	// Output the connection string
	awscdk.NewCfnOutput(stack, jsii.String("ConnectionString"), &awscdk.CfnOutputProps{
		Value: jsii.String(connectionString),
	})

	return stack
}

// ...
```

------

If we pass this value to `connectionString`, the output value when we run `cdk synth` may be confusing due to the number\-encoded string:

```
$ cdk synth --quiet
connectionString: jdbc:mysql://mydb.cluster.amazonaws.com:-1.888154589708796e+289/mydatabase
```

To convert a number\-encoded token to a string, use `[cdk\.Tokenization\.stringifyNumber\(*token*\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Tokenization.html#static-stringifywbrnumberx)`\. In the following example, we convert the number\-encoded token to a string before defining our connection string:

------
#### [ TypeScript ]

```
import { Stack, Duration, Tokenization, CfnOutput, StackProps } from 'aws-cdk-lib';
// ...

export class CdkDemoAppStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // Define a new VPC
    // ...

    // Define an RDS database cluster
    // ...

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // ...

    // Convert the encoded number to an encoded string for use in the connection string
    const portAsString = Tokenization.stringifyNumber(portToken);

    // Example connection string with the port token as a string
    const connectionString = `jdbc:mysql://mydb.cluster.amazonaws.com:${portAsString}/mydatabase`;

    // Use the connection string as an environment variable in a Lambda function
    const myFunction = new lambda.Function(this, 'MyLambdaFunction', {
      // ...
      environment: {
        DATABASE_CONNECTION_STRING: connectionString,  // Using the port token as part of the string
      },
    });

    // Output the value of our connection string at synthesis
    console.log("connectionString: " + connectionString);

    // Output the connection string
    new CfnOutput(this, 'ConnectionString', {
      value: connectionString,
    });
  }
}
```

------
#### [ JavaScript ]

```
const { Stack, Duration, Tokenization, CfnOutput } = require('aws-cdk-lib');
// ...

class CdkDemoAppStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define a new VPC
    // ...

    // Define an RDS database cluster
    // ...

    // Get the port token (this is a token encoded as a number)
    const portToken = dbCluster.clusterEndpoint.port;

    // ...

    // Convert the encoded number to an encoded string for use in the connection string
    const portAsString = Tokenization.stringifyNumber(portToken);

    // Example connection string with the port token as a string
    const connectionString = `jdbc:mysql://mydb.cluster.amazonaws.com:${portAsString}/mydatabase`;

    // Use the connection string as an environment variable in a Lambda function
    const myFunction = new lambda.Function(this, 'MyLambdaFunction', {
      // ...
      environment: {
        DATABASE_CONNECTION_STRING: connectionString,  // Using the port token as part of the string
      },
    });

    // Output the value of our connection string at synthesis
    console.log("connectionString: " + connectionString);

    // Output the connection string
    new CfnOutput(this, 'ConnectionString', {
      value: connectionString,
    });
  }
}

module.exports = { CdkDemoAppStack }
```

------
#### [ Python ]

```
from aws_cdk import (
    Duration,
    Stack,
    Tokenization,
    CfnOutput,
)
# ...

class CdkDemoAppStack(Stack):

    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        # Define a new VPC
        # ...

        # Define an RDS database cluster
        # ...

        # Get the port token (this is a token encoded as a number)
        port_token = db_cluster.cluster_endpoint.port

        # Convert the encoded number to an encoded string for use in the connection string
        port_as_string = Tokenization.stringify_number(port_token)

        # Example connection string with the port token as a string
        connection_string = f"jdbc:mysql://mydb.cluster.amazonaws.com:{port_as_string}/mydatabase"

        # Use the connection string as an environment variable in a Lambda function
        my_function = _lambda.Function(self, 'MyLambdaFunction',
            # ...
            environment={
                'DATABASE_CONNECTION_STRING': connection_string  # Using the port token as part of the string
            }
        )

        # Output the value of our connection string at synthesis
        print(f"connectionString: {connection_string}")

        # Output the connection string
        CfnOutput(self, 'ConnectionString',
            value=connection_string
        )
```

------
#### [ Java ]

```
// ...
import software.amazon.awscdk.Tokenization;

public class CdkDemoAppStack extends Stack {
    public CdkDemoAppStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public CdkDemoAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Define a new VPC
        // ...

        // Define an RDS database cluster
        // ...
        
        // Get the port token (this is a token encoded as a number)
        Number portToken = dbCluster.getClusterEndpoint().getPort();

        // ...
        
        // Convert the encoded number to an encoded string for use in the connection string
        String portAsString = Tokenization.stringifyNumber(portToken);

        // Example connection string with the port token as a string
        String connectionString = "jdbc:mysql://mydb.cluster.amazonaws.com:" + portAsString + "/mydatabase";

        // Use the connection string as an environment variable in a Lambda function
        Function myFunction = Function.Builder.create(this, "MyLambdaFunction")
            // ...
            .environment(Map.of(
                "DATABASE_CONNECTION_STRING", connectionString // Using the port token as part of the string
            ))
            .build();

        // Output the value of our connection string at synthesis
        System.out.println("connectionString: " + connectionString);

        // Output the connection string
        CfnOutput.Builder.create(this, "ConnectionString")
            .value(connectionString)
            .build();
    }   
}
```

------
#### [ C\# ]

```
// ...

namespace CdkDemoApp
{
    public class CdkDemoAppStack : Stack
    {
        internal CdkDemoAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Define a new VPC
            // ...

            // Define an RDS database cluster
            // ...

            // Get the port token (this is a token encoded as a number)
            var portToken = dbCluster.ClusterEndpoint.Port;

            // ...

            // Convert the encoded number to an encoded string for use in the connection string
            var portAsString = Tokenization.StringifyNumber(portToken);

            // Example connection string with the port token as a string
            var connectionString = $"jdbc:mysql://mydb.cluster.amazonaws.com:{portAsString}/mydatabase";

            // Use the connection string as an environment variable in a Lambda function
            var myFunction = new Function(this, "MyLambdaFunction", new FunctionProps
            {
                // ...
                Environment = new Dictionary<string, string>
                {
                    { "DATABASE_CONNECTION_STRING", connectionString }  // Using the port token as part of the string
                }
            });

            // Output the value of our connection string at synthesis
            Console.WriteLine($"connectionString: {connectionString}");

            // Output the connection string
            new CfnOutput(this, "ConnectionString", new CfnOutputProps
            {
                Value = connectionString
            });
        }
    }
}
```

------
#### [ Go ]

```
// ...

func NewCdkDemoAppStack(scope constructs.Construct, id string, props *CdkDemoAppStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	// Define a new VPC
	// ...
	
	// Define an RDS database cluster
	// ...

	// Get the port token (this is a token encoded as a number)
	portToken := dbCluster.ClusterEndpoint().Port()

	// ...

	// Convert the encoded number to an encoded string for use in the connection string
	portAsString := awscdk.Tokenization_StringifyNumber(portToken)

	// Example connection string with the port token as a string
	connectionString := fmt.Sprintf("jdbc:mysql://mydb.cluster.amazonaws.com:%s/mydatabase", portAsString)

	// Use the connection string as an environment variable in a Lambda function
	myFunction := awslambda.NewFunction(stack, jsii.String("MyLambdaFunction"), &awslambda.FunctionProps{
		// ...
		Environment: &map[string]*string{
			"DATABASE_CONNECTION_STRING": jsii.String(connectionString), // Using the port token as part of the string
		},
	})

	// Output the value of our connection string at synthesis
	fmt.Println("connectionString: ", connectionString)

	// Output the connection string
	awscdk.NewCfnOutput(stack, jsii.String("ConnectionString"), &awscdk.CfnOutputProps{
		Value: jsii.String(connectionString),
	})

	fmt.Println(myFunction)

	return stack
}

// ...
```

------

When we run `cdk synth`, the value for our connection string is represented in a cleaner and clearer format:

```
$ cdk synth --quiet
connectionString: jdbc:mysql://mydb.cluster.amazonaws.com:${Token[TOKEN.242]}/mydatabase
```

## Lazy values<a name="tokens_lazy"></a>

In addition to representing deploy\-time values, such as AWS CloudFormation [parameters](parameters.md), tokens are also commonly used to represent synthesis\-time lazy values\. These are values for which the final value will be determined before synthesis has completed, but not at the point where the value is constructed\. Use tokens to pass a literal string or number value to another construct, while the actual value at synthesis time might depend on some calculation that has yet to occur\.

You can construct tokens representing synth\-time lazy values using static methods on the `Lazy` class, such as `[Lazy\.string](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Lazy.html#static-stringproducer-options)` and `[Lazy\.number](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Lazy.html#static-numberproducer)`\. These methods accept an object whose `produce` property is a function that accepts a context argument and returns the final value when called\.

The following example creates an Auto Scaling group whose capacity is determined after its creation\.

------
#### [ TypeScript ]

```
let actualValue: number;

new AutoScalingGroup(this, 'Group', {
  desiredCapacity: Lazy.numberValue({
    produce(context) {
      return actualValue;
    }
  })
});

// At some later point
actualValue = 10;
```

------
#### [ JavaScript ]

```
let actualValue;

new AutoScalingGroup(this, 'Group', {
  desiredCapacity: Lazy.numberValue({
    produce(context) {
      return (actualValue);
    }
  })
});

// At some later point
actualValue = 10;
```

------
#### [ Python ]

```
class Producer:
    def __init__(self, func):
        self.produce = func

actual_value = None
          
AutoScalingGroup(self, "Group", 
    desired_capacity=Lazy.number_value(Producer(lambda context: actual_value))
)
    
# At some later point
actual_value = 10
```

------
#### [ Java ]

```
double actualValue = 0;

class ProduceActualValue implements INumberProducer {

    @Override
    public Number produce(IResolveContext context) {
        return actualValue;
    }
}

AutoScalingGroup.Builder.create(this, "Group")
    .desiredCapacity(Lazy.numberValue(new ProduceActualValue())).build();

// At some later point
actualValue = 10;
```

------
#### [ C\# ]

```
public class NumberProducer : INumberProducer
{
    Func<Double> function;
        
    public NumberProducer(Func<Double> function)
    {
        this.function = function;
    }

    public Double Produce(IResolveContext context)
    {
        return function();
    }
}

double actualValue = 0;

new AutoScalingGroup(this, "Group", new AutoScalingGroupProps
{
    DesiredCapacity = Lazy.NumberValue(new NumberProducer(() => actualValue))
});

// At some later point
actualValue = 10;
```

------

## Converting to JSON<a name="tokens_json"></a>

Sometimes you want to produce a JSON string of arbitrary data, and you may not know whether the data contains tokens\. To properly JSON\-encode any data structure, regardless of whether it contains tokens, use the method `[stack\.toJsonString](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#towbrjsonwbrstringobj-space)`, as shown in the following example\.

------
#### [ TypeScript ]

```
const stack = Stack.of(this);
const str = stack.toJsonString({
  value: bucket.bucketName
});
```

------
#### [ JavaScript ]

```
const stack = Stack.of(this);
const str = stack.toJsonString({
  value: bucket.bucketName
});
```

------
#### [ Python ]

```
stack = Stack.of(self)
string = stack.to_json_string(dict(value=bucket.bucket_name))
```

------
#### [ Java ]

```
Stack stack = Stack.of(this);
String stringVal = stack.toJsonString(java.util.Map.of(    // Map.of requires Java 9+
        put("value", bucket.getBucketName())));
```

------
#### [ C\# ]

```
var stack = Stack.Of(this);
var stringVal = stack.ToJsonString(new Dictionary<string, string>
{
    ["value"] = bucket.BucketName
});
```

------