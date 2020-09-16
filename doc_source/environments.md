# Environments<a name="environments"></a>

Each `Stack` instance in your AWS CDK app is explicitly or implicitly associated with an environment \(`env`\)\. An environment is the target AWS account and region into which the stack is intended to be deployed\.

**Note**  
For all but the simplest deployments, you will need to [bootstrap](bootstrapping.md) each environment you will deploy into\. Deployment requires certain AWS resources to be available, and these resources are provisined by bootstrapping\.

If you don't specify an environment when you define a stack, the stack is said to be *environment\-agnostic*\. AWS CloudFormation templates synthesized from such a stack will try to use deploy\-time resolution on environment\-related attributes such as `stack.account`, `stack.region`, and `stack.availablityZones` \(Python: `availability_zones`\)\.

In an environment\-agnostic stack, any constructs that use availability zones will see two of them\. This allows the stack to be deployed to almost any region, since nearly all regions have at least two availability zones\. The only exception is Osaka \(`ap-northeast-3`\), which has one\.

When using cdk deploy to deploy environment\-agnostic stacks, the AWS CDK CLI uses the specified AWS CLI profile \(or the default profile, if none is specified\) to determine where to deploy\. The AWS CDK CLI follows a protocol similar to the AWS CLI to determine which AWS credentials to use when performing operations against your AWS account\. See [AWS CDK Toolkit \(`cdk` command\)](cli.md) for details\.

For production stacks, we recommend that you explicitly specify the environment for each stack in your app using the `env` property\. The following example specifies different environments for its two different stacks\.

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
env_EU = core.Environment(account="8373873873", region="eu-west-1")
env_USA = core.Environment(account="2383838383", region="us-west-2")

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

When you hard\-code the target account and region as above, the stack will always be deployed to that specific account and region\. To make the stack deployable to a different target, but to determine the target at synthesis time, your stack can use two environment variables provided by the AWS CDK CLI: `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`\. These variables are set based on the AWS profile specified using the \-\-profile option, or the default AWS profile if you don't specify one\.

The following code fragment shows how to access the account and region passed from the AWS CDK CLI in your stack\.

------
#### [ TypeScript ]

Access environment variables via Node's `process` object\.

**Note**  
 You need the DefinitelyTyped module to use `process` in TypeScript\. `cdk init` installs this module for you, but if you are working with a project created before it was added, or didn't set up your project using `cdk init`, install it manually\.  

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
MyDevStack(app, "dev", env=core.Environment(
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

The AWS CDK distinguishes between not specifying the `env` property at all and specifying it using `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`\. The former implies that the stack should synthesize an environment\-agnostic template\. Constructs that are defined in such a stack cannot use any information about their environment\. For example, you can't write code like `if (stack.region === 'us-east-1')` or use framework facilities like [Vpc\.fromLookup](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-ec2.Vpc.html#static-from-wbr-lookupscope-id-options) \(Python: `from_lookup`\), which need to query your AWS account\. These features do not work at all without an explicit environment specified; to use them, you must specify `env`\.

When you pass in your environment using `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`, the stack will be deployed in the account and Region determined by the AWS CDK CLI at the time of synthesis\. This allows environment\-dependent code to work, but it also means that the synthesized template could be different based on the machine, user, or session under which it is synthesized\. This behavior is often acceptable or even desirable during development, but it would probably be an anti\-pattern for production use\.

You can set `env` however you like, using any valid expression\. For example, you might write your stack to support two additional environment variables to let you override the account and region at synthesis time\. We'll call these `CDK_DEPLOY_ACCOUNT` and `CDK_DEPLOY_REGION` here, but you could name them anything you like, as they are not set by the AWS CDK\. In the following stack's environment, we use our alternative environment variables if they're set, falling back to the default environment provided by the AWS CDK if they are not\.

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
MyDevStack(app, "dev", env=core.Environment(
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

With your stack's environment declared this way, you can now write a short script or batch file like the following to set the variables from command line arguments, then call `cdk deploy`\. Any arguments beyond the first two are passed through to `cdk deploy` and can be used to specify command\-line options or stacks\.

------
#### [ Mac OS X/Linux ]

```
#!/usr/bin/env bash
if [[ $# -ge 2 ]]; then
    export CDK_DEPLOY_ACCOUNT=$1
    export CDK_DEPLOY_REGION=$2
    shift; shift
    npx cdk deploy "$@"
    exit $?
else
    echo 1>&2 "Provide AWS account and region as first two args."
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
    [console]::error.writeline("Provide AWS account and region as first two args.")
    [console]::error.writeline("Additional args are passed through to cdk deploy.")
    exit 1
}
```

The Windows version of the script uses PowerShell to provide the same functionality as the Mac OS X/Linux version\. It also contains instructions to allow it to be run as a batch file so it can be easily invoked from a command line\. It should be saved as `cdk-deploy-to.bat`\. The file `cdk-deploy-to.ps1` will be created when the batch file is invoked\.

------

Then you can write additional scripts that call the "deploy\-to" script to deploy to specific environments \(even multiple environments per script\):

------
#### [ Mac OS X/Linux ]

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

When deploying to multiple environments, consider whether you want to continue deploying to other environments after a deployment fails\. The following example avoids deploying to the second production environment if the first doesn't succeed\.

------
#### [ Mac OS X/Linux ]

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

Developers could still use the normal `cdk deploy` command to deploy to their own AWS environments for development\.