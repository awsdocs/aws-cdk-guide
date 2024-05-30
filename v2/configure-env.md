# Configure environments to use with the AWS CDK<a name="configure-env"></a>

You can configure AWS environments in multiple ways to use with the AWS Cloud Development Kit \(AWS CDK\)\. The best method of managing AWS environments will vary, based on your specific needs\.

Each CDK stack in your application must eventually be associated with an environment to determine where the stack gets deployed to\.

For an introduction to AWS environments, see [Environments](environments.md)\.

**Topics**
+ [Where you can specify environments from](#configure-env-where)
+ [Environment precedence with the AWS CDK](#configure-env-precedence)
+ [When to specify environments](#configure-env-when)
+ [How to specify environments with the AWS CDK](#configure-env-how)
+ [Considerations when configuring environments with the AWS CDK](#configure-env-considerations)
+ [Examples](#configure-env-examples)

## Where you can specify environments from<a name="configure-env-where"></a>

You can specify environments in credentials and configuration files, or by using the `[env](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#env)` property of the `Stack` construct from the AWS Construct Library\.

### Credentials and configuration files<a name="configure-env-where-files"></a>

You can use the AWS Command Line Interface \(AWS CLI\) to create `credentials` and `config` files that store, organize, and manage your AWS environment information\. To learn more about these files, see [Configuration and credential file settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) in the *AWS Command Line Interface User Guide*\.

Values stored in these files are organized by *profiles*\. How you name your profiles and the key\-value pairs in these files will vary based on your method of configuring programmatic access\. To learn more about the different methods, see [Configure security credentials for the AWS CDK CLI](configure-access.md)\.

In general, the AWS CDK resolves AWS account information from your `credentials` file and AWS Region information from your `config` file\.

Once you have your `credentials` and `config` files configured, you can specify the environment to use with the AWS CDK CLI and through environment variables\.

### env property of the Stack construct<a name="configure-env-where-env"></a>

You can specify the environment for each stack by using the `[env](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.Stack.html#env)` property of the `Stack` construct\. This property defines an account and Region to use\. You can pass hard\-coded values to this property or pass environment variables that are offered by the CDK\.

To pass environment variables, use the `AWS_DEFAULT_ACCOUNT` and `AWS_DEFAULT_REGION` environment variables\. These environment variables can pass values from your `credentials` and `config` files\. You can also use logic within your CDK code to determine the values of these environment variables\.

## Environment precedence with the AWS CDK<a name="configure-env-precedence"></a>

If you use multiple methods of specifying environments, the AWS CDK adheres to the following precedence:

1. Hard\-coded values specified with the `env` property of the `Stack` construct\.

1. `AWS_DEFAULT_ACCOUNT` and `AWS_DEFAULT_REGION` environment variables specified with the `env` property of the `Stack` construct\.

1. Environment information associated with the profile from your `credentials` and `config` files and passed to the CDK CLI using the `--profile` option\.

1. The `default` profile from your `credentials` and `config` files\.

## When to specify environments<a name="configure-env-when"></a>

When you develop with the CDK, you start by defining CDK stacks, which contain constructs that represent AWS resources\. Next, you synthesize each CDK stack into an AWS CloudFormation template\. You then deploy the CloudFormation template to your environment\. How you specify environments determines when your environment information gets applied and can affect CDK behavior and outcomes\.

### Specify environments at template synthesis<a name="configure-env-when-synth"></a>

When you specify environment information using the `env` property of the `Stack` construct, your environment information is applied at template synthesis\. Running `cdk synth` or `cdk deploy` produces an environment\-specific CloudFormation template\.

If you use environment variables within the `env` property, you must use the `--profile` option with CDK CLI commands to pass in the profile containing your environment information from your credentials and configuration files\. This information will then be applied at template synthesis to produce an environment\-specific template\.

Environment information within the CloudFormation template takes precedence over other methods\. For example, if you provide a different environment with `cdk deploy --profile profile`, the profile will be ignored\.

When you provide environment information in this way, you can use environment\-dependent code and logic within your CDK app\. This also means that the synthesized template could be different, based on the machine, user, or session that it’s synthesized under\. This approach is often acceptable or desirable during development, but is not recommended for production use\.

### Specify environments at stack deployment<a name="configure-env-when-deploy"></a>

If you don't specify an environment using the `env` property of the `Stack` construct, the CDK CLI will produce an environment\-agnostic CloudFormation template at synthesis\. You can then specify the environment to deploy to by using `cdk deploy --profile profile`\.

If you don't specify a profile when deploying an environment\-agnostic template, the CDK CLI will attempt to use environment values from the `default` profile of your `credentials` and `config` files at deployment\.

If environment information is not available at deployment, AWS CloudFormation will attempt to resolve environment information at deployment through environment\-related attributes such as `stack.account`, `stack.region`, and `stack.availabilityZones`\.

For environment\-agnostic stacks, constructs within the stack cannot use environment information and you cannot use logic that requires environment information\. For example, you cannot write code like `if (stack.region ==== 'us-east-1')` or use construct methods that require environment information such as `[Vpc\.fromLookup](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ec2.Vpc.html#static-fromwbrlookupscope-id-options)`\. To use these features, you must specify an environment with the `env` property\.

For environment\-agnostic stacks, any construct that uses Availability Zones will see two Availability Zones, allowing the stack to be deployed to any Region\.

## How to specify environments with the AWS CDK<a name="configure-env-how"></a>

### Specify hard\-coded environments for each stack<a name="configure-env-how-hard-coded"></a>

Use the `env` property of the `Stack` construct to specify AWS environment values for your stack\. The following is an example:

------
#### [ TypeScript ]

```
const envEU  = { account: '2383838383', region: 'eu-west-1' };
const envUSA = { account: '8373873873', region: 'us-west-2' };

new MyFirstStack(app, 'first-stack-us', { env: envUSA });
new MyFirstStack(app, 'first-stack-eu', { env: envEU });
```

------
#### [ JavaScript ]

```
const envEU  = { account: '2383838383', region: 'eu-west-1' };
const envUSA = { account: '8373873873', region: 'us-west-2' };

new MyFirstStack(app, 'first-stack-us', { env: envUSA });
new MyFirstStack(app, 'first-stack-eu', { env: envEU });
```

------
#### [ Python ]

```
env_EU = cdk.Environment(account="8373873873", region="eu-west-1")
env_USA = cdk.Environment(account="2383838383", region="us-west-2")

MyFirstStack(app, "first-stack-us", env=env_USA)
MyFirstStack(app, "first-stack-eu", env=env_EU)
```

------
#### [ Java ]

```
public class MyApp {

    // Helper method to build an environment
    static Environment makeEnv(String account, String region) {
        return Environment.builder()
                .account(account)
                .region(region)
                .build();
    }

    public static void main(final String argv[]) {
        App app = new App();

        Environment envEU = makeEnv("8373873873", "eu-west-1");
        Environment envUSA = makeEnv("2383838383", "us-west-2");
        
        new MyFirstStack(app, "first-stack-us", StackProps.builder()
                .env(envUSA).build());
        new MyFirstStack(app, "first-stack-eu", StackProps.builder()
                .env(envEU).build());
 
        app.synth();
    }
}
```

------
#### [ C\# ]

```
Amazon.CDK.Environment makeEnv(string account, string region)
{
    return new Amazon.CDK.Environment
    {
        Account = account,
        Region = region
    };
}

var envEU = makeEnv(account: "8373873873", region: "eu-west-1");
var envUSA = makeEnv(account: "2383838383", region: "us-west-2");

new MyFirstStack(app, "first-stack-us", new StackProps { Env=envUSA });
new MyFirstStack(app, "first-stack-eu", new StackProps { Env=envEU });
```

------
#### [ Go ]

```
	envEU := &awscdk.Environment{
		Account: "2383838383",
		Region:  "eu-west-1",
	}

	envUSA := &awscdk.Environment{
		Account: "8373873873",
		Region:  "us-west-2",
	}

	NewMyFirstStack(app, "first-stack-us", &MyFirstStackProps{
		StackProps: awscdk.StackProps{
			Env: envUSA,
		},
	})

	NewMyFirstStack(app, "first-stack-eu", &MyFirstStackProps{
		StackProps: awscdk.StackProps{
			Env: envEU,
		},
	})
```

------

We recommend this approach for production environments\. By explicitly specifying the environment in this way, you can ensure that the stack is always deployed to the specific environment\.

### Specify environments using environment variables<a name="configure-env-how-env"></a>

The AWS CDK provides two environment variables that you can use within your CDK code: `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`\. When you use these environment variables within the `env` property of your stack instance, you can pass environment information from your credentials and configuration files using the CDK CLI `--profile` option\.

The following is an example of how to specify these environment variables:

------
#### [ TypeScript ]

Access environment variables via Node's `process` object\.

**Note**  
 You need the `DefinitelyTyped` module to use `process` in TypeScript\. `cdk init` installs this module for you\. However, you should install this module manually if you are working with a project created before it was added, or if you didn't set up your project using `cdk init`\.  

```
npm install @types/node
```

```
new MyDevStack(app, 'dev', { 
  env: { 
    account: process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEFAULT_REGION 
}});
```

------
#### [ JavaScript ]

Access environment variables via Node's `process` object\. 

```
new MyDevStack(app, 'dev', { 
  env: { 
    account: process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEFAULT_REGION 
}});
```

------
#### [ Python ]

Use the `os` module's `environ` dictionary to access environment variables\.

```
import os
MyDevStack(app, "dev", env=cdk.Environment(
    account=os.environ["CDK_DEFAULT_ACCOUNT"],
    region=os.environ["CDK_DEFAULT_REGION"]))
```

------
#### [ Java ]

Use `System.getenv()` to get the value of an environment variable\.

```
public class MyApp {

    // Helper method to build an environment
    static Environment makeEnv(String account, String region) {
        account = (account == null) ? System.getenv("CDK_DEFAULT_ACCOUNT") : account;
        region = (region == null) ? System.getenv("CDK_DEFAULT_REGION") : region;

        return Environment.builder()
                .account(account)
                .region(region)
                .build();
    }

    public static void main(final String argv[]) {
        App app = new App();

        Environment envEU = makeEnv(null, null);
        Environment envUSA = makeEnv(null, null);
        
        new MyDevStack(app, "first-stack-us", StackProps.builder()
                .env(envUSA).build());
        new MyDevStack(app, "first-stack-eu", StackProps.builder()
                .env(envEU).build());
 
        app.synth();
    }
}
```

------
#### [ C\# ]

Use `System.Environment.GetEnvironmentVariable()` to get the value of an environment variable\.

```
Amazon.CDK.Environment makeEnv(string account=null, string region=null)
{
    return new Amazon.CDK.Environment
    {
        Account = account ?? System.Environment.GetEnvironmentVariable("CDK_DEFAULT_ACCOUNT"),
        Region = region ?? System.Environment.GetEnvironmentVariable("CDK_DEFAULT_REGION")
    };
}

new MyDevStack(app, "dev", new StackProps { Env = makeEnv() });
```

------
#### [ Go ]

```
	devEnv := &awscdk.Environment{
		Account: os.Getenv("CDK_DEFAULT_ACCOUNT"),
		Region:  os.Getenv("CDK_DEFAULT_REGION"),
	}

	NewMyDevStack(app, "dev", &MyDevStackProps{
		StackProps: awscdk.StackProps{
			Env: devEnv,
		},
	})
```

------

By specifying environments using environment variables, you can have the same CDK stack synthesize to AWS CloudFormation templates for different environments\. This means that you can deploy the same CDK stack to different AWS environments without having to modify your CDK code\. You only have to specify the profile to use when running `cdk synth`\.

This approach is great for development environments when deploying the same stack to different environments\. However, we do not recommend this approach for production environments since the same CDK code can synthesize different templates, depending on the machine, user, or session that it's synthesized under\.

### Specify environments from your credentials and configuration files with the CDK CLI<a name="configure-env-how-files"></a>

When deploying an environment\-agnostic template, use the `--profile` option with any CDK CLI command to specify the profile to use\. The following is an example that deploys a CDK stack named `myStack` using the `prod` profile that is defined in the `credentials` and `config` files:

```
$ cdk deploy myStack --profile prod
```

For more information on the `--profile` option, along with other CDK CLI commands and options, see [AWS CDK CLI command reference](ref-cli-cmd.md)\.

## Considerations when configuring environments with the AWS CDK<a name="configure-env-considerations"></a>

Services that you define by using constructs within your stacks must support the Region that you are deploying to\. For a list of supported AWS services per region, see [AWS Services by Region](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)\.

You must have valid AWS Identity and Access Management \(IAM\) credentials to perform stack deployments with the AWS CDK into your specified environments\.

## Examples<a name="configure-env-examples"></a>

### Synthesize an environment–agnostic CloudFormation template from a CDK stack<a name="configure-env-examples-agnostic"></a>

In this example, we create an environment\-agnostic CloudFormation template from our CDK stack\. We can then deploy this template to any environment\.

The following is our example CDK stack\. This stack defines an Amazon S3 bucket and a CloudFormation stack output for the bucket’s Region\. For this example, `env` is not defined:

------
#### [ TypeScript ]

```
export class CdkAppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create the S3 bucket
    const bucket = new s3.Bucket(this, 'myBucket', {
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });
    
    // Create an output for the bucket's Region
    new cdk.CfnOutput(this, 'BucketRegion', {
      value: bucket.env.region,
    });
  }
}
```

------
#### [ JavaScript ]

```
class CdkAppStack extends cdk.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Create the S3 bucket
    const bucket = new s3.Bucket(this, 'myBucket', {
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });
    
    // Create an output for the bucket's Region
    new cdk.CfnOutput(this, 'BucketRegion', {
      value: bucket.env.region,
    });
  }
}
```

------
#### [ Python ]

```
class CdkAppStack(cdk.Stack):

    def __init__(self, scope: cdk.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Create the S3 bucket
        bucket = s3.Bucket(self, 'myBucket',
            removal_policy=cdk.RemovalPolicy.DESTROY
        )
        
        # Create an output for the bucket's Region
        cdk.CfnOutput(self, 'BucketRegion',
            value=bucket.env.region
        )
```

------
#### [ Java ]

```
public class CdkAppStack extends Stack {

    public CdkAppStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        // Create the S3 bucket
        Bucket bucket = Bucket.Builder.create(this, "myBucket")
            .removalPolicy(RemovalPolicy.DESTROY)
            .build();
        
        // Create an output for the bucket's Region
        CfnOutput.Builder.create(this, "BucketRegion")
            .value(this.getRegion())
            .build();
    }
}
```

------
#### [ C\# ]

```
namespace MyCdkApp
{
    public class CdkAppStack : Stack
    {
        public CdkAppStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            // Create the S3 bucket
            var bucket = new Bucket(this, "myBucket", new BucketProps
            {
                RemovalPolicy = RemovalPolicy.DESTROY
            });
            
            // Create an output for the bucket's Region
            new CfnOutput(this, "BucketRegion", new CfnOutputProps
            {
                Value = this.Region
            });
        }
    }
}
```

------
#### [ Go ]

```
func NewCdkAppStack(scope constructs.Construct, id string, props *CdkAppStackProps) awscdk.Stack {
	stack := awscdk.NewStack(scope, &id, &props.StackProps)

	// Create the S3 bucket
	bucket := awss3.NewBucket(stack, jsii.String("myBucket"), &awss3.BucketProps{
		RemovalPolicy: awscdk.RemovalPolicy_DESTROY,
	})
	
	// Create an output for the bucket's Region
	awscdk.NewCfnOutput(stack, jsii.String("BucketRegion"), &awscdk.CfnOutputProps{
		Value: stack.Region(),
	})

	return stack
}
```

------

When we run `cdk synth`, the CDK CLI produces a CloudFormation template with the pseudo parameter `AWS::Region` as the output value for the bucket’s Region\. This parameter will be resolved at deployment:

```
Outputs:
  BucketRegion:
    Value:
      Ref: AWS::Region
```

To deploy this stack to an environment that is specified in the `dev` profile of our credentials and configuration files, we run the following:

```
$ cdk deploy CdkAppStack --profile dev
```

If we don't specify a profile, the CDK CLI will attempt to use environment information from the `default` profile in our credentials and configuration files\.

### Use logic to determine environment information at template synthesis<a name="configure-env-example-logic"></a>

In this example, we configure the `env` property of our `stack` instance to use a valid expression\. We specify two additional environment variables, `CDK_DEPLOY_ACCOUNT` and `CDK_DEPLOY_REGION`\. These environment variables can override defaults at synthesis time if they exist:

------
#### [ TypeScript ]

```
new MyDevStack(app, 'dev', { 
  env: { 
    account: process.env.CDK_DEPLOY_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEPLOY_REGION || process.env.CDK_DEFAULT_REGION 
}});
```

------
#### [ JavaScript ]

```
new MyDevStack(app, 'dev', { 
  env: { 
    account: process.env.CDK_DEPLOY_ACCOUNT || process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEPLOY_REGION || process.env.CDK_DEFAULT_REGION 
}});
```

------
#### [ Python ]

```
MyDevStack(app, "dev", env=cdk.Environment(
    account=os.environ.get("CDK_DEPLOY_ACCOUNT", os.environ["CDK_DEFAULT_ACCOUNT"]),
    region=os.environ.get("CDK_DEPLOY_REGION", os.environ["CDK_DEFAULT_REGION"])
```

------
#### [ Java ]

```
public class MyApp {

    // Helper method to build an environment
    static Environment makeEnv(String account, String region) {
        account = (account == null) ? System.getenv("CDK_DEPLOY_ACCOUNT") : account;
        region = (region == null) ? System.getenv("CDK_DEPLOY_REGION") : region;
        account = (account == null) ? System.getenv("CDK_DEFAULT_ACCOUNT") : account;
        region = (region == null) ? System.getenv("CDK_DEFAULT_REGION") : region;

        return Environment.builder()
                .account(account)
                .region(region)
                .build();
    }

    public static void main(final String argv[]) {
        App app = new App();

        Environment envEU = makeEnv(null, null);
        Environment envUSA = makeEnv(null, null);
        
        new MyDevStack(app, "first-stack-us", StackProps.builder()
                .env(envUSA).build());
        new MyDevStack(app, "first-stack-eu", StackProps.builder()
                .env(envEU).build());
 
        app.synth();
    }
}
```

------
#### [ C\# ]

```
Amazon.CDK.Environment makeEnv(string account=null, string region=null)
{
    return new Amazon.CDK.Environment
    {
        Account = account ??
            System.Environment.GetEnvironmentVariable("CDK_DEPLOY_ACCOUNT") ??
            System.Environment.GetEnvironmentVariable("CDK_DEFAULT_ACCOUNT"),
        Region = region ?? 
            System.Environment.GetEnvironmentVariable("CDK_DEPLOY_REGION") ??
            System.Environment.GetEnvironmentVariable("CDK_DEFAULT_REGION")
    };
}

new MyDevStack(app, "dev", new StackProps { Env = makeEnv() });
```

------
#### [ Go ]

```
app := awscdk.NewApp(nil)

	devEnv := &awscdk.Environment{
		Account: os.Getenv("CDK_DEPLOY_ACCOUNT"),
		Region:  os.Getenv("CDK_DEPLOY_REGION"),
	}

	if devEnv.Account == "" {
		devEnv.Account = os.Getenv("CDK_DEFAULT_ACCOUNT")
	}

	if devEnv.Region == "" {
		devEnv.Region = os.Getenv("CDK_DEFAULT_REGION")
	}

	NewMyDevStack(app, "dev", &MyDevStackProps{
		StackProps: awscdk.StackProps{
			Env: devEnv,
		},
	})
```

------

With our stack’s environment declared this way, we can then write a short script or batch file and set variables from command line arguments, then call `cdk deploy`\. The following is an example\. Any arguments beyond the first two are passed through to `cdk deploy` to specify command line options or arguments:

------
#### [ macOS/Linux ]

```
#!/usr/bin/env bash
if [[ $# -ge 2 ]]; then
    export CDK_DEPLOY_ACCOUNT=$1
    export CDK_DEPLOY_REGION=$2
    shift; shift
    npx cdk deploy "$@"
    exit $?
else
    echo 1>&2 "Provide account and region as first two args."
    echo 1>&2 "Additional args are passed through to cdk deploy."
    exit 1
fi
```

Save the script as `cdk-deploy-to.sh`, then execute `chmod +x cdk-deploy-to.sh` to make it executable\.

------
#### [ Windows ]

```
@findstr /B /V @ %~dpnx0 > %~dpn0.ps1 && powershell -ExecutionPolicy Bypass %~dpn0.ps1 %*
@exit /B %ERRORLEVEL%
if ($args.length -ge 2) {
    $env:CDK_DEPLOY_ACCOUNT, $args = $args
    $env:CDK_DEPLOY_REGION,  $args = $args
    npx cdk deploy $args
    exit $lastExitCode
} else {
    [console]::error.writeline("Provide account and region as first two args.")
    [console]::error.writeline("Additional args are passed through to cdk deploy.")
    exit 1
}
```

The Windows version of the script uses PowerShell to provide the same functionality as the macOS/Linux version\. It also contains instructions to allow it to be run as a batch file so it can be easily invoked from a command line\. It should be saved as `cdk-deploy-to.bat`\. The file `cdk-deploy-to.ps1` will be created when the batch file is invoked\.

------

We can then write additional scripts that use the `cdk-deploy-to` script to deploy to specific environments\. The following is an example:

------
#### [ macOS/Linux ]

```
#!/usr/bin/env bash
# cdk-deploy-to-test.sh
./cdk-deploy-to.sh 123457689 us-east-1 "$@"
```

------
#### [ Windows ]

```
@echo off
rem cdk-deploy-to-test.bat
cdk-deploy-to 135792469 us-east-1 %*
```

------

The following is an example that uses the `cdk-deploy-to` script to deploy to multiple environments\. If the first deployment fails, the process stops:

------
#### [ macOS/Linux ]

```
#!/usr/bin/env bash
# cdk-deploy-to-prod.sh
./cdk-deploy-to.sh 135792468 us-west-1 "$@" || exit
./cdk-deploy-to.sh 246813579 eu-west-1 "$@"
```

------
#### [ Windows ]

```
@echo off
rem cdk-deploy-to-prod.bat
cdk-deploy-to 135792469 us-west-1 %* || exit /B
cdk-deploy-to 245813579 eu-west-1 %*
```

------