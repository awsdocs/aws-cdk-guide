# Troubleshoot AWS CDK bootstrapping issues<a name="bootstrapping-troubleshoot"></a>

Troubleshoot common issues when bootstrapping your environment with the AWS Cloud Development Kit \(AWS CDK\)\.

For an introduction to bootstrapping, see [AWS CDK bootstrapping](bootstrapping.md)\.

For instructions on bootstrapping, see [Bootstrap your environment for use with the AWS CDK](bootstrapping-env.md)\.

## When bootstrapping using the default template, you get a 'CREATE\_FAILED' error for the Amazon S3 bucket<a name="bootstrapping-troubleshoot-s3-bucket-name"></a>

When bootstrapping using the AWS CDK Command Line Interface \(CDK CLI\) `cdk bootstrap` command with the default bootstrap template, you receive the following error:

```
CREATE_FAILED | AWS::S3::Bucket | BucketName already exists
```

Before troubleshooting, ensure that you are using the latest version of the CDK CLI\.
+ To check your version, run `cdk --version`\.
+ To install the latest version, run `npm install -g aws-cdk`\.

After installing the latest version, try bootstrapping your environment again\. If you still receive the same error, continue with troubleshooting\.

### Common causes<a name="bootstrapping-troubleshoot-s3-bucket-name-causes"></a>

When you bootstrap your environment, the AWS CDK generates physical IDs for your bootstrap resources\. For more information, see [Resource IDs created during bootstrapping](bootstrapping-env.md#bootstrapping-env-default-id)\.

Unlike the other bootstrap resources, Amazon S3 bucket names are global\. This means that each bucket name must be unique across all AWS accounts in all AWS Regions within a partition\. For more information, see [Buckets overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html) in the *Amazon S3 User Guide*\. Therefore, the most common cause of this error is that the physical ID generated as your bucket name already exists somewhere within the partition\. This could be within your account or another account\.

The following is an example bucket name: `cdk-hnb659fds-assets-012345678910-us-west-1`\. While unlikely, due to the qualifier and account ID being a part of the name, it is possible that this name for an Amazon S3 bucket is used by another AWS account\. Since bucket names are globally scoped, it can’t be used by you if its used by a different account in the same partition\. Most likely, a bucket with the same name exists somewhere in your account\. This could be in the Region you are attempting to bootstrap, or another Region\.

Generally, the resolution is to locate this bucket in your account and determine what to do with it, or customize bootstrapping to create bootstrap resources of a different name\.

### Resolution<a name="bootstrapping-troubleshoot-s3-bucket-name-resolution"></a>

First, determine if a bucket with the same name exists within your AWS account\. Using an AWS identity with valid permissions to lookup Amazon S3 buckets in your account, you can do this in the following ways:
+ Use the AWS Command Line Interface \(AWS CLI\) `aws s3 ls` command to view a list of all your buckets\.
+ Look up bucket names for each Region in your account using the [Amazon S3 console](https://console.aws.amazon.com/s3)\.

If a bucket with the same name exists, determine if it's being used\. If it's not being used, consider deleting the bucket and attempting to bootstrap your environment again\.

If a bucket with the same name exists and you don't want to delete it, determine if it’s already associated with a bootstrap stack in your account\. You may have to check multiple Regions\. The Region in the Amazon S3 bucket name doesn’t necessarily mean that the bucket is in that Region\. To check if it’s associated with the `CDKToolkit` bootstrap stack, you can do either of the following for each Region:
+ Use the AWS CLI `aws cloudformation describe-stack-resources --stack-name CDKToolkit --region Region` command to view the resources in your bootstrap stack and check if the bucket is listed\.
+ In the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation), locate the `CDKToolkit` stack\. Then, on the **Resources** tab, check if the bucket exists\.

If the bucket is associated with a bootstrap stack, determine if the bootstrap stack is in the same Region that you are attempting to bootstrap\. If it is, your environment is already bootstrapped and you should be able to start using the CDK to deploy applications into your environment\. If the Amazon S3 bucket is associated with a bootstrap stack in a different Region, you’ll need to determine what to do\. Possible resolutions include renaming the existing Amazon S3 bucket, deleting the current Amazon S3 bucket if its not being used, or using a new name for the Amazon S3 bucket you are attempting to create\.

If you are unable to locate an Amazon S3 bucket with the same name in your account, it may exist in a different account\. To resolve this, you’ll need to customize bootstrapping to create new names for all of your bootstrap resources or for just your Amazon S3 bucket\. To create new names for all bootstrap resources, you can modify the qualifier\. To create a new name for only your Amazon S3 bucket, you can provide a new bucket name\.

To customize bootstrapping, you can use options with the CDK CLI `cdk bootstrap` command or by modifying the bootstrap template\. For instructions, see [Customize AWS CDK bootstrapping](bootstrapping-customizing.md)\.

If you customize bootstrapping, you will need to apply the same changes to synthesis before you can properly deploy an application\. For instructions, see [Customize CDK stack synthesis](configure-synth.md#bootstrapping-custom-synth)\.

For example, you can provide a new qualifier with `cdk bootstrap`:

```
$ cdk bootstrap --qualifier abcde0123
```

The following is an example Amazon S3 bucket name that will be created with this modification: `cdk-abcde0123-assets-012345678910-us-west-1`\. All bootstrap resources created during bootstrapping will use this qualifier\.

When developing your CDK app, you must specify your custom qualifier in your synthesizer\. This helps the CDK with identifying your bootstrap resources during synthesis and deployment\. The following is an example of customizing the default synthesizer for your stack instance:

------
#### [ TypeScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    qualifier: 'abcde0123',
  }),
});
```

------
#### [ JavaScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    qualifier: 'abcde0123',
  }),
})
```

------
#### [ Python ]

```
MyStack(self, "MyStack",
    synthesizer=DefaultStackSynthesizer(
        qualifier="abcde0123"
))
```

------
#### [ Java ]

```
new MyStack(app, "MyStack", StackProps.builder()
  .synthesizer(DefaultStackSynthesizer.Builder.create()
    .qualifier("abcde0123")
    .build())
  .build();
```

------
#### [ C\# ]



```
new MyStack(app, "MyStack", new StackProps
{
    Synthesizer = new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
    {
        Qualifier = "abcde0123"
    })
});
```

------
#### [ Go ]

```
func NewMyStack(scope constructs.Construct, id string, props *MyStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	synth := awscdk.NewDefaultStackSynthesizer(&awscdk.DefaultStackSynthesizerProps{
		Qualifier: jsii.String("abcde0123"),
	})

	stack.SetSynthesizer(synth)

	return stack
}
```

------

You can also specify the new qualifier in the `cdk.json` file of your CDK project:

```
{
  "app": "...",
  "context": {
    "@aws-cdk/core:bootstrapQualifier": "abcde0123"
  }
}
```

To modify only the Amazon S3 bucket name, you can use the `\-\-bootstrap\-bucket\-name` option\. The following is an example:

```
$ cdk bootstrap --bootstrap-bucket-name 'my-new-bucket-name'
```

When developing your CDK app, you must specify your new bucket name in your synthesizer\. The following is an example of customizing the default synthesizer for your stack instance:

------
#### [ TypeScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    fileAssetsBucketName: 'my-new-bucket-name',
  }),
});
```

------
#### [ JavaScript ]

```
new MyStack(this, 'MyStack', {
  synthesizer: new DefaultStackSynthesizer({
    fileAssetsBucketName: 'my-new-bucket-name',
  }),
})
```

------
#### [ Python ]

```
MyStack(self, "MyStack",
    synthesizer=DefaultStackSynthesizer(
        file_assets_bucket_name='my-new-bucket-name'
))
```

------
#### [ Java ]

```
new MyStack(app, "MyStack", StackProps.builder()
  .synthesizer(DefaultStackSynthesizer.Builder.create()
    .fileAssetsBucketName("my-new-bucket-name")
    .build())
  .build();
```

------
#### [ C\# ]



```
new MyStack(app, "MyStack", new StackProps
{
    Synthesizer = new DefaultStackSynthesizer(new DefaultStackSynthesizerProps
    {
        FileAssetsBucketName = "my-new-bucket-name"
    })
});
```

------
#### [ Go ]

```
func NewMyStack(scope constructs.Construct, id string, props *MyStackProps) awscdk.Stack {
	var sprops awscdk.StackProps
	if props != nil {
		sprops = props.StackProps
	}
	stack := awscdk.NewStack(scope, &id, &sprops)

	synth := awscdk.NewDefaultStackSynthesizer(&awscdk.DefaultStackSynthesizerProps{
		FileAssetsBucketName: jsii.String("my-new-bucket-name"),
	})

	stack.SetSynthesizer(synth)

	return stack
}
```

------

### Prevention<a name="bootstrapping-troubleshoot-s3-bucket-name-prevention"></a>

**Proactively bootstrap your environments**  <a name="bootstrapping-troubleshoot-s3-bucket-name-prevention-proactive"></a>
If you don’t have reasons to customize bootstrapping or synthesis, we recommend that you use the default bootstrapping process and synthesizer, which work together automatically\. To do this, the default Amazon S3 bucket name that will be created during bootstrapping must not yet exist\. To prevent against this from happening, we recommend that you proactively bootstrap each Region for each AWS account that you plan to use\. You can do this before you ever plan on actually deploying CDK applications into the environment\. Specifically for the Amazon S3 bucket naming issue, this will create Amazon S3 buckets in each AWS environment and prevent others from using your Amazon S3 bucket name\.

**Protect your account ID**  <a name="bootstrapping-troubleshoot-s3-bucket-name-prevention-protect"></a>
Limit the sharing of your account ID\. This will help to prevent bad actors from using your account ID to generate their own Amazon S3 bucket names\.