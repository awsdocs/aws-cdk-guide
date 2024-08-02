# Environments for the AWS CDK<a name="environments"></a>

An environment consists of the AWS account and AWS Region that you deploy an AWS Cloud Development Kit \(AWS CDK\) stack to\.

**AWS account**  
When you create an AWS account, you receive an account ID\. This ID is a 12\-digit number, such as **012345678901**, that uniquely identifies your account\. To learn more, see [View AWS account identifiers](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-identifiers.html) in the *AWS Account Management Reference Guide*\.

**AWS Region**  
AWS Regions are named by using a combination of geographical location and a number that represents an Availability Zone in the Region\. For example, **us\-east\-1** represents an Availability Zone in the US East \(N\. Virginia\) Region\. To learn more about AWS Regions, see [Regions and Availability Zones](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/)\. For a list of Region codes, see [Regional endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#regional-endpoints) in the *AWS General Reference* Reference Guide\.

The AWS CDK can determine environments from your credentials and configuration files\. These files can be created and managed with the AWS Command Line Interface \(AWS CLI\)\. The following is a basic example of these files:

**Credentials file**

```
[default]
aws_access_key_id=ASIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
aws_session_token = IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE

[user1]
aws_access_key_id=ASIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
aws_session_token = fcZib3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZ2luX2IQoJb3JpZVERYLONGSTRINGEXAMPLE
```

**Configuration file**

```
[default]
region=us-west-2
output=json

[profile user1]
region=us-east-1
output=text
```

You can pass environment information from these files in your CDK code through environment variables that are provided by the CDK\. When you run a CDK CLI command, such as `cdk deploy`, you then provide the profile from your credentials and configuration files to gather environment information from\.

The following is an example of specifying these environment variables in your CDK code:

```
new MyDevStack(app, 'dev', {
  env: {
    account: process.env.CDK_DEFAULT_ACCOUNT,
    region: process.env.CDK_DEFAULT_REGION
}});
```

The following is an example of passing values associated with the `user1` profile from your credentials and configuration files to the CDK CLI using the `--profile` option\. Values from these files will be passed to your environment variables:

```
$ cdk deploy myStack --profile user1
```

Instead of using values from the credentials and configuration files, you can also hard\-code environment values in your CDK code\. The following is an example:

```
const envEU = { account: '238383838383', region: 'eu-west-1' };
const envUSA = { account: '837873873873', region: 'us-west-2' };

new MyFirstStack(app, 'first-stack-us', { env: envUSA });
new MyFirstStack(app, 'first-stack-eu', { env: envEU });
```

## Learn more<a name="environments-learn"></a>

To get started with using environments with the AWS CDK, see [Configure environments to use with the AWS CDK](configure-env.md)\.