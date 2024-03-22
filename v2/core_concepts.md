# AWS CDK concepts<a name="core_concepts"></a>

Learn core concepts behind the AWS Cloud Development Kit \(AWS CDK\)\.

## AWS CDK and IaC<a name="concepts-iac"></a>

The AWS CDK is an open\-source framework that you can use to manage your AWS infrastructure using code\. This approach is known as *infrastructure as code \(IaC\)*\. By managing and provisioning your infrastructure as code, you treat your infrastructure in the same way that developers treat code\. This provides many benefits, such as version control and scalability\. To learn more about IaC, see [What is Infrastructure as Code?](https://aws.amazon.com/what-is/iac/)

## AWS CDK and AWS CloudFormation<a name="concepts-cfn"></a>

The AWS CDK is tightly integrated with AWS CloudFormation\. AWS CloudFormation is a fully managed service that you can use to manage and provision your infrastructure on AWS\. With AWS CloudFormation, you define your infrastructure in templates and deploy them to AWS CloudFormation\. The AWS CloudFormation service then provisions your infrastructure according to the configuration defined on your templates\.

AWS CloudFormation templates are *declarative*, meaning they declare the desired state or outcome of your infrastructure\. Using JSON or YAML, you declare your AWS infrastructure by defining AWS *resources* and *properties*\. Resources represent the many services on AWS and properties represent your desired configuration of those services\. When you deploy your template to AWS CloudFormation, your resources and their configured properties are provisioned as described on your template\.

With the AWS CDK, you can manage your infrastructure *imperatively*, using general programming languages\. Instead of just defining a desired state declaratively, you can define the logic or sequence necessary to reach the desired state\. For example, you can use `if` statements or conditional loops that determine how to reach a desired end state for your infrastructure\.

Infrastructure created with the AWS CDK is eventually translated, or *synthesized* into AWS CloudFormation templates and deployed using the AWS CloudFormation service\. So while the AWS CDK offers a different approach to creating your infrastructure, you still receive the benefits of AWS CloudFormation, such as extensive AWS resource configuration support and robust deployment processes\.

To learn more about AWS CloudFormation, see [ What is AWS CloudFormation?](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) in the *AWS CloudFormation User Guide*\.

## AWS CDK and abstractions<a name="concepts-abstractions"></a>

With AWS CloudFormation, you must define every detail of how your resources are configured\. This provides the benefit of having complete control over your infrastructure\. However, this requires you to learn, understand, and create robust templates that contain resource configuration details and relationships between resources, such as permissions and event\-driven interactions\.

With the AWS CDK, you can have the same control over your resource configurations\. However, the AWS CDK also offers powerful abstractions, which can speed up and simplify the infrastructure development process\. For example, the AWS CDK includes constructs that provide sensible default configurations and helper methods that generate boilerplate code for you\. The AWS CDK also offers tools, such as the AWS CDK Command Line Interface \(AWS CDK CLI\), that perform infrastructure management actions for you\.

## Learn more about core AWS CDK concepts<a name="concepts-learn"></a>

### Interacting with the AWS CDK<a name="concepts-learn-interact"></a>

When using with the AWS CDK, you will primarily interact with the AWS Construct Library and the AWS CDK CLI\.

### Developing with the AWS CDK<a name="concepts-learn-develop"></a>

The AWS CDK can be written in any [supported programming language](languages.md)\. You start with a [CDK project](projects.md), which contains a structure of folders and files, including [assets](assets.md)\. Within the project, you create a [CDK application](apps.md)\. Within the app, you define a [stack](stacks.md), which directly represents a CloudFormation stack\. Within the stack, you define your AWS resources and properties using [constructs](constructs.md)\.

### Deploying with the AWS CDK<a name="concepts-learn-deploy"></a>

You deploy CDK apps into an AWS [environment](environments.md)\. Before deploying, you must perform a one\-time [bootstrapping](bootstrapping.md) to prepare your environment\.

### Learn more<a name="concepts-learn-more"></a>

To learn more about AWS CDK core concepts, see the topics in this section\.