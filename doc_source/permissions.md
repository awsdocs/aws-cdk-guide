# Permissions<a name="permissions"></a>

The AWS Construct Library uses a few common, widely\-implemented idioms to manage access and permissions\. The IAM module provides you with the tools you need to use these idioms\.

## Grants<a name="permissions_grants"></a>

Every construct that represents a resource that can be accessed, such as an Amazon S3 bucket or Amazon DynamoDB table, has methods that grant access to another entity\. All such methods have names starting with with **grant**\. For example, Amazon S3 buckets have the methods [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-readidentity-objectskeypattern](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-readidentity-objectskeypattern) and [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-read-writeidentity-objectskeypattern](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-read-writeidentity-objectskeypattern) to enable read and read/write access, respectively, from an entity to the bucket without having to know exactly which Amazon S3 IAM permissions are required to perform these operations\.

The first argument of a **grant** method is always of type [IGrantable](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.IGrantable.html)\. This interface represents entities that can be granted permissions—that is, resources with roles, such as the IAM objects [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html), [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html), and [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html)\.

Other entities can also be granted permissions\. For example, later in this topic, we show how to grant a CodeBuild project access to an Amazon S3 bucket\. Generally, the associated role is obtained via a `role` property on the entity being granted access\. Other enttites that can be granted permissions are Amazon EC2 instances and CodeBuild projects\.

Resources that use execution roles, such as [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html), also implement `IGrantable`, so you can grant them access directly \(`bucket.grantRead(lambda)`\) instead of granting access to their role \(`bucket.grantRead(lambda.role)`\)\.

## Roles<a name="permissions_roles"></a>

The IAM package contains a [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html) construct that represents IAM roles\. The following code creates a new role, trusting the Amazon EC2 Service Principal\.

```
import iam = require('@aws-cdk/aws-iam');

const role = new iam.Role(this, 'Role', {
  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),   // required
});
```

You can add permissions to a role by calling the role's [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.htm#add-to-policystatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.htm#add-to-policystatement) method, passing in a [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) that defines the rule to be added\. The statement is added to the role's default policy; if it has none, one is created\. 

 The following example adds a `Deny` policy statement to the role for the actions `ec2:SomeAction` and `s3:AnotherAction` on the resources `bucket` and `otherRole`, under the condition that the authorized service is AWS CodeBuild\.

```
role.addToPolicy(new iam.PolicyStatement({
  effect: iam.Effect.DENY,
  resources: [bucket.bucketArn, otherRole.roleArn],
  actions: ['ec2:SomeAction', 's3:AnotherAction'],
  conditions: {StringEquals: {
    'ec2:AuthorizedService': 'codebuild.amazonaws.com',
}}}));
```

**Note**  
 In our example above, we've created a new [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) inline with the [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html#add-to-policystatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html#add-to-policystatement) call\. You can also pass in an existing policy statement or one you've modified\. The [PolicyStatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) object has [numerous methods](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html#methods) for adding principals, resources, conditions, and actions\. 

If you're using a construct that requires a role to function correctly, you can either pass in an existing role when instantiating the construct object, or let the construct create a new role for you, trusting the appropriate service principal\. The following example uses such a construct: a CodeBuild project\.

```
import codebuild = require('@aws-cdk/aws-codebuild');

// imagine roleOrUndefined is a function that might return a Role object
// under some conditions, and undefined under other conditions
const someRole: iam.IRole | undefined = roleOrUndefined();

const project = new codebuild.Project(this, 'Project', {
  // if someRole is undefined, the Project creates a new default role, 
  // trusting the codebuild.amazonaws.com service principal
  role: someRole,
});
```

Once the object is created, the role \(whether the role passed in or the default one created by the construct\) is available as the property `role`\. This property is not available on imported resources, however, so the constructs have an `addToRolePolicy` method that does nothing if the construct is an imported resource, and calls the `addToPolicy` method of the `role` property otherwise, saving you the trouble of handling the undefined case explicitly\. The following example demonstrates:

```
// project is imported into the CDK application
const project = codebuild.Project.fromProjectName(this, 'Project', 'ProjectName');

// project is imported, so project.role is undefined, and this call has no effect
project.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,   // ... and so on defining the policy
}));
```

## Resource Policies<a name="permissions_resource_policies"></a>

A few resources in AWS, such as Amazon S3 buckets and IAM roles, also have a resource policy\. These constructs have an `addToResourcePolicy` method, which takes a [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) as its argument\. Every policy statement added to a resource policy must specify at least one principal\.

In the following example, the [Amazon S3 bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) `bucket` grants a role with the `s3:SomeAction` permission to itself\.

```
bucket.addToResourcePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: ['s3:SomeAction'],
  resources: [bucket.bucketArn],
  principals: [role]
}));
```

## Principals<a name="permissions_principals"></a>

The AWS CDK Construct Library supports many types of principals, including:

1. IAM resources such as [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html), [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html), and [https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html)

1. Service principals \(`new iam.[ServicePrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.ServicePrincipal.html)('service.amazonaws.com')`\)

1. Account principals \(`new iam.[AccountPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.AccountPrincipal.html)('0123456789012'))`

1. Canonical user principals \(`new iam.[CanonicalUserPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.CanonicalUserPrincipal.html)('79a59d[...]7ef2be')`\)

1. AWS organizations principals \(`new iam.[OrganizationPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.OrganizationPrincipal.html)('org-id')`\)

1. Arbitrary ARN principals \(`new iam.[ArnPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.ArnPrincipal.html)(res.arn)`\)

1. An `iam.[CompositePrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.CompositePrincipal.html)(principal1, principal2, ...)` to trust multiple principals