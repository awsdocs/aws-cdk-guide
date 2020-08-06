# Permissions<a name="permissions"></a>

The AWS Construct Library uses a few common, widely\-implemented idioms to manage access and permissions\. The IAM module provides you with the tools you need to use these idioms\.

## Principals<a name="permissions_principals"></a>

An IAM principal is an entity that can be authenticated in order to access AWS resources, such as a user, a service, or an application\. The AWS Construct Library supports many types of principals, including:

1. IAM resources such as `[Role](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html)`, `[User](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html)`, and `[Group](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html)`

1. Service principals \(`new iam.[ServicePrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.ServicePrincipal.html)('service.amazonaws.com')`\)

1. Federated principals \(`new iam.[FederatedPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.FederatedPrincipal.html)('cognito-identity.amazonaws.com')`\)

1. Account principals \(`new iam.[AccountPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.AccountPrincipal.html)('0123456789012'))`

1. Canonical user principals \(`new iam.[CanonicalUserPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.CanonicalUserPrincipal.html)('79a59d[...]7ef2be')`\)

1. AWS organizations principals \(`new iam.[OrganizationPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.OrganizationPrincipal.html)('org-id')`\)

1. Arbitrary ARN principals \(`new iam.[ArnPrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.ArnPrincipal.html)(res.arn)`\)

1. An `iam.[CompositePrincipal](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.CompositePrincipal.html)(principal1, principal2, ...)` to trust multiple principals

## Grants<a name="permissions_grants"></a>

Every construct that represents a resource that can be accessed, such as an Amazon S3 bucket or Amazon DynamoDB table, has methods that grant access to another entity\. All such methods have names starting with **grant**\. For example, Amazon S3 buckets have the methods `[grantRead](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-readidentity-objectskeypattern)` and `[grantReadWrite](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html#grant-read-writeidentity-objectskeypattern)` \(Python: `grant_read`, `grant_read_write`\) to enable read and read/write access, respectively, from an entity to the bucket without having to know exactly which Amazon S3 IAM permissions are required to perform these operations\.

The first argument of a **grant** method is always of type [IGrantable](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.IGrantable.html)\. This interface represents entities that can be granted permissions—that is, resources with roles, such as the IAM objects `[Role](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html)`, `[User](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.User.html)`, and `[Group](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Group.html)`\.

Other entities can also be granted permissions\. For example, later in this topic, we show how to grant a CodeBuild project access to an Amazon S3 bucket\. Generally, the associated role is obtained via a `role` property on the entity being granted access\. Other entities that can be granted permissions are Amazon EC2 instances and CodeBuild projects\.

Resources that use execution roles, such as `[lambda\.Function](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-lambda.Function.html)`, also implement `IGrantable`, so you can grant them access directly instead of granting access to their role\. For example, if `bucket` is an Amazon S3 bucket, and `function` is a Lambda function, the code below grants the function read access to the bucket\.

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

 Sometimes permissions must be applied while your stack is being deployed\. One such case is when you grant a AWS CloudFormation custom resource access to some other resource\. The custom resource will be invoked during deployment, so it must have the specified permissions at deployment time\. Another case is when a service verifies that the role you pass to it has the right policies applied \(a number of AWS services do this to make sure you didn't forget to set the policies\)\. In those cases, the deployment may fail if the permissions are applied too late\. 

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

The IAM package contains a `[Role](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html)` construct that represents IAM roles\. The following code creates a new role, trusting the Amazon EC2 service\.

------
#### [ TypeScript ]

```
import * as iam from '@aws-cdk/aws-iam';

const role = new iam.Role(this, 'Role', {
  assumedBy: new iam.ServicePrincipal('ec2.amazonaws.com'),   // required
});
```

------
#### [ JavaScript ]

```
const iam = require('@aws-cdk/aws-iam');

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

You can add permissions to a role by calling the role's `[addToPolicy](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html#add-to-policystatement)` method \(Python: `add_to_policy`\), passing in a `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html)` that defines the rule to be added\. The statement is added to the role's default policy; if it has none, one is created\. 

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
        .conditions(new HashMap<String, Object>() {{
            put("StringEquals", new HashMap<String, String>() {{
                put("ec2:AuthorizedService", "codebuild.amazonaws.com");                 
            }});
        }}).build());
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

 In our example above, we've created a new `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html)` inline with the `[addToPolicy](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.Role.html#add-to-policystatement)` \(Python: `add_to_policy`\) call\. You can also pass in an existing policy statement or one you've modified\. The [PolicyStatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html) object has [numerous methods](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html#methods) for adding principals, resources, conditions, and actions\. 

If you're using a construct that requires a role to function correctly, you can either pass in an existing role when instantiating the construct object, or let the construct create a new role for you, trusting the appropriate service principal\. The following example uses such a construct: a CodeBuild project\.

------
#### [ TypeScript ]

```
import * as codebuild from '@aws-cdk/aws-codebuild';

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
const codebuild = require('@aws-cdk/aws-codebuild');

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

Once the object is created, the role \(whether the role passed in or the default one created by the construct\) is available as the property `role`\. This property is not available on imported resources, however, so such constructs have an `addToRolePolicy` \(Python: `add_to_role_policy`\) method that does nothing if the construct is an imported resource, and calls the `addToPolicy` \(Python: `add_to_policy`\) method of the `role` property otherwise, saving you the trouble of handling the undefined case explicitly\. The following example demonstrates:

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

A few resources in AWS, such as Amazon S3 buckets and IAM roles, also have a resource policy\. These constructs have an `addToResourcePolicy` method \(Python: `add_to_resource_policy`\), which takes a `[PolicyStatement](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-iam.PolicyStatement.html)` as its argument\. Every policy statement added to a resource policy must specify at least one principal\.

In the following example, the [Amazon S3 bucket](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-s3.Bucket.html) `bucket` grants a role with the `s3:SomeAction` permission to itself\.

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