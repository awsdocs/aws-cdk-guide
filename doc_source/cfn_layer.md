# Escape Hatches<a name="cfn_layer"></a>

It's possible that neither the high\-level constructs nor the low\-level CFN Resource constructs have a specific feature you are looking for\. There are three possible reasons for this lack of functionality:
+ The AWS service feature is available through AWS CloudFormation, but there are no Construct classes for the service\.
+ The AWS service feature is available through AWS CloudFormation, and there are Construct classes for the service, but the Construct classes don’t yet expose the feature\.
+ The feature is not yet available through AWS CloudFormation\.

To determine whether a feature is available through AWS CloudFormation, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

## Using AWS CloudFormation Constructs Directly<a name="cfn_layer_cfn"></a>

If there are no Construct classes available for the service, you can fall back to the automatically generated CFN Resources, which map 1:1 onto all available AWS CloudFormation resources and properties\. These resources can be recognized by their name starting with `Cfn`, such as `CfnBucket` or `CfnRole`\. You instantiate them exactly as you would use the equivalent AWS CloudFormation resource\. For more information, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

For example, to instantiate a low\-level Amazon S3 bucket CFN Resource with analytics enabled, you would write something like the following\.

```
new s3.CfnBucket(this, 'MyBucket', {
  analyticsConfigurations: [
    { 
      id: 'Config',
      // ...        
    } 
  ]
});
```

In the rare case where you want to define a resource that doesn't have a corresponding `CfnXxx` class, such as a new resource that wasn't published yet in the AWS CloudFormation resource specification, you can instantiate the `cdk.CfnResource` directly and specify the resource type and properties\. This is shown in the following example\.

```
new cdk.CfnResource(this, 'MyBucket', {
  type: 'AWS::S3::Bucket',
  properties: {
    // Note the PascalCase here!
    AnalyticsConfigurations: [
      {
        Id: 'Config',
        // ...
      }
    ] 
  }
});
```

For more information, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

## Modifying the AWS CloudFormation Resource behind AWS Constructs<a name="cfn_layer_resource"></a>

If an Construct is missing a feature or you are trying to work around an issue, you can modify the CFN Resource that is encapsulated by the Construct\.

All Constructs contain within them the corresponding CFN Resource\. For example, the high\-level `Bucket` construct wraps the low\-level `CfnBucket` construct\. Because the `CfnBucket` corresponds directly to the AWS CloudFormation resource, it exposes all features that are available through AWS CloudFormation\.

The basic approach to get access to the CFN Resource class is to use `construct.node.defaultChild`, cast it to the right type for the resource, and modify its properties\. Again, let’s take the example of a `Bucket`\.

```
// Get the AWS CloudFormation resource
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;

// Change its properties
cfnBucket.analyticsConfiguration = [
  { 
    id: 'Config',
    // ...        
  } 
];
```

You can also use this object to change AWS CloudFormation options such as `Metadata` and `UpdatePolicy`\.

```
cfnBucket.cfnOptions.metadata = {
  MetadataKey: 'MetadataValue'
};
```

## Raw Overrides<a name="cfn_layer_raw"></a>

If there are properties that are missing from the CFN Resource, you can bypass all typing using raw overrides\. This also makes it possible to delete synthesized properties\. 

Use one of the `addOverride` or `addDeletionOverride` methods, as shown in the following example\.

```
// Get the AWS CloudFormation resource
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;

// Use dot notation to address inside the resource template fragment
cfnBucket.addOverride('Properties.VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addDeletionOverride('Properties.VersioningConfiguration.Status');

// addPropertyOverride is a convenience function, which implies the
// path starts with "Properties."
cfnBucket.addPropertyOverride('VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addPropertyDeletionOverride('VersioningConfiguration.Status');
```

## Custom Resources<a name="cfn_layer_custom"></a>

If the feature isn't available through AWS CloudFormation, but only through a direct API call, the only solution is to write an AWS CloudFormation Custom Resource to make the API call you need\. Don’t worry, the AWS CDK makes it easier to write these, and wrap them up into a regular construct interface, so from another user’s perspective the feature feels native\.

Building a custom resource involves writing a Lambda function that responds to a resource’s CREATE, UPDATE and DELETE lifecycle events\. If your custom resource needs to make only a single API call, consider using the [AwsCustomResource](https://github.com/awslabs/aws-cdk/tree/master/packages/%40aws-cdk/custom-resources)\. This makes it possible to perform arbitrary SDK calls during an AWS CloudFormation deployment\. Otherwise, you should write your own Lambda function to perform the work you need to get done\.

The subject is too broad to completely cover here, but the following links should get you started:
+ [Custom Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html)
+ [Custom\-Resource Example](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/custom-resource/)
+ For a more fully fledged example, see the [DnsValidatedCertificate](https://github.com/awslabs/aws-cdk/blob/master/packages/@aws-cdk/aws-certificatemanager/lib/dns-validated-certificate.ts) class in the CDK standard library\. This is implemented as a custom resource\.