--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Stack How\-Tos<a name="stack_how_to"></a>

This section contains some additional information about using stacks, including how to create a stack in a specific region with a specific account ID, and creating an app with multiple stacks\. For conceptual information about stacks, see [Stacks](apps_and_stacks.md#stacks)\.

## Create a Stack with an Account and Region<a name="stack_how_to_create_stack_with_account_region"></a>

The following example shows how to create a stack in `us-west-1` for the account with ID `11223344`\.

```
new HelloStack(this, 'hello-cdk-us-west-1', {
  env: {
    account: '11223344',
    region: 'us-west-1'
}});
```

## Create an App with Multiple Stacks<a name="stack_how_to_create_multiple_stacks"></a>

The following example shows one solution to parameterizing how you create a stack\. It creates one stack with a `t2.micro` AMI for development, and three stacks with the more expensive `c5.2xlarge` AMI\. The development stack and one of the production stacks use the same account to create the stack in `us-east-1`; the other two production stacks use different account IDs and regions\.

```
// lib/mystack.ts 
// Defines my parameterized application which can be deployed anywhere.
// If dev is true, the stack is used for development,
// so it does not require the more expensive AMI.
interface MyStackProps extends cdk.StackProps {
  dev: boolean; 
} 

class MyStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props: MyStackProps) { 
    super(scope, id, props); 
    
    const instanceType = props.dev 
      ? new ec.InstanceType('t2.micro') 
      : new ec.InstanceType('c5.2xlarge'); 
      
    const fleet new MyComputeFleet(this, 'Fleet', {
      instanceType, }); 
      
    const queue = new sqs.Queue(this, 'Queue'); 
    queue.grantConsumeMessages(fleet.role); 
  } 
} 

// bin/myapp.ts 
const app = new cdk.App(); 

// Specify where MyStack instances are deployed 
// and under what account.
const environments = [ 
  { account: '12345678', region: 'us-east-1' }, 
  { account: '87654321', region: 'eu-west-1' }, 
  { account: '12344321', region: 'ap-southeast-1' }, 
]; 

// Beta instance in the first environment 
new MyStack(app, 'BetaStack', { dev: true, env: environments[0] }); 

// Production instance in every other environment 
for (const env of environments) { 
  new MyStack(app, `ProdStack-${env.region}`, { dev: false, env }); 
}
```

The following example shows how to deploy a production stack using the `c5.2xlarge` AMI to the `us-east-1` region\.

```
cdk deploy ProdStack-us-east-1
```