# Permissions and the AWS CDK<a name="permissions"></a>

The AWS Construct Library uses a few common, widely implemented idioms to manage access and permissions\. The IAM module provides you with the tools you need to use these idioms\.

AWS CDK uses AWS CloudFormation to deploy changes\. Every deployment involves an actor \(either a developer, or an automated system\) that starts a AWS CloudFormation deployment\. In the course of doing this, the actor will assume one or more IAM Identities \(user or roles\) and optionally pass a role to AWS CloudFormation\. 

 If you use AWS IAM Identity Center to authenticate as a user, then the single sign\-on provider supplies short\-lived session credentials that authorize you to act as a pre\-defined IAM role\. To learn how the AWS CDK obtains AWS credentials from IAM Identity Center authentication, see [Understand IAM Identity Center authentication](https://docs.aws.amazon.com/sdkref/latest/guide/understanding-sso.html) in the *AWS SDKs and Tools Reference Guide*\. 

## Principals<a name="permissions_principals"></a>

An IAM principal is an authenticated AWS entity representing a user, service, or application that can call AWS APIs\. The AWS Construct Library supports specifying principals in several flexible ways to grant them access your AWS resources\.

In security contexts, the term "principal" refers specifically to authenticated entities such as users\. Objects like groups and roles do not *represent* users \(and other authenticated entities\) but rather *identify* them indirectly for the purpose of granting permissions\.

For example, if you create an IAM group, you can grant the group \(and thus its members\) write access to an Amazon RDS table\. However, the group itself is not a principal because it doesn't represent a single entity \(also, you cannot log in to a group\)\.

In the CDK's IAM library, classes that directly or indirectly identify principals implement the [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.IPrincipal.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.IPrincipal.html) interface, allowing these objects to be used interchangeably in access policies\. However, not all of them are principals in the security sense\. These objects include:

1. IAM resources such as `[Role](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html)`, `[User](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.User.html)`, and `[Group](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Group.html)`

1. Service principals \(`new iam.[ServicePrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ServicePrincipal.html)('service.amazonaws.com')`\)

1. Federated principals \(`new iam.[FederatedPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.FederatedPrincipal.html)('cognito-identity.amazonaws.com')`\)

1. Account principals \(`new iam.[AccountPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.AccountPrincipal.html)('0123456789012'))`

1. Canonical user principals \(`new iam.[CanonicalUserPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.CanonicalUserPrincipal.html)('79a59d[...]7ef2be')`\)

1. AWS Organizations principals \(`new iam.[OrganizationPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.OrganizationPrincipal.html)('org-id')`\)

1. Arbitrary ARN principals \(`new iam.[ArnPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ArnPrincipal.html)(res.arn)`\)

1. An `iam.[CompositePrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.CompositePrincipal.html)(principal1, principal2, ...)` to trust multiple principals

## Grants<a name="permissions_grants"></a>

Every construct that represents a resource that can be accessed, such as an Amazon S3 bucket or Amazon DynamoDB table, has methods that grant access to another entity\. All such methods have names starting with **grant**\.

For example, Amazon S3 buckets have the methods `[grantRead](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#grantwbrreadidentity-objectskeypattern)` and `[grantReadWrite](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html#grantwbrreadwbrwriteidentity-objectskeypattern)` \(Python: `grant_read`, `grant_read_write`\) to enable read and read/write access, respectively, from an entity to the bucket\. The entity doesn't have to know exactly which Amazon S3 IAM permissions are required to perform these operations\.

The first argument of a **grant** method is always of type [IGrantable](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.IGrantable.html)\. This interface represents entities that can be granted permissions\. That is, it represents resources with roles, such as the IAM objects `[Role](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html)`, `[User](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.User.html)`, and `[Group](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.User.html)`\.

Other entities can also be granted permissions\. For example, later in this topic, we show how to grant a CodeBuild project access to an Amazon S3 bucket\. Generally, the associated role is obtained via a `role` property on the entity being granted access\.

Resources that use execution roles, such as `[lambda\.Function](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda.Function.html)`, also implement `IGrantable`, so you can grant them access directly instead of granting access to their role\. For example, if `bucket` is an Amazon S3 bucket, and `function` is a Lambda function, the following code grants the function read access to the bucket\.

------
#### [ TypeScript ]

```
bucket.grantRead(function);
```

------
#### [ JavaScript ]

```
bucket.grantRead(function);
```

------
#### [ Python ]

```
bucket.grant_read(function)
```

------
#### [ Java ]

```
bucket.grantRead(function);
```

------
#### [ C\# ]

```
bucket.GrantRead(function);
```

------

 Sometimes permissions must be applied while your stack is being deployed\. One such case is when you grant an AWS CloudFormation custom resource access to some other resource\. The custom resource will be invoked during deployment, so it must have the specified permissions at deployment time\.

Another case is when a service verifies that the role you pass to it has the right policies applied\. \(A number of AWS services do this to make sure that you didn't forget to set the policies\.\) In those cases, the deployment might fail if the permissions are applied too late\. 

 To force the grant's permissions to be applied before another resource is created, you can add a dependency on the grant itself, as shown here\. Though the return value of grant methods is commonly discarded, every grant method in fact returns an `iam.Grant` object\.

------
#### [ TypeScript ]

```
const grant = bucket.grantRead(lambda);
const custom = new CustomResource(...);
custom.node.addDependency(grant);
```

------
#### [ JavaScript ]

```
const grant = bucket.grantRead(lambda);
const custom = new CustomResource(...);
custom.node.addDependency(grant);
```

------
#### [ Python ]

```
grant = bucket.grant_read(function)
custom = CustomResource(...)
custom.node.add_dependency(grant)
```

------
#### [ Java ]

```
Grant grant = bucket.grantRead(function);
CustomResource custom = new CustomResource(...);
custom.node.addDependency(grant);
```

------
#### [ C\# ]

```
var grant = bucket.GrantRead(function);
var custom = new CustomResource(...);
custom.node.AddDependency(grant);
```

------

## Roles<a name="permissions_roles"></a>

The IAM package contains a `[Role](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html)` construct that represents IAM roles\. The following code creates a new role, trusting the Amazon EC2 service\.

------
#### [ TypeScript ]

```
import * as iam from 'aws-cdk-lib/aws-iam';

const role = new iam.Role(this, 'Role', {
  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),   // required
});
```

------
#### [ JavaScript ]

```
const iam = require('aws-cdk-lib/aws-iam');

const role = new iam.Role(this, 'Role', {
  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com')   // required
});
```

------
#### [ Python ]

```
import aws_cdk.aws_iam as iam
    
role = iam.Role(self, "Role",
          assumed_by=iam.ServicePrincipal("ec2.amazonaws.com")) # required
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.iam.Role;
import software.amazon.awscdk.services.iam.ServicePrincipal;

Role role = Role.Builder.create(this, "Role")
        .assumedBy(new ServicePrincipal("ec2.amazonaws.com")).build();
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.IAM;

var role = new Role(this, "Role", new RoleProps
{
    AssumedBy = new ServicePrincipal("ec2.amazonaws.com"),   // required
});
```

------

You can add permissions to a role by calling the role's `[addToPolicy](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html#addwbrtowbrpolicystatement)` method \(Python: `add_to_policy`\), passing in a `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html)` that defines the rule to be added\. The statement is added to the role's default policy; if it has none, one is created\. 

 The following example adds a `Deny` policy statement to the role for the actions `ec2:SomeAction` and `s3:AnotherAction` on the resources `bucket` and `otherRole` \(Python: `other_role`\), under the condition that the authorized service is AWS CodeBuild\.

------
#### [ TypeScript ]

```
role.addToPolicy(new iam.PolicyStatement({
  effect: iam.Effect.DENY,
  resources: [bucket.bucketArn, otherRole.roleArn],
  actions: ['ec2:SomeAction', 's3:AnotherAction'],
  conditions: {StringEquals: {
    'ec2:AuthorizedService': 'codebuild.amazonaws.com',
}}}));
```

------
#### [ JavaScript ]

```
role.addToPolicy(new iam.PolicyStatement({
  effect: iam.Effect.DENY,
  resources: [bucket.bucketArn, otherRole.roleArn],
  actions: ['ec2:SomeAction', 's3:AnotherAction'],
  conditions: {StringEquals: {
    'ec2:AuthorizedService': 'codebuild.amazonaws.com'
}}}));
```

------
#### [ Python ]

```
role.add_to_policy(iam.PolicyStatement(
    effect=iam.Effect.DENY,
    resources=[bucket.bucket_arn, other_role.role_arn],
    actions=["ec2:SomeAction", "s3:AnotherAction"],
    conditions={"StringEquals": {
        "ec2:AuthorizedService": "codebuild.amazonaws.com"}}
))
```

------
#### [ Java ]

```
role.addToPolicy(PolicyStatement.Builder.create()
        .effect(Effect.DENY)
        .resources(Arrays.asList(bucket.getBucketArn(), otherRole.getRoleArn()))
        .actions(Arrays.asList("ec2:SomeAction", "s3:AnotherAction"))
        .conditions(java.util.Map.of(    // Map.of requires Java 9 or later
            "StringEquals", java.util.Map.of(
                "ec2:AuthorizedService", "codebuild.amazonaws.com")))
        .build());
```

------
#### [ C\# ]

```
role.AddToPolicy(new PolicyStatement(new PolicyStatementProps
{
    Effect = Effect.DENY,
    Resources = new string[] { bucket.BucketArn, otherRole.RoleArn },
    Actions = new string[] { "ec2:SomeAction", "s3:AnotherAction" },
    Conditions = new Dictionary<string, object>
    {
        ["StringEquals"] = new Dictionary<string, string>
        {
            ["ec2:AuthorizedService"] = "codebuild.amazonaws.com"
        }
    }
}));
```

------

 In the preceding example, we've created a new `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html)` inline with the `[addToPolicy](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html#addwbrtowbrpolicystatement)` \(Python: `add_to_policy`\) call\. You can also pass in an existing policy statement or one you've modified\. The [PolicyStatement](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html) object has [numerous methods](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html#methods) for adding principals, resources, conditions, and actions\. 

If you're using a construct that requires a role to function correctly, you can do one of the following:
+ Pass in an existing role when instantiating the construct object\.
+ Let the construct create a new role for you, trusting the appropriate service principal\. The following example uses such a construct: a CodeBuild project\.

------
#### [ TypeScript ]

```
import * as codebuild from 'aws-cdk-lib/aws-codebuild';

// imagine roleOrUndefined is a function that might return a Role object
// under some conditions, and undefined under other conditions
const someRole: iam.IRole | undefined = roleOrUndefined();

const project = new codebuild.Project(this, 'Project', {
  // if someRole is undefined, the Project creates a new default role, 
  // trusting the codebuild.amazonaws.com service principal
  role: someRole,
});
```

------
#### [ JavaScript ]

```
const codebuild = require('aws-cdk-lib/aws-codebuild');

// imagine roleOrUndefined is a function that might return a Role object
// under some conditions, and undefined under other conditions
const someRole = roleOrUndefined();

const project = new codebuild.Project(this, 'Project', {
  // if someRole is undefined, the Project creates a new default role, 
  // trusting the codebuild.amazonaws.com service principal
  role: someRole
});
```

------
#### [ Python ]

```
import aws_cdk.aws_codebuild as codebuild

# imagine role_or_none is a function that might return a Role object
# under some conditions, and None under other conditions
some_role = role_or_none();

project = codebuild.Project(self, "Project",
# if role is None, the Project creates a new default role, 
# trusting the codebuild.amazonaws.com service principal
role=some_role)
```

------
#### [ Java ]

```
import software.amazon.awscdk.services.iam.Role;
import software.amazon.awscdk.services.codebuild.Project;

// imagine roleOrNull is a function that might return a Role object
// under some conditions, and null under other conditions
Role someRole = roleOrNull();

// if someRole is null, the Project creates a new default role, 
// trusting the codebuild.amazonaws.com service principal
Project project = Project.Builder.create(this, "Project")
        .role(someRole).build();
```

------
#### [ C\# ]

```
using Amazon.CDK.AWS.CodeBuild;

// imagine roleOrNull is a function that might return a Role object
// under some conditions, and null under other conditions
var someRole = roleOrNull();

// if someRole is null, the Project creates a new default role, 
// trusting the codebuild.amazonaws.com service principal
var project = new Project(this, "Project", new ProjectProps
{
    Role = someRole
});
```

------

Once the object is created, the role \(whether the role passed in or the default one created by the construct\) is available as the property `role`\. However, this property is not available on external resources\. Therefore, these constructs have an `addToRolePolicy` \(Python: `add_to_role_policy`\) method\.

The method does nothing if the construct is an external resource, and it calls the `addToPolicy` \(Python: `add_to_policy`\) method of the `role` property otherwise\. This saves you the trouble of handling the undefined case explicitly\.

The following example demonstrates:

------
#### [ TypeScript ]

```
// project is imported into the CDK application
const project = codebuild.Project.fromProjectName(this, 'Project', 'ProjectName');

// project is imported, so project.role is undefined, and this call has no effect
project.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,   // ... and so on defining the policy
}));
```

------
#### [ JavaScript ]

```
// project is imported into the CDK application
const project = codebuild.Project.fromProjectName(this, 'Project', 'ProjectName');

// project is imported, so project.role is undefined, and this call has no effect
project.addToRolePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW   // ... and so on defining the policy
}));
```

------
#### [ Python ]

```
# project is imported into the CDK application
project = codebuild.Project.from_project_name(self, 'Project', 'ProjectName')

# project is imported, so project.role is undefined, and this call has no effect
project.add_to_role_policy(iam.PolicyStatement(
  effect=iam.Effect.ALLOW,   # ... and so on defining the policy
)
```

------
#### [ Java ]

```
// project is imported into the CDK application
Project project = Project.fromProjectName(this, "Project", "ProjectName");

// project is imported, so project.getRole() is null, and this call has no effect
project.addToRolePolicy(PolicyStatement.Builder.create()
        .effect(Effect.ALLOW)   // .. and so on defining the policy
        .build();
```

------
#### [ C\# ]

```
// project is imported into the CDK application
var project = Project.FromProjectName(this, "Project", "ProjectName");

// project is imported, so project.role is null, and this call has no effect
project.AddToRolePolicy(new PolicyStatement(new PolicyStatementProps
{
    Effect = Effect.ALLOW, // ... and so on defining the policy 
}));
```

------

## Resource policies<a name="permissions_resource_policies"></a>

A few resources in AWS, such as Amazon S3 buckets and IAM roles, also have a resource policy\. These constructs have an `addToResourcePolicy` method \(Python: `add_to_resource_policy`\), which takes a `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.PolicyStatement.html)` as its argument\. Every policy statement added to a resource policy must specify at least one principal\.

In the following example, the [Amazon S3 bucket](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_s3.Bucket.html) `bucket` grants a role with the `s3:SomeAction` permission to itself\.

------
#### [ TypeScript ]

```
bucket.addToResourcePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: ['s3:SomeAction'],
  resources: [bucket.bucketArn],
  principals: [role]
}));
```

------
#### [ JavaScript ]

```
bucket.addToResourcePolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: ['s3:SomeAction'],
  resources: [bucket.bucketArn],
  principals: [role]
}));
```

------
#### [ Python ]

```
bucket.add_to_resource_policy(iam.PolicyStatement(
    effect=iam.Effect.ALLOW,
    actions=["s3:SomeAction"],
    resources=[bucket.bucket_arn],
    principals=role))
```

------
#### [ Java ]

```
bucket.addToResourcePolicy(PolicyStatement.Builder.create()
        .effect(Effect.ALLOW)
        .actions(Arrays.asList("s3:SomeAction"))
        .resources(Arrays.asList(bucket.getBucketArn()))
        .principals(Arrays.asList(role))
        .build());
```

------
#### [ C\# ]

```
bucket.AddToResourcePolicy(new PolicyStatement(new PolicyStatementProps
{
    Effect = Effect.ALLOW,
    Actions = new string[] { "s3:SomeAction" },
    Resources = new string[] { bucket.BucketArn },
    Principals = new IPrincipal[] { role }
}));
```

------

## Using external IAM objects<a name="permissions_existing"></a>

If you have defined an IAM user, principal, group, or role outside your AWS CDK app, you can use that IAM object in your AWS CDK app\. To do so, create a reference to it using its ARN or its name\. \(Use the name for users, groups, and roles\.\) The returned reference can then be used to grant permissions or to construct policy statements as explained previously\.
+ For users, call `[User\.fromUserArn\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.User.html#static-fromwbruserwbrarnscope-id-userarn)` or `[User\.fromUserName\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.User.html#static-fromwbruserwbrnamescope-id-username)`\. `User.fromUserAttributes()` is also available, but currently provides the same functionality as `User.fromUserArn()`\.
+ For principals, instantiate an `[ArnPrincipal](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ArnPrincipal.html)` object\.
+ For groups, call `[Group\.fromGroupArn\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Group.html#static-fromwbrgroupwbrarnscope-id-grouparn)` or `[Group\.fromGroupName\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Group.html#static-fromwbrgroupwbrnamescope-id-groupname)`\.
+ For roles, call `[Role\.fromRoleArn\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html#static-fromwbrrolewbrarnscope-id-rolearn-options)` or `[Role\.fromRoleName\(\)](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Role.html#static-fromwbrrolewbrnamescope-id-rolename)`\.

Policies \(including managed policies\) can be used in similar fashion using the following methods\. You can use references to these objects anywhere an IAM policy is required\.
+ `[Policy\.fromPolicyName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.Policy.html#static-fromwbrpolicywbrnamescope-id-policyname)`
+ `[ManagedPolicy\.fromManagedPolicyArn](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ManagedPolicy.html#static-fromwbrmanagedwbrpolicywbrarnscope-id-managedpolicyarn)`
+ `[ManagedPolicy\.fromManagedPolicyName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ManagedPolicy.html#static-fromwbrmanagedwbrpolicywbrnamescope-id-managedpolicyname)`
+ `[ManagedPolicy\.fromAwsManagedPolicyName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_iam.ManagedPolicy.html#static-fromwbrawswbrmanagedwbrpolicywbrnamemanagedpolicyname)`

**Note**  
As with all references to external AWS resources, you cannot modify external IAM objects in your CDK app\.