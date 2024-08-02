# Parameters and the AWS CDK<a name="parameters"></a>

*Parameters* are custom values that are supplied at deployment time\. [Parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html) are a feature of AWS CloudFormation\. Since the AWS Cloud Development Kit \(AWS CDK\) synthesizes AWS CloudFormation templates, it also offers support for deployment\-time parameters\.

## About parameters<a name="parameters-about"></a>

Using the AWS CDK, you can define parameters, which can then be used in the properties of constructs you create\. You can also deploy stacks that contain parameters\. 

When deploying the AWS CloudFormation template using the AWS CDK CLI, you provide the parameter values on the command line\. If you deploy the template through the AWS CloudFormation console, you are prompted for the parameter values\.

In general, we recommend against using AWS CloudFormation parameters with the AWS CDK\. The usual ways to pass values into AWS CDK apps are [context values](context.md) and environment variables\. Because they are not available at synthesis time, parameter values cannot be easily used for flow control and other purposes in your CDK app\.

**Note**  
To do control flow with parameters, you can use [https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnCondition.html](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.CfnCondition.html) constructs, although this is awkward compared to native `if` statements\.

Using parameters requires you to be mindful of how the code you're writing behaves at deployment time, and also at synthesis time\. This makes it harder to understand and reason about your AWS CDK application, in many cases for little benefit\.

Generally, it's better to have your CDK app accept necessary information in a well\-defined way and use it directly to declare constructs in your CDK app\. An ideal AWS CDK–generated AWS CloudFormation template is concrete, with no values remaining to be specified at deployment time\. 

There are, however, use cases to which AWS CloudFormation parameters are uniquely suited\. If you have separate teams defining and deploying infrastructure, for example, you can use parameters to make the generated templates more widely useful\. Also, because the AWS CDK supports AWS CloudFormation parameters, you can use the AWS CDK with AWS services that use AWS CloudFormation templates \(such as Service Catalog\)\. These AWS services use parameters to configure the template that's being deployed\.

## Learn more<a name="parameters-learn"></a>

For instructions on developing CDK apps with parameters, see [Use CloudFormation parameters to get a CloudFormation value](get_cfn_param.md)\.