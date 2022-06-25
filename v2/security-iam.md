# Identity and access management for the AWS Cloud Development Kit \(AWS CDK\)<a name="security-iam"></a>

AWS Identity and Access Management \(IAM\) is an Amazon Web Services \(AWS\) service that helps an administrator securely control access to AWS resources\. IAM administrators control who can be *authenticated* \(signed in\) and *authorized* \(have permissions\) to use resources in AWS services\. IAM is an AWS service that you can use with no additional charge\.

To use the AWS CDK to access AWS, you need an AWS account and AWS credentials\. To increase the security of your AWS account, we recommend that you use an *IAM role* to provide access credentials instead of using an *IAM user* or your AWS account credentials\.

For details about working with IAM, see [AWS Identity and Access Management](https://aws.amazon.com/iam/)\.

For an overview of IAM users and why they are important for the security of your account, see [AWS Security Credentials](https://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html) in the [Amazon Web Services General Reference](https://docs.aws.amazon.com/general/latest/gr/)\.

The AWS CDK follows the [shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/) through the specific Amazon Web Services \(AWS\) services it supports\. For AWS service security information, see the [AWS service security documentation page](https://docs.aws.amazon.com/security/?id=docs_gateway#aws-security) and [AWS services that are in scope of AWS compliance efforts by compliance program](https://aws.amazon.com/compliance/services-in-scope/)\.

## CDK deployment minimum privileges<a name="security-iam-privileges"></a>

The [bootstrapping template](bootstrapping.md#bootstrapping-template) required by CDK v2 provision IAM roles that are used by CDK cli during `cdk diff` and `cdk deploy` actions during the different [phases of a CDK deployment](apps.md#lifecycle)\.

You need to grant your deployment roles permissions to assume the CDK roles adding the following to your roles policies if they don't have already wider permissions:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AssumeCDKRoles",
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Resource": [
                "arn:aws:iam::aws-account-id:role/cdk-*"
            ]
        }
    ]
}
```

Where *aws\-account\-id* is the AWS account ID where your CDK deployment and your roles are placed.
