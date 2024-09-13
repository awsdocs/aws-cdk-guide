# Supported programming languages for the AWS CDK<a name="languages"></a>

The AWS Cloud Development Kit \(AWS CDK\) has first\-class support for the following general\-purpose programming languages:
+ TypeScript
+ JavaScript
+ Python
+ Java
+ C\#
+ Go

Other JVM and \.NET CLR languages may also be used in theory, but we do not offer official support at this time\.

The AWS CDK is developed in one language, TypeScript\. To support the other languages, the AWS CDK utilizes a tool called [https://github.com/aws/jsii](https://github.com/aws/jsii) to generate language bindings\.

We attempt to offer each language's usual conventions to make development with the AWS CDK as natural and intuitive as possible\. For example, we distribute AWS Construct Library modules using your preferred language's standard repository, and you install them using the language's standard package manager\. Methods and properties are also named using your language's recommended naming patterns\.

The following are a few code examples:

------
#### [ TypeScript ]

```
const bucket = new s3.Bucket(this, 'amzn-s3-demo-bucket', {
  bucketName: 'amzn-s3-demo-bucket',
  versioned: true,
  websiteRedirect: {hostName: 'aws.amazon.com'}});
```

------
#### [ JavaScript ]

```
const bucket = new s3.Bucket(this, 'amzn-s3-demo-bucket', {
  bucketName: 'amzn-s3-demo-bucket',
  versioned: true,
  websiteRedirect: {hostName: 'aws.amazon.com'}});
```

------
#### [ Python ]

```
bucket = s3.Bucket("amzn-s3-demo-bucket", bucket_name="amzn-s3-demo-bucket", versioned=True,
            website_redirect=s3.RedirectTarget(host_name="aws.amazon.com"))
```

------
#### [ Java ]

```
Bucket bucket = Bucket.Builder.create(self, "amzn-s3-demo-bucket")
                      .bucketName("amzn-s3-demo-bucket")
                      .versioned(true)
                      .websiteRedirect(new RedirectTarget.Builder()
                          .hostName("aws.amazon.com").build())
                      .build();
```

------
#### [ C\# ]

```
var bucket = new Bucket(this, "amzn-s3-demo-bucket", new BucketProps {
                      BucketName = "amzn-s3-demo-bucket",
                      Versioned  = true,
                      WebsiteRedirect = new RedirectTarget {
                              HostName = "aws.amazon.com"
                      }});
```

------
#### [ Go ]

```
bucket := awss3.NewBucket(scope, jsii.String("amzn-s3-demo-bucket"), &awss3.BucketProps {
	BucketName: jsii.String("amzn-s3-demo-bucket"),
	Versioned: jsii.Bool(true),
	WebsiteRedirect: &awss3.RedirectTarget {
		HostName: jsii.String("aws.amazon.com"),
	},
})
```

------

**Note**  
These code snippets are intended for illustration only\. They are incomplete and won't run as they are\.

The AWS Construct Library is distributed using each language's standard package management tools, including NPM, PyPi, Maven, and NuGet\. We also provide a version of the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html) for each language\.

To help you use the AWS CDK in your preferred language, this guide includes the following topics for supported languages:
+ [Working with the AWS CDK in TypeScript](work-with-cdk-typescript.md)
+ [Working with the AWS CDK in JavaScript](work-with-cdk-javascript.md)
+ [Working with the AWS CDK in Python](work-with-cdk-python.md)
+ [Working with the AWS CDK in Java](work-with-cdk-java.md)
+ [Working with the AWS CDK in C\#](work-with-cdk-csharp.md)
+ [Working with the AWS CDK in Go](work-with-cdk-go.md)

TypeScript was the first language supported by the AWS CDK, and much of the AWS CDK example code is written in TypeScript\. This guide includes a topic specifically to show how to adapt TypeScript AWS CDK code for use with the other supported languages\. For more information, see [Comparing AWS CDK in TypeScript with other languages](work-with.md#work-with-cdk-compare)\.