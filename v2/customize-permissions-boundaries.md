# Create and apply permissions boundaries for the AWS CDK<a name="customize-permissions-boundaries"></a>

A *permissions boundary* is an AWS Identity and Access Management \(IAM\) advanced feature that you can use to set the maximum permissions that an IAM entity, such as a user or role, can have\. You can use permissions boundaries to restrict the actions that IAM entities can perform when using the AWS Cloud Development Kit \(AWS CDK\)\.

To learn more about permissions boundaries, see [Permissions boundaries for IAM entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) in the *IAM User Guide*\.

## When to use permissions boundaries with the AWS CDK<a name="customize-permissions-boundaries-when"></a>

Consider applying permissions boundaries when you need to restrict developers in your organization from performing certain actions with the AWS CDK\. For example, if there are specific resources in your AWS environment that you don’t want developers to modify, you can create and apply a permissions boundary\.

## How to apply permissions boundaries with the AWS CDK<a name="customize-permissions-boundaries-how"></a>

### Create the permissions boundary<a name="customize-permissions-boundaries-how-create"></a>

First, you create the permissions boundary, using an AWS managed policy or a customer managed policy to set the boundary for an IAM entity \(user or role\)\. This policy limits the maximum permissions for the user or role\. For instructions on creating permissions boundaries, see [Permissions boundaries for IAM entities](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) in the *IAM User Guide*\.

Permissions boundaries set the maximum permissions that an IAM entity can have, but don’t grant permissions on their own\. You must use permissions boundaries with IAM policies to effectively limit and grant the proper permissions for your organization\. You must also prevent IAM entities from being able to escape the boundaries that you set\. For an example, see [Delegating responsibility to others using permissions boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html#access_policies_boundaries-delegate) in the *IAM User Guide*\.

### Apply the permissions boundary during bootstrapping<a name="customize-permissions-boundaries-how-apply"></a>

After creating the permissions boundary, you can enforce it for the AWS CDK by applying it during bootstrapping\.

Use the `\-\-custom\-permissions\-boundary` option and specify the name of the permissions boundary to apply\. The following is an example that applies a permissions boundary named `cdk-permissions-boundary`:

```
$ cdk bootstrap --custom-permissions-boundary cdk-permissions-boundary
```

By default, the CDK uses the `CloudFormationExecutionRole` IAM role, defined in the bootstrap template, to receive permissions for performing deployments\. By applying the custom permissions boundary during bootstrapping, the permissions boundary gets attached to this role\. The permissions boundary will then set the maximum permissions that can be performed by developers in your organization when using the AWS CDK\. To learn more about this role, see [IAM roles created during bootstrapping](bootstrapping-env.md#bootstrapping-env-roles)\.

When you apply permissions boundaries in this way, they are applied to the specific environment that you bootstrap\. To use the same permissions boundary across multiple environments, you must apply the permissions boundary for each environment during bootstrapping\. You can also apply different permissions boundaries for different environments\.

## Learn more<a name="customize-permissions-boundaries-learn"></a>

For more information on permissions boundaries, see [When and where to use IAM permissions boundaries](https://aws.amazon.com/blogs/security/when-and-where-to-use-iam-permissions-boundaries/) in the *AWS Security Blog*\.