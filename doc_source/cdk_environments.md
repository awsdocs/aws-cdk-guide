--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Environments and Authentication<a name="cdk_environments"></a>

The AWS CDK refers to the combination of an account ID and a Region as an *environment*\. The simplest environment is the one you get by default, which is the one you get when you have set up your credentials and a default Region as described in [Configuring the AWS CDK Toolkit](cdk_install_config.md#cdk_credentials)\.

When you create a [Stack](https://awslabs.github.io/aws-cdk/refs/_aws-cdk_cdk.html#@aws-cdk/cdk.Stack) instance, you can supply the target deployment environment for the stack using the `env` property, as shown in the following example, where REGION is the Region in which you want to create the stack and ACCOUNT is your account ID\.

```
new MyStack(app, { env: { region: 'REGION', account: 'ACCOUNT' } });
```

For each of the two arguments, **region** and **account**, the AWS CDK uses the following lookup procedure:
+ If **region** or **account** are provided directly as an property to the Stack, use that\.
+ Otherwise, read **default\-account** and **default\-region** from the application's context\. These can be set in the AWS CDK Toolkit in either the local `cdk.json` file or the global version in *$HOME/\.cdk* on Linux or MacOS or *%USERPROFILE%\\\\\.cdk* on Windows\.
+ If these are not defined, it will determine them as follows:
  + **account**: use account from default SDK credentials\. Environment variables are tried first \(`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`\), followed by credentials in *$HOME/\.aws/credentials* on Linux or MacOS or *%USERPROFILE%\\\\\.aws\\\\credentials* on Windows\.
  + **region**: use the default region configured in *$HOME/\.aws/config* on Linux or MacOS or *%USERPROFILE%\\\\\.aws\\\\config* on Windows\.
  + You can set these defaults manually, but we recommend you use `aws configure`, as described in [Installing and Configuring the AWS CDK](cdk_install_config.md)

We recommend that you use the default environment for development stacks, and explicitly specify accounts and Regions for production stacks\.

**Note**  
Note that even though the region and account might explicitly be set on your Stack, if you run `cdk deploy`, the AWS CDK will still use the currently\-configured SDK credentials, as provided via the `AWS_` environment variables or `aws configure`\. This means that if you want to deploy stacks to multiple accounts, you will have to set the correct credentials for each invocation to `cdk deploy STACK`\.