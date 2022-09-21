# Abstractions and escape hatches<a name="cfn_layer"></a>

The AWS CDK lets you describe AWS resources using constructs that operate at varying levels of abstraction\.
+ *Layer 1 \(L1\)* constructs directly represent AWS CloudFormation resources as defined by the CloudFormation specification\. These constructs can be identified via a name beginning with "Cfn," so they are also referred to as "Cfn constructs\." If a resource exists in AWS CloudFormation, it exists in the CDK as a L1 construct\.
+ *Layer 2 \(L2\)* or "curated" constructs are thoughtfully developed to provide a more ergonomic developer experience compared to the L1 construct they're built upon\. In a typical CDK app, L2 constructs are usually the most widely used type\. Often, L2 constructs define additional supporting resources, such as IAM policies, Amazon SNS topics, or AWS KMS keys\. L2 constructs provide sensible defaults, best\-practice security policies, and a more ergonomic developer experience\.
+ *Layer 3 \(L3\)* constructs or *patterns* define entire collections of AWS resources for specific use cases, making it easy to stand up a build pipeline, an Amazon ECS application, or one of many other types of common deployment scenarios\. Because they can constitute complete system designs, or substantial parts of a larger system, L3 constructs are often "opinionated"—they are built around a very particular approach toward solving the problem at hand, and things work out best when you follow their lead\.

**Tip**  
For more details on AWS CDK constructs, see [Constructs](constructs.md)\.

At the highest level, your AWS CDK application and the stacks in it are themselves abstractions of your entire cloud infrastructure, or significant chunks of it, and may be parameterized to deploy them in different environments or for different needs\.

Abstractions are powerful tools for designing and implementing cloud applications\. The AWS CDK gives you the power not only to build with its abstractions, but also to create new abstractions\. Using the existing open\-source L2 and L3 constructs as guidance, you can build your own L2 and L3 constructs to reflect your own organization's best practices and opinions\.

No abstraction is perfect, and even good abstractions cannot cover every possible use case\. While the value of the AWS CDK's model is plain, sometimes you'll come upon a construct that's perfect for your needs—if only you could make a small \(or large\) tweak\. For this reason, the AWS CDK provides ways to "break out" of the construct model, moving to a lower level of abstraction or to a different model entirely\. As their name implies, the CDK's *escape hatches* let you "escape" the AWS CDK paradigm and extend it in ways the AWS CDK designers never anticipated\. Then you can wrap all that in a new construct to hide the underlying complexity and provide a clean API for developers\.

Some situations in which you'll reach for escape hatches include:
+ An AWS service feature is available through AWS CloudFormation, but there are no L2 constructs for it\.
+ An AWS service feature is available through AWS CloudFormation, and there are L2 constructs for the service, but these don't yet expose the feature\. Because L2 constructs are developed "by hand," they may sometimes lag behind the L1 constructs\.
+ The feature is not yet available through AWS CloudFormation at all\.

To determine whether a feature is available through AWS CloudFormation, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

## Using AWS CloudFormation constructs directly<a name="cfn_layer_cfn"></a>

If there are no L23 classes available for the service, you can fall back to the automatically generated L1 constructs, which map 1:1 onto all available AWS CloudFormation resources and properties\. These resources can be recognized by their name starting with `Cfn`, such as `CfnBucket` or `CfnRole`\. You instantiate them exactly as you would use the equivalent AWS CloudFormation resource\. For more information, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

For example, to instantiate a low\-level Amazon S3 bucket L1 with analytics enabled, you would write something like the following\.

------
#### [ TypeScript ]

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

------
#### [ JavaScript ]

```
new s3.CfnBucket(this, 'MyBucket', {
  analyticsConfigurations: [
    { 
      id: 'Config'
      // ...        
    } 
  ]
});
```

------
#### [ Python ]

```
s3.CfnBucket(self, "MyBucket",
    analytics_configurations: [
        dict(id="Config",
             # ...
             )
    ]
)
```

------
#### [ Java ]

```
CfnBucket.Builder.create(this, "MyBucket")
    .analyticsConfigurations(Arrays.asList(java.util.Map.of(    // Java 9 or later
        "id", "Config", // ...
    ))).build();
```

------
#### [ C\# ]

```
new CfnBucket(this, 'MyBucket', new CfnBucketProps {
    AnalyticsConfigurations = new Dictionary<string, string>
    {
        ["id"] = "Config",
        // ...
    }
});
```

------

In the rare case where you want to define a resource that doesn't have a corresponding `CfnXxx` class, such as a new resource type that hasn't yet been published in the AWS CloudFormation resource specification, you can instantiate the `cdk.CfnResource` directly and specify the resource type and properties\. This is shown in the following example\.

------
#### [ TypeScript ]

```
new cdk.CfnResource(this, 'MyBucket', {
  type: 'AWS::S3::Bucket',
  properties: {
    // Note the PascalCase here! These are CloudFormation identifiers.
    AnalyticsConfigurations: [
      {
        Id: 'Config',
        // ...
      }
    ] 
  }
});
```

------
#### [ JavaScript ]

```
new cdk.CfnResource(this, 'MyBucket', {
  type: 'AWS::S3::Bucket',
  properties: {
    // Note the PascalCase here! These are CloudFormation identifiers.
    AnalyticsConfigurations: [
      {
        Id: 'Config'
        // ...
      }
    ] 
  }
});
```

------
#### [ Python ]

```
cdk.CfnResource(self, 'MyBucket',
  type="AWS::S3::Bucket",
  properties=dict(
    # Note the PascalCase here! These are CloudFormation identifiers.
    "AnalyticsConfigurations": [
      {
        "Id": "Config",
        # ...
      }
    ] 
  }
)
```

------
#### [ Java ]

```
CfnResource.Builder.create(this, "MyBucket")
        .type("AWS::S3::Bucket")
        .properties(java.util.Map.of(    // Map.of requires Java 9 or later
            // Note the PascalCase here! These are CloudFormation identifiers
            "AnalyticsConfigurations", Arrays.asList(
                    java.util.Map.of("Id", "Config", // ...
                    ))))
        .build();
```

------
#### [ C\# ]

```
new CfnResource(this, "MyBucket", new CfnResourceProps
{
    Type = "AWS::S3::Bucket",
    Properties = new Dictionary<string, object>
    {   // Note the PascalCase here! These are CloudFormation identifiers
        ["AnalyticsConfigurations"] = new Dictionary<string, string>[]
        {
            new Dictionary<string, string> {
                ["Id"] = "Config"
            }
        }
    }
});
```

------

For more information, see [AWS Resource and Property Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)\.

## Modifying the AWS CloudFormation resource behind AWS constructs<a name="cfn_layer_resource"></a>

If a L2 construct is missing a feature or you are trying to work around an issue, you can modify the L1 construct that is encapsulated by the L2 construct\.

All L2 constructs contain within them the corresponding L1 construct\. For example, the high\-level `Bucket` construct wraps the low\-level `CfnBucket` construct\. Because the `CfnBucket` corresponds directly to the AWS CloudFormation resource, it exposes all features that are available through AWS CloudFormation\.

The basic approach to get access to the L1 class is to use `construct.node.defaultChild` \(Python: `default_child`\), cast it to the right type \(if necessary\), and modify its properties\. Again, let's take the example of a `Bucket`\.

------
#### [ TypeScript ]

```
// Get the CloudFormation resource
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;

// Change its properties
cfnBucket.analyticsConfiguration = [
  { 
    id: 'Config',
    // ...        
  } 
];
```

------
#### [ JavaScript ]

```
// Get the CloudFormation resource
const cfnBucket = bucket.node.defaultChild;

// Change its properties
cfnBucket.analyticsConfiguration = [
  { 
    id: 'Config'
    // ...        
  } 
];
```

------
#### [ Python ]

```
# Get the CloudFormation resource
cfn_bucket = bucket.node.default_child

# Change its properties
cfn_bucket.analytics_configuration = [
    {
        "id": "Config",
        # ...
    }
]
```

------
#### [ Java ]

```
// Get the CloudFormation resource
CfnBucket cfnBucket = (CfnBucket)bucket.getNode().getDefaultChild();

cfnBucket.setAnalyticsConfigurations(
        Arrays.asList(java.util.Map.of(    // Java 9 or later
            "Id", "Config", // ...
        ));
```

------
#### [ C\# ]

```
// Get the CloudFormation resource
var cfnBucket = (CfnBucket)bucket.Node.DefaultChild;

cfnBucket.AnalyticsConfigurations = new List<object> {
    new Dictionary<string, string>
    {
        ["Id"] = "Config",
        // ...
    }
};
```

------

You can also use this object to change AWS CloudFormation options such as `Metadata` and `UpdatePolicy`\.

------
#### [ TypeScript ]

```
cfnBucket.cfnOptions.metadata = {
  MetadataKey: 'MetadataValue'
};
```

------
#### [ JavaScript ]

```
cfnBucket.cfnOptions.metadata = {
  MetadataKey: 'MetadataValue'
};
```

------
#### [ Python ]

```
cfn_bucket.cfn_options.metadata = {
    "MetadataKey": "MetadataValue"
}
```

------
#### [ Java ]

```
cfnBucket.getCfnOptions().setMetadata(java.util.Map.of(    // Java 9+
    "MetadataKey", "Metadatavalue"));
```

------
#### [ C\# ]

```
cfnBucket.CfnOptions.Metadata = new Dictionary<string, object>
{
    ["MetadataKey"] = "Metadatavalue"
};
```

------

## Raw overrides<a name="cfn_layer_raw"></a>

If there are properties that are missing from the L1 construct, you can bypass all typing using raw overrides\. This also makes it possible to delete synthesized properties\. 

Use one of the `addOverride` methods \(Python: `add_override`\) methods, as shown in the following example\.

------
#### [ TypeScript ]

```
// Get the CloudFormation resource
const cfnBucket = bucket.node.defaultChild as s3.CfnBucket;

// Use dot notation to address inside the resource template fragment
cfnBucket.addOverride('Properties.VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addDeletionOverride('Properties.VersioningConfiguration.Status');

// use index (0 here) to address an element of a list
cfnBucket.addOverride('Properties.Tags.0.Value', 'NewValue');
cfnBucket.addDeletionOverride('Properties.Tags.0');

// addPropertyOverride is a convenience function for paths starting with "Properties."
cfnBucket.addPropertyOverride('VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addPropertyDeletionOverride('VersioningConfiguration.Status');
cfnBucket.addPropertyOverride('Tags.0.Value', 'NewValue');
cfnBucket.addPropertyDeletionOverride('Tags.0');
```

------
#### [ JavaScript ]

```
// Get the CloudFormation resource
const cfnBucket = bucket.node.defaultChild ;

// Use dot notation to address inside the resource template fragment
cfnBucket.addOverride('Properties.VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addDeletionOverride('Properties.VersioningConfiguration.Status');

// use index (0 here) to address an element of a list
cfnBucket.addOverride('Properties.Tags.0.Value', 'NewValue');
cfnBucket.addDeletionOverride('Properties.Tags.0');

// addPropertyOverride is a convenience function for paths starting with "Properties."
cfnBucket.addPropertyOverride('VersioningConfiguration.Status', 'NewStatus');
cfnBucket.addPropertyDeletionOverride('VersioningConfiguration.Status');
cfnBucket.addPropertyOverride('Tags.0.Value', 'NewValue');
cfnBucket.addPropertyDeletionOverride('Tags.0');
```

------
#### [ Python ]

```
# Get the CloudFormation resource
cfn_bucket = bucket.node.default_child

# Use dot notation to address inside the resource template fragment
cfn_bucket.add_override("Properties.VersioningConfiguration.Status", "NewStatus")
cfn_bucket.add_deletion_override("Properties.VersioningConfiguration.Status")

# use index (0 here) to address an element of a list
cfn_bucket.add_override("Properties.Tags.0.Value", "NewValue")
cfn_bucket.add_deletion_override("Properties.Tags.0")

# addPropertyOverride is a convenience function for paths starting with "Properties."
cfn_bucket.add_property_override("VersioningConfiguration.Status", "NewStatus")
cfn_bucket.add_property_deletion_override("VersioningConfiguration.Status")
cfn_bucket.add_property_override("Tags.0.Value", "NewValue")
cfn_bucket.add_property_deletion_override("Tags.0")
```

------
#### [ Java ]

```
// Get the CloudFormation resource
CfnBucket cfnBucket = (CfnBucket)bucket.getNode().getDefaultChild();

// Use dot notation to address inside the resource template fragment
cfnBucket.addOverride("Properties.VersioningConfiguration.Status", "NewStatus");
cfnBucket.addDeletionOverride("Properties.VersioningConfiguration.Status");

// use index (0 here) to address an element of a list
cfnBucket.addOverride("Properties.Tags.0.Value", "NewValue");
cfnBucket.addDeletionOverride("Properties.Tags.0");

// addPropertyOverride is a convenience function for paths starting with "Properties."
cfnBucket.addPropertyOverride("VersioningConfiguration.Status", "NewStatus");
cfnBucket.addPropertyDeletionOverride("VersioningConfiguration.Status");
cfnBucket.addPropertyOverride("Tags.0.Value", "NewValue");
cfnBucket.addPropertyDeletionOverride("Tags.0");
```

------
#### [ C\# ]

```
// Get the CloudFormation resource
var cfnBucket = (CfnBucket)bucket.node.defaultChild;

// Use dot notation to address inside the resource template fragment
cfnBucket.AddOverride("Properties.VersioningConfiguration.Status", "NewStatus");
cfnBucket.AddDeletionOverride("Properties.VersioningConfiguration.Status");

// use index (0 here) to address an element of a list
cfnBucket.AddOverride("Properties.Tags.0.Value", "NewValue");
cfnBucket.AddDeletionOverride("Properties.Tags.0");

// addPropertyOverride is a convenience function for paths starting with "Properties."
cfnBucket.AddPropertyOverride("VersioningConfiguration.Status", "NewStatus");
cfnBucket.AddPropertyDeletionOverride("VersioningConfiguration.Status");
cfnBucket.AddPropertyOverride("Tags.0.Value", "NewValue");
cfnBucket.AddPropertyDeletionOverride("Tags.0");
```

------

## Custom resources<a name="cfn_layer_custom"></a>

If the feature isn't available through AWS CloudFormation, but only through a direct API call, the only solution is to write an AWS CloudFormation Custom Resource to make the API call you need\. Don't worry, the AWS CDK makes it easier to write these, and wrap them up into a regular construct interface, so from another user's perspective the feature feels native\.

Building a custom resource involves writing a Lambda function that responds to a resource's CREATE, UPDATE and DELETE lifecycle events\. If your custom resource needs to make only a single API call, consider using the [AwsCustomResource](https://github.com/awslabs/aws-cdk/tree/master/packages/%40aws-cdk/custom-resources)\. This makes it possible to perform arbitrary SDK calls during an AWS CloudFormation deployment\. Otherwise, you should write your own Lambda function to perform the work you need to get done\.

The subject is too broad to completely cover here, but the following links should get you started:
+ [Custom Resources](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html)
+ [Custom\-Resource Example](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript/custom-resource/)
+ For a more fully fledged example, see the [DnsValidatedCertificate](https://github.com/awslabs/aws-cdk/blob/master/packages/@aws-cdk/aws-certificatemanager/lib/dns-validated-certificate.ts) class in the CDK standard library\. This is implemented as a custom resource\.