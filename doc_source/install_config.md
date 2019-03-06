--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Installing and Configuring the AWS CDK<a name="install_config"></a>

This topic describes how to install the AWS CDK\.

## Prerequisites<a name="install_config_prerequisites"></a>

**CDK command line tools**  
+ [Node\.js \(>= 8\.11\.x\)](https://nodejs.org/en/download)
+ You must specify both your credentials and an AWS Region to use the CDK Toolkit;, as described in [Specifying Your Credentials](#credentials)\.

------
#### [ TypeScript ]

TypeScript >= 2\.7

------
#### [ JavaScript ]

TypeScript >= 2\.7

------
#### [ Java ]
+ Maven 3\.5\.4 and Java 8
+ Set the `JAVA_HOME` environment variable to the path to where you have installed the JDK on your machine

------
#### [ C\# ]

TBD

------

## Installing the CDK<a name="install_config_install"></a>

Install the CDK using the following command\.

```
npm install -g aws-cdk
```

Run the following command to see the version number of the CDK\.

```
cdk --version
```

## Updating Your Language Dependencies<a name="update"></a>

If you get an error message that your language framework is out of date, use one of the following commands to update the components that the CDK needs to support the lanuage\.

------
#### [ TypeScript ]

```
npx npm-check-updates -u
```

------
#### [ JavaScript ]

```
npx npm-check-updates -u
```

------
#### [ Java ]

```
mvn versions:use-latest-versions
```

------
#### [ C\# ]

```
nuget update
```

------

## Specifying Your Credentials<a name="credentials"></a>

You must specify your default credentials and AWS Region to use the CDK Toolkit\. You can do that in the following ways:
+ Using the AWS Command Line Interface \(AWS CLI\)\. This is the method we recommend\.
+ Using environment variables\.
+ Using the \-\-profile option to cdk commands\.

### Using the AWS CLI to Specify Credentials<a name="credentials_config"></a>

Use the [AWS Command Line Interface](https://docs.aws.amazon.com/cli/latest/userguide) aws configure command to specify your default credentials and Region\.

### Using Environment Variables to Specify Credentials<a name="credentials_env_vars"></a>

You can also set environment variables for your default credentials and Region\. Environment variables take precedence over settings in the credentials or config file\.
+ `AWS_ACCESS_KEY_ID` – Specifies your access key\.
+ `AWS_SECRET_ACCESS_KEY` – Specifies your secret access key\.
+ `AWS_DEFAULT_REGION` – Specifies your default Region\.

For example, to set the default Region to `us-east-2` on Linux or macOS:

```
export AWS_DEFAULT_REGION=us-east-2
```

To do the same on Windows:

```
set AWS_DEFAULT_REGION=us-east-2
```

See [Environment Variables](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html) in the *AWS Command Line Interface User Guide* for details\.

### Using the \-\-profile Option to Specify Credentials<a name="credentials_profile"></a>

Use the \-\-profile *PROFILE* option to a cdk command to the alternative profile *PROFILE* when executing the command\.

For example, if the `~/.aws/credentials` \(Linux or Mac\) or `%USERPROFILE%\.aws\credentials` \(Windows\) file contains the following profiles:

```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
        
[test]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

You can deploy your app using the **test** profile with the following command\.

```
cdk deploy --profile test
```

See [Named Profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) in the AWS CLI documentation for details\.