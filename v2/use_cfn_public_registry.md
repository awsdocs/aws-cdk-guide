# Use resources from the AWS CloudFormation Public Registry<a name="use_cfn_public_registry"></a>

The AWS CloudFormation Public Registry lets you manage extensions, both public and private, such as resources, modules, and hooks that are available for use in your AWS account\. You can use public resource extensions in your AWS Cloud Development Kit \(AWS CDK\) applications with the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html) construct\. 

To learn more about the AWS CloudFormation Public Registry, see [Using the AWS CloudFormation registry](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/registry.html) in the *AWS CloudFormation User Guide*\.

All public extensions published by AWS are available to all accounts in all Regions without any action on your part\. However, you must activate each third\-party extension you want to use, in each account and Region where you want to use it\. 

**Note**  
When you use AWS CloudFormation with third\-party resource types, you will incur charges\. Charges are based on the number of handler operations you run per month and handler operation duration\. See [CloudFormation pricing](https://aws.amazon.com/cloudformation/pricing/) for complete details\.

To learn more about public extensions, see [Using public extensions in CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/registry-public.html) in the *AWS CloudFormation User Guide*

**Topics**
+ [Activate a third\-party resource in your account and Region](#use_cfn_public_registry_activate)
+ [Add a resource from the AWS CloudFormation Public Registry to your CDK app](#use_cfn_public_registry_add)

## Activate a third\-party resource in your account and Region<a name="use_cfn_public_registry_activate"></a>

Extensions published by AWS do not require activation\. They are always available in every account and Region\. You can activate a third\-party extension through the AWS Management Console, via the AWS Command Line Interface, or by deploying a special AWS CloudFormation resource\.

**To activate a third\-party extension through the AWS Management Console or see what resources are available**  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/v2/guide/images/activate-cfn-extension.png)

1. Sign in to the AWS account in which you want to use the extension, then switch to the Region where you want to use it\.

1. Navigate to the CloudFormation console via the **Services** menu\.

1. Choose **Public extensions** on the navigation bar, then activate the **Third party** radio button under **Publisher**\. A list of the available third\-party public extensions appears\. \(You may also choose **AWS** to see a list of the public extensions published by AWS, though you don't need to activate them\.\)

1. Browse the list and find the extension you want to activate\. Alternatively, search for it, then activate the radio button in the upper right corner of the extension's card\.

1. Choose the **Activate** button at the top of the list to activate the selected extension\. The extension's **Activate** page appears\.

1. In the **Activate** page, you can override the extension's default name and specify an execution role and logging configuration\. You can also choose whether to automatically update the extension when a new version is released\. When you have set these options as you like, choose **Activate extension** at the bottom of the page\.

**To activate a third\-party extension using the AWS CLI**
+ Use the `activate-type` command\. Substitute the ARN of the custom type you want to use where indicated\.

  The following is an example:

  ```
  aws cloudformation activate-type --public-type-arn public_extension_ARN --auto-update-activated
  ```

**To activate a third\-party extension through CloudFormation or CDK**
+ Deploy a resource of type `AWS::CloudFormation::TypeActivation` and specify the following properties:

  1. `TypeName` \- The name of the type, such as `AWSQS::EKS::Cluster`\.

  1. `MajorVersion` \- The major version number of the extension that you want\. Omit if you want the latest version\.

  1. `AutoUpdate` \- Whether to automatically update this extension when a new minor version is released by the publisher\. \(Major version updates require explicitly changing the `MajorVersion` property\.\)

  1. `ExecutionRoleArn` \- The ARN of the IAM role under which this extension will run\.

  1. `LoggingConfig` \- The logging configuration for the extension\.

  The `TypeActivation` resource can be deployed by the CDK using the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html) construct\. This is shown for the actual extensions in the following section\.

## Add a resource from the AWS CloudFormation Public Registry to your CDK app<a name="use_cfn_public_registry_add"></a>

 Use the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnResource.html) construct to include a resource from the AWS CloudFormation Public Registry in your application\. This construct is in the CDK's `aws-cdk-lib` module\. 

For example, suppose that there is a public resource named `MY::S5::UltimateBucket` that you want to use in your AWS CDK application\. This resource takes one property: the bucket name\. The corresponding `CfnResource` instantiation looks like this\.

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
	.properties(java.util.Map.of(    // Map.of requires Java 9+
	    "BucketName", "UltimateBucket"))
	.build();
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