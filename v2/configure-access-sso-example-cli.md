# Example: Authenticate with IAM Identity Center automatic token refresh for use with the AWS CDK CLI<a name="configure-access-sso-example-cli"></a>

In this example, we configure the AWS Command Line Interface \(AWS CLI\) to authenticate our user with the AWS IAM Identity Center token provider configuration\. The SSO token provider configuration lets the AWS CLI automatically retrieve refreshed authentication tokens to generate short\-term credentials that we can use with the AWS Cloud Development Kit \(AWS CDK\) Command Line Interface \(AWS CDK CLI\)\.

**Topics**
+ [Prerequisites](#configure-access-sso-example-cli-prerequisites)
+ [Step 1: Configure the AWS CLI](#configure-access-sso-example-cli-configure)
+ [Step 2: Use the AWS CLI to generate security credentials](#configure-access-sso-example-cli-credentials)
+ [Step 3: Use the CDK CLI](#configure-access-sso-example-cli-cdk)

## Prerequisites<a name="configure-access-sso-example-cli-prerequisites"></a>

This example assumes that the following prerequisites have been completed:
+ Prerequisites required to get set up with AWS and install our starting CLI tools\. For more information, see [Prerequisites](configure-access.md#configure-access-prerequisites)\.
+ IAM Identity Center has been set up by our organization as the method of managing users\.
+ At least one user has been created in IAM Identity Center\.

## Step 1: Configure the AWS CLI<a name="configure-access-sso-example-cli-configure"></a>

For detailed instructions on this step, see [Configure the AWS CLI to use IAM Identity Center token provider credentials with automatic authentication refresh](https://docs.aws.amazon.com/cli/latest/userguide/sso-configure-profile-token.html) in the *AWS Command Line Interface User Guide*\.

We sign in to the AWS access portal provided by our organization to gather our IAM Identity Center information\. This includes the **SSO start URL** and **SSO Region**\.

Next, we use the AWS CLI `aws configure sso` command to configure an IAM Identity Center profile and `sso-session` on our local machine:

```
$ aws configure sso
SSO session name (Recommended): my-sso
SSO start URL [None]: https://my-sso-portal.awsapps.com/start
SSO region [None]: us-east-1
SSO registration scopes [sso:account:access]: <ENTER>
```

The AWS CLI attempts to open our default browser to begin the login process for our IAM Identity Center account\. If the AWS CLI is unable to open our browser, instructions are provided to manually start the login process\. This process associates the IAM Identity Center session with our current AWS CLI session\.

After establishing our session, the AWS CLI displays the AWS accounts available to us:

```
There are 2 AWS accounts available to you.
> DeveloperAccount, developer-account-admin@example.com (123456789011) 
  ProductionAccount, production-account-admin@example.com (123456789022)
```

We use the arrow keys to select our **DeveloperAccount**\.

Next, the AWS CLI displays the IAM roles available to us from our selected account:

```
Using the account ID 123456789011
There are 2 roles available to you.
> ReadOnly
  FullAccess
```

We use the arrow keys to select **FullAccess**\.

Next, the AWS CLI prompts us to complete configuration by specifying a default output format, default AWS Region, and name for our profile:

```
CLI default client Region [None]: us-west-2 <ENTER>>
CLI default output format [None]: json <ENTER>
CLI profile name [123456789011_FullAccess]: my-dev-profile <ENTER>
```

The AWS CLI displays a final message, showing how to use the named profile with the AWS CLI:

```
To use this profile, specify the profile name using --profile, as shown:

aws s3 ls --profile my-dev-profile
```

After completing this step, our `config` file will look like the following:

```
[profile my-dev-profile]
sso_session = my-sso
sso_account_id = 123456789011
sso_role_name = fullAccess
region = us-west-2
output = json
			
[sso-session my-sso]
sso_region = us-east-1
sso_start_url = https://my-sso-portal.awsapps.com/start
sso_registration_scopes = sso:account:access
```

We can now use this `sso-session` and named profile to request security credentials\.

## Step 2: Use the AWS CLI to generate security credentials<a name="configure-access-sso-example-cli-credentials"></a>

For detailed instructions on this step, see [Use an IAM Identity Center named profile](https://docs.aws.amazon.com/cli/latest/userguide/sso-using-profile.html) in the *AWS Command Line Interface User Guide*\.

We use the AWS CLI `aws sso login` command to request security credentials for our profile:

```
$ aws sso login --profile my-dev-profile
```

The AWS CLI attempts to open our default browser and verifies our IAM log in\. If we are not currently signed into IAM Identity Center, we will be prompted to complete the sign in process\. If the AWS CLI is unable to open our browser, instructions are provided to manually start the authorization process\.

After successfully logging in, the AWS CLI caches our IAM Identity Center session credentials\. These credentials include an expiration timestamp\. When they expire, the AWS CLI will request that we sign in to IAM Identity Center again\.

Using valid IAM Identity Center credentials, the AWS CLI securely retrieves AWS credentials for the IAM role specified in our profile\. From here, we can use the AWS CDK CLI with our credentials\.

## Step 3: Use the CDK CLI<a name="configure-access-sso-example-cli-cdk"></a>

With any CDK CLI command, we use the `\-\-profile` option to specify the named profile that we generated credentials for\. If our credentials are valid, the CDK CLI will successfully perform the command\. The following is an example:

```
$ cdk diff --profile my-dev-profile
Stack CdkAppStack
Hold on while we create a read-only change set to get a diff with accurate replacement information (use --no-change-set to use a less accurate but faster template-only diff)
Resources
[-] AWS::S3::Bucket amzn-s3-demo-bucket amzn-s3-demo-bucket5AF9C99B destroy

Outputs
[-] Output BucketRegion: {"Value":{"Ref":"AWS::Region"}}


✨  Number of stacks with differences: 1
```

When our credentials expire, an error message like the following will display:

```
$ cdk diff --profile my-dev-profile
Stack CdkAppStack

Unable to resolve AWS account to use. It must be either configured when you define your CDK Stack, or through the environment
```

To refresh our credentials, we use the AWS CLI `aws sso login` command:

```
$ aws sso login --profile my-dev-profile
```