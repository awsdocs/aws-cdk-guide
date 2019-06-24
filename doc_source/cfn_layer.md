--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Work around Missing AWS CDK Features<a name="cfn_layer"></a>

This topic describes how to modify the underlying AWS CloudFormation resources in the AWS Construct Library\. We also call this technique an "escape hatch" because it allows users to "escape" from the abstraction boundary defined by the AWS construct, and patch the underlying resources\.

**Important**  
We don't recommend this method because it breaks the abstraction layer and might produce unexpected results\.  
If you modify an AWS construct in this way, we can't ensure that your code will be compatible with later releases\.

AWS constructs, such as [Topic](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-sns/topic.html), encapsulate one or more AWS CloudFormation resources behind their APIs\. These resources are also represented as `CfnXxx` constructs in each library\. For example, the [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) construct encapsulates the [CfnBucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/cfnbucket.html)\. When a stack that includes an AWS construct is synthesized, the AWS CloudFormation definitions of the underlying resources are included in the resulting template\.

In later versions, we expect the APIs provided by AWS constructs to support all of the AWS services and their capabilities\. We know that there are still gaps in the AWS Construct Library, both at the service level \(some services currently don't have any constructs\), and at the resource level \(existing AWS constructs might be missing some features\)\.

**Note**  
If you encounter a missing capability in the AWS Construct Library, whether it's an entire library, a specific resource, or a feature, create an [issue](https://github.com/awslabs/aws-cdk/issues/new) on GitHub and let us know\.

This section describes the following use cases:
+ How to access the low\-level AWS CloudFormation resources encapsulated by an AWS construct
+ How to specify resource options, such as metadata, and dependencies on resources
+ How to add overrides to AWS CloudFormation resources and property definitions
+ How to directly define low\-level AWS CloudFormation resources without an AWS construct

For more information about how to work directly with the AWS CloudFormation layer, see the [AWS Construct Library](aws_construct_lib.md)\.

## Accessing Low\-Level Resources<a name="cfn_layer_low_level"></a>

Use [construct\.findChild\(\)](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.Construct.findChild) to access any child of a construct by its construct ID\. By convention, the main resource of any AWS construct is named **Resource**\.

The following example shows how to access the underlying Amazon Simple Storage Service \(Amazon S3\) bucket resource, given a [Bucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html) construct\.

```
// Create an AWS bucket construct
const bucket = new s3.Bucket(this, 'MyBucket');
      
// The main construct is named "Resource" and
// its type is s3.CfnBucket; const
const bucketResource = bucket.node.findChild('Resource') as s3.CfnBucket;
```

The `bucketResource` represents the low\-level AWS CloudFormation resource of type [CfnBucket](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/cfnbucket.html) that's encapsulated by the bucket\.

If the child can't be located, [construct\.findChildren\(id\)](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3#@aws-cdk/cdk.Construct.findChild) fails\. This means that if the underlying AWS Construct Library changes the IDs or structure for some reason, synthesis fails\.

You can also use [construct\.children](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3#@aws-cdk/cdk.Construct.children) for more advanced queries\. For example, you can look for a child that has a certain AWS CloudFormation resource type\.

```
const bucketResource =
    bucket.children.find(c => (c as cdk.Resource).resourceType === 'AWS::S3::Bucket')
      as s3.CfnBucket;
```

When you have an AWS CloudFormation resource, you are interacting with AWS CloudFormation resource classes, which extend [cdk\.CfnResource](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource)\.

## Setting Resource Options<a name="cfn_layer_resources"></a>

Set resource options using [cdk\.CfnResource](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource) properties, such as *Metadata* and *DependsOn*\.

For example, the following code:

```
const bucketResource = bucket.node.findChild('Resource') as s3.CfnBucket;
      
bucketResource.options.metadata = { MetadataKey: 'MetadataValue' };
bucketResource.options.updatePolicy = {
    autoScalingRollingUpdate: {
        pauseTime: '390'
    }
};
      
bucketResource.addDependency(otherBucket.node.findChild('Resource') as cdk.Resource);
```

Synthesizes into the following template\.

```
{
    "Type": "AWS::S3::Bucket",
    "DependsOn": [ "Other34654A52" ],
    "UpdatePolicy": {
      "AutoScalingRollingUpdate": {
        "PauseTime": "390"
      }
    },
    "Metadata": {
      "MetadataKey": "MetadataValue"
  }
}
```

## Raw Overrides<a name="cfn_layer_raw_overrides"></a>

 Use the [cdk\.CfnResource\.addOverride\(path, value\)](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource.addOverride) method to define an override that is applied to the resource definition during synthesis, as shown in the following example\.

```
// Define an override at the resource definition root, you can even modify the "Type"
// of the resource if needed.
bucketResource.addOverride('Type', 'AWS::S3::SpecialBucket');
      
// fine an override for a property (both are equivalent operations):
bucketResource.addPropertyOverride('VersioningConfiguration.Status', 'NewStatus');
bucketResource.addOverride('Properties.VersioningConfiguration.Status', 'NewStatus');
      
// se dot-notation to define overrides in complex structures which will be merged
// with the values set by the higher-level construct.
bucketResource.addPropertyOverride('LoggingConfiguration.DestinationBucketName', otherBucket.bucketName);
      
// It's also possible to assign a null value
bucketResource.addPropertyOverride('Foo.Bar', null);
```

This synthesizes to the following\.

```
{
    "Type": "AWS::S3::SpecialBucket",
    "Properties": {
        "Foo": {
            "Bar": null
        },
        "VersioningConfiguration": {
            "Status": "NewStatus"
        },
        "LoggingConfiguration": {
            "DestinationBucketName": {
              "Ref": "Other34654A52"
            }
        }
    }
}
```

To delete values, use `undefined`, [cdk\.CfnResource\.addDeletionOverride](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource.addDeletionOverride), or [cdk\.CfnResource\.addPropertyDeletionOverride](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource.addPropertyDeletionOverride)\.

```
const bucket = new s3.Bucket(this, 'MyBucket', {
    versioned: true,
    encryption: s3.BucketEncryption.KmsManaged
});
      
const bucketResource = bucket.node.findChild('Resource') as s3.CfnBucket;
bucketResource.addPropertyOverride('BucketEncryption.ServerSideEncryptionConfiguration.0.EncryptEverythingAndAlways', true);
bucketResource.addPropertyDeletionOverride('BucketEncryption.ServerSideEncryptionConfiguration.0.ServerSideEncryptionByDefault');
```

This synthesizes to the following\.

```
{
    "MyBucketF68F3FF0": {
        "Type": "AWS::S3::Bucket",
        "Properties": {
            "BucketEncryption": {
                "ServerSideEncryptionConfiguration": [
                    {
                        "EncryptEverythingAndAlways": true
                    }
                ]
            },
            "VersioningConfiguration": {
                "Status": "Enabled"
            }
        }
    }
}
```

## Directly Defining AWS CloudFormation Resources<a name="cfn_layer_direct_define"></a>

You can also explicitly define AWS CloudFormation resources in your stack\. 

To do this, instantiate one of the `CfnXxx` constructs of the dedicated library\.

```
new s3.CfnBucket(this, 'MyBucket', {
    analyticsConfigurations: [
      // ...
    ]
});
```

In the rare case where you want to define a resource that doesn't have a corresponding `CfnXxx` class \(such as a new resource that wasn't published yet in the AWS CloudFormation resource specification\), you can instantiate the [cdk\.CfnResource](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/cdk.html#@aws-cdk/cdk.CfnResource)\.

```
new cdk.Resource(this, 'MyBucket', {
    type: 'AWS::S3::Bucket',
    properties: {
        AnalyticsConfiguration: [ /* ... */ ] // note the PascalCase here
    }
});
```