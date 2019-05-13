--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Multi\-Language Support in the CDK<a name="multiple_languages"></a>

This section describes the multi\-language support in the CDK, including hints for porting TypeScript to one of the supported languages\. See [Hello World Tutorial](getting_started.md#hello_world_tutorial) for an example of creating a CDK app in a supported language\.

The CDK supports C\#, Java, JavaScript, and TypeScript\. Since the CDK is developed in TypeScript, many code examples are still only available in TypeScript\. This section will help you port those TypeScript examples to one of the other programming languages\.

## Importing a Package<a name="multiple_languages_import"></a>

In TypeScript, you import a package as follows \(we'll use Amazon S3 for our examples\):

```
import s3 = require("@aws-cdk/aws-s3");
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.S3;
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.s3.*;
```

------
#### [ JavaScript ]

```
const s3 = require('@aws-cdk/aws-s3');
```

------

## Creating a New Object<a name="multiple_languages_new"></a>

In TypeScript, you create a new object as follows\. The first argument, `scope`, is always `this`, the second is the `id` of the construct, and the last is a list of properties, often optional\.

```
new s3.Bucket(this, 'MyFirstBucket', {
  // options
});
```

------
#### [ C\# ]

```
new Bucket(this, "MyFirstBucket", new BucketProps
{
  // options
});
```

------
#### [ Java ]

```
new Bucket(this, "MyFirstBucket", BucketProps.builder()
  // options
}
```

------
#### [ JavaScript ]

```
new s3.Bucket(this, 'MyFirstBucket', {
  // options
});
```

------