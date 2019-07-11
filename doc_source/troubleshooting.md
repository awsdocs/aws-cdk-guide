# Troubleshooting the AWS CDK<a name="troubleshooting"></a>

This section describes how to troubleshoot problems with your AWS CDK app\.

## Inconsistent Module Versions<a name="troubleshooting_modules"></a>

If you have inconsistent module versions in your `package.json` file or `node_modules` directory, you might see error messages such as the following:

```
lib/my_ecs_construct-stack.ts:56:49 - error TS2345: Argument of type 'this' is not assignable to parameter of type 'Construct'.
  Type 'MyEcsConstructStack' is not assignable to type 'Construct'.
    Types of property 'node' are incompatible.
      Property 'root' is missing in type 'ConstructNode' but required in type 'ConstructNode'.
 
56     new ecs_patterns.LoadBalancedFargateService(this, "MyNewFargateService", {
                                                   ~~~~
 
  node_modules/@aws-cdk/aws-ecs-patterns/node_modules/@aws-cdk/cdk/lib/construct.d.ts:187:14
    187     readonly root: IConstruct;
                     ~~~~
    'root' is declared here.
```

The solution is to delete the `node_modules` directory and re\-install your AWS CDK modules:

```
rm -rf node_modules
npm install @aws-cdk/...
```