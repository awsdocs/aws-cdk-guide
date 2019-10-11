# Environments<a name="environments"></a>

Each `Stack` instance in your AWS CDK app is explicitly or implicitly associated with an environment \(`env`\)\. An environment is the target AWS account and AWS Region into which this stack needs to be deployed\.

By default, if you don't specify an environment when you define a stack, the stack is said to be environment agnostic\. This means that AWS CloudFormation templates which are synthesized from this stack will try to use deploy\-time resolution on environment\-related attributes such as `stack.account`, `stack.region`, and `stack.availablityZones`\. When using cdk deploy to deploy environment\-agnostic stacks, the CLI uses the default CLI environment configuration to determine where to deploy this stack\. The AWS CDK CLI follows a protocol similar to the AWS CLI for determining which AWS credentials to use when performing operations against your AWS account\. See [AWS CDK Command Line Interface \(cdk\)](tools.md#cli) for details\.

For production systems, we recommend that you explicitly specify the environment for each stack in your app using the `env` property\. The following example specifies two environments used in two different stacks\.

```
const envEU  = { account: '2383838383', region: 'eu-west-1' };
const envUSA = { account: '8373873873', region: 'us-west-2' };

new MyFirstStack(this, 'first-stack-us', { env: envUSA, encryption: false });
new MyFirstStack(this, 'first-stack-eu', { env: envEU, encryption: true  });
```

By explicitly specifying the target account and Region, the AWS CDK CLI ensures that you only deploy this stack to the desired target and deployment if the configured credentials are not aligned\. To deploy stacks to multiple accounts or different Regions, you must set the correct credentials or Region each time you run a cdk deploy *STACK* command\.

You can use the `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION` environment variables to explicitly associate a stack with the default CLI environment, as shown in the following example\. This is a common practice for development stacks that you want to include in your app, and have each developer use their own account\.

```
new MyDevStack(this, 'dev', { 
  env: { 
    account: process.env.CDK_DEFAULT_ACCOUNT, 
    region: process.env.CDK_DEFAULT_REGION 
  }
});
```

**Note**  
There is a distinction between not specifying the `env` property at all and specifying `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`\. The former implies that the stack should synthesize an environment\-agnostic template\. This means that constructs that are defined in such a stack cannot make assumptions about their environment\. For example, you can't write code like `if (stack.region === 'us-east-1')` or use framework facilities like [Vpc\.fromLookup](https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-ec2/vpc.html#aws_ec2_Vpc_fromLookup), which need to query your AWS account\. When specifying `CDK_DEFAULT_ACCOUNT` and `CDK_DEFAULT_REGION`, you tell the stack that it will be deployed in the account and Region as configured in the CLI at the time of synthesis\. This means that the synthesized template could be different on different machines, users, or sessions\. This might be acceptable for use cases like development stacks, but would be an anti\-pattern for production stacks\.