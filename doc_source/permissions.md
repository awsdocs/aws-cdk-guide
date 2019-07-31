# Permissions<a name="permissions"></a>

The AWS CDK contains an IAM package to help you deal with permissions\. There are a few idioms for managing access and permissions that are implemented uniformly across the entire AWS CDK Construct Library\.

## Roles<a name="permissions_roles"></a>

The IAM package contains a `Role` construct that enables you to manage IAM **Role** instances\. The following code creates a new **Role**, trusting the Amazon EC2 Service Principal\.

```
import iam = require('@aws-cdk/aws-iam');

const role = new iam.Role(this, 'Role', {
  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),
});
```

You can add permissions to a role by calling methods on it, and passing the appropriate `Policy` statement\. The following example adds a `Deny` statement to the role for the actions `ec2:SomeAction` and `s3:AnotherAction` on the resources `bucket` and `otherRole`, under the condition that the authorized service is AWS CodeBuild\.

```
role.addToPolicy(new iam.PolicyStatement(PolicyStatementEffect.Deny) // default is Allow
  // there's also addResource() to add one, and addAllResources() to add '*'
  .addResources(bucket.bucketArn, otherRole.roleArn)
  // there's also addAction() to add one
  .addActions('ec2:SomeAction', 's3:AnotherAction')
  .addCondition('StringEquals', {
    'ec2:AuthorizedService': 'codebuild.amazonaws.com',
  })
);
```

If you're using a construct that requires a role to function correctly, you can either pass in an existing role when instantiating the construct object, or let the construct create a new role for you, trusting the appropriate service principal\. The following example of using such a construct, in this case a CodeBuild project\.

```
import codebuild = require('@aws-cdk/aws-codebuild');

// imagine roleOrUndefined is a function that might return a Role object
// under some conditions, and undefined under other conditions
const someRole: iam.IRole | undefined = roleOrUndefined();

const project = new codebuild.Project(this, 'Project', {
  // if someRole is undefined, a new role will be created,
  // trusting the codebuild.amazonaws.com service principal
  role: someRole,
});
```

In either case, once the object is created, the role is available as the property role on the construct\. That property is not available on imported resources\. Because of that, the constructs have an `addToRolePolicy` method that does nothing if the construct is an imported resource, and calls the `addToPolicy` method of the `role` property \(passing the policy statement it received as argument\) otherwise, saving you from the trouble of handling the undefined case explicitly in your code\. The following example demonstrates the latter case\.

```
// project is imported into the CDK application
const project = codebuild.Project.fromProjectName(this, 'Project', 'ProjectName');

// project.role is undefined

// this method call will have no effect
project.addToRolePolicy(new iam.PolicyStatement()
  // ...
);
```

## Grants<a name="permissions_grants"></a>

Every construct that represents a resource that can be accessed, such as an Amazon S3 bucket or Amazon DynamoDB table, has methods that bestow a given level of access to another entity\. All those methods start with the prefix **grant**\. For example, Amazon S3 buckets have the methods `grantRead` and `grantReadWrite` to enable read and write access, respectively, from an entity to the bucket without having to know which exact Amazon S3 IAM actions are required to perform read or read/write operations\.

The first argument to the **grant** methods is always of the type `IGrantable`\. This type represents entities that can be granted permissions\. Those include all constructs in the AWS CDK Construct Library that represent resources with roles, such as CodeBuild projects as shown in the previous example, and IAM objects such as `Role`, `User`, and `Group`\.

## Resource Policies<a name="permissions_resource_policies"></a>

A few resources in AWS, such as Amazon S3 buckets and IAM roles, also have a resource policy\. In those cases, the construct exposes the `addToResourcePolicy` method, which takes a `PolicyStatement` as its argument, that enables you to modify the policy\. You must specify a `Principal` when dealing with resource policies\.

In the following example, the Amazon S3 bucket `bucket` grants a role with the `s3:SomeAction` permission to itself\.

```
bucket.addToResourcePolicy(new iam.PolicyStatement()
  .addAction('s3:SomeAction')
  .addResource(bucket.bucketArn)
  .addPrincipal(role)
);
```

## Principals<a name="permissions_principals"></a>

The AWS CDK Construct Library supports many types of principals, including:

1. IAM resources, such as `Roles`, `Users`, and `Groups`

1. Service principals \(`new iam.ServicePrincipal('service.amazonaws.com')`\)

1. Account principals \(`new iam.AccountPrincipal('0123456789012'))`

1. Canonical user principals \(`new iam.CanonicalUserPrincipal('79a59d[...]7ef2be')`\)

1. AWS organizations principals \(`new iam.OrganizationPrincipal('org-id')`\)

1. Arbitrary ARN principals \(`new iam.ArnPrincipal(res.arn)`\)

1. A `CompositePrincipal(principal1, principal2, ...)` if you need your role to trust multiple principals