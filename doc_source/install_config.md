--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Installing and Configuring the AWS CDK<a name="install_config"></a>

This topic describes how to install and configure the AWS CDK\.

## Prerequisites<a name="install_config_prerequisites"></a>

You must install [Node\.js \(>= 8\.11\.x\)](https://nodejs.org/en/download) to use the command\-line toolkit and language bindings\.

If you use Java, you must set the `JAVA_HOME` environment variable to the path to where you have installed the JDK on your machine to build an AWS CDK app in Java\.

Specify your credentials and region with the [AWS CLI](https://aws.amazon.com/cli)\. You must specify both your credentials and a region to use the toolkit\. See [Configuring the AWS CDK Toolkit](#credentials) for information on using the AWS CLI to specify your credentials\.

## Installing the AWS CDK<a name="install_config_install"></a>

Install the AWS CDK using the following command:

```
npm install -g aws-cdk
```

Run the following command to see the currently installed version of the AWS CDK:

```
cdk --version
```

## Configuring the AWS CDK Toolkit<a name="credentials"></a>

Use the AWS CDK toolkit to view the contents of an app\.

**Note**  
You must specify your default credentials and region to use the toolkit\.  
Use the AWS Command Line Interface aws configure command to specify your default credentials and region\.  
You can also set environment variables for your default credentials and region\. Environment variables take precedence over settings in the credentials or config file\.  
`AWS_ACCESS_KEY_ID` specifies your access key
`AWS_SECRET_ACCESS_KEY` specifies your secret access key
`AWS_DEFAULT_REGION` specifies your default region
See [Environment Variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html) in the CLI User Guide for details\.

**Note**  
The China regions \(cn\-north\-1 and cn\-northwest\-1\) do not support version reporting\.  
See [Version Reporting](tools.md#version_reporting) for details, including how to [opt\-out](tools.md#version_reporting_opt_out)\.