# Using resources from the AWS CloudFormation Public Registry<a name="use_cfn_public_registry"></a>

 The AWS CloudFormation Public Registry is a collection of AWS CloudFormation extensions from both AWS and third parties that are available for use by all AWS customers\. You can also publish your own extension for others to use\. Extensions are of two types: resources and modules\. You can use public resource extensions in your AWS CDK app using the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html) construct\.

All public extensions published by AWS are available to all accounts in all regions without any action on your part\. On the other hand, you must activate each third\-party extension you want to use, in each account and region where you want to use it\. 

**Note**  
When you use AWS CloudFormation with third\-party resource types, you will incur charges based on the number of handler operations you run per month and handler operation duration\. See [CloudFormation pricing](http://aws.amazon.com/cloudformation/pricing/) for complete details\.

See [Using public extensions in CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/registry-public.html) for complete documentation of this feature from the AWS CloudFormation side\.

## Activating a third\-party resource in your account and region<a name="use_cfn_public_registry_activate"></a>

Extensions published by AWS do not require activation; they are always available in every account and region\. You can activate a third\-party extension through the AWS Management Console, via the AWS Command Line Interface, or by deploying a special AWS CloudFormation resource\.

To activate a third\-party extension through the AWS Management Console, or to simply see what resources are available, follow these steps\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/activate-cfn-extension.png)

1. Log in to the AWS account in which you want to use the extension, then switch to the region where you want to use it\.

1. Navigate to the CloudFormation console via the **Services** menu\.

1. Click **Public extensions** on the navigation bar, then activate the **Third party** radio button under **Publisher**\. A list of the available third\-party public extensions appears\. \(You may also choose **AWS** to see a list of the public extensions published by AWS, though you don't need to activate them\.\)

1. Browse the list and find the extension you want to activate, or search for it, then activate the radio button in the upper right corner of the extension's card\.

1. Click the **Activate** button at the top of the list to activate the selected extension\. The extension's **Activate** page appears\.

1. In the **Activate** page, you may override the extension's default name, specify an execution role and logging configuration, and choose whether to automatically update the extension when a new version is released\. When you have set these options as you like, click **Activate extension** at the bottom of the page\.

To activate a third\-party extension using the AWS CLI, use the `activate-type `command, substituting the ARN of the custom type you want to use for the one given\.

```
aws cloudformation activate-type --public-type-arn public_extension_ARN --auto-update-activated
```

To activate an extension through CloudFormation or the CDK, deploy a resource of type `AWS::CloudFormation::TypeActivation`, specifying the following properties\.
+ `TypeName` \- The name of the type, such as `AWSQS::EKS::Cluster`\.
+ `MajorVersion` \- The major version number of the extension that you want; omit if you want the latest version\.
+ `AutoUpdate` \- Whether to automatically update this extension when a new minor version is released by the publisher\. \(Major version updates require explicitly changing the `MajorVersion` property\.\)
+ `ExecutionRoleArn` \- The ARN of the IAM role under which this extension will run\.
+ `LoggingConfig` \- The logging configuration for the extension\.

The `TypeActivation` resource can be deployed by the CDK using the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html) construct, as shown below for the actual etxensions\.

## Adding a resource from the AWS CloudFormation Public Registry to your CDK app<a name="use_cfn_public_registry_add"></a>

 Use the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnResource.html) construct to include a resource from the AWS CloudFormation Public Registry in your application\. This construct is in the CDK's `core` module\. 

For example, suppose there is a public resource named `MY::S5::UltimateBucket` that you want to use in your AWS CDK application\. This resource takes one property: the bucket name\. The corresponding `CfnResource` instantiation looks like this\.

------
#### [ TypeScript ]

```
const ubucket = new CfnResource(this, 'MyUltimateBucket', {
    type: 'MY::S5::UltimateBucket::MODULE',
    properties: {
           BucketName: 'UltimateBucket'
    }
});
```

------
#### [ JavaScript ]

```
const ubucket = new CfnResource(this, 'MyUltimateBucket', {
    type: 'MY::S5::UltimateBucket::MODULE',
    properties: {
           BucketName: 'UltimateBucket'
    }
});
```

------
#### [ Python ]

```
ubucket = CfnResource(self, "MyUltimateBucket",
            type="MY::S5::UltimateBucket::MODULE",
            properties=dict(
                BucketName="UltimateBucket"))
```

------
#### [ Java ]

```
CfnResource.Builder.create(this, "MyUltimateBucket")
	.type("MY::S5::UltimateBucket::MODULE")
	.properties(new HashMap<String, String>() {{
	    put("BucketName", "UltimateBucket");
	}});
```

------
#### [ C\# ]

```
new CfnResource(this, "MyUltimateBucket", new CfnResourceProps
{
    Type = "MY::S5::UltimateBucket::MODULE",
    Properties = new Dictionary<string, object>
    {
        ["BucketName"] = "UltimateBucket"
    }
});
```

------