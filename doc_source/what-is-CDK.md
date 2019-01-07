--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# What Is the AWS Cloud Development Kit?<a name="what-is-CDK"></a>

Welcome to the AWS Cloud Development Kit \(AWS CDK\) User Guide\.

The AWS CDK is an infrastructure modeling framework that allows you to define your cloud resources using an imperative programming interface\. *The AWS CDK is currently in developer preview\. We look forward to community feedback and collaboration*\.

## Why Should you use the AWS CDK?<a name="why_use_cdk"></a>

Perhaps the best reason is shown graphically\. Here is the TypeScript code in an AWS CDK project to create a Fargate service \(this is the code we use in the [Creating an Amazon ECS Fargate Service Using the AWS CDK](cdk_ecs_example.md)\):

```
export class MyEcsConstructStack extends cdk.Stack {
  constructor(parent: cdk.App, name: string, props?: cdk.StackProps) {
    super(parent, name, props);
     
    const vpc = new ec2.VpcNetwork(this, 'MyVpc', {
      maxAZs: 3 // Default is all AZs in region
    });
     
    const cluster = new ecs.Cluster(this, 'MyCluster', {
      vpc: vpc
    });
     
    // Create a load-balanced Fargate service and make it public
    new ecs.LoadBalancedFargateService(this, 'MyFargateService', {
      cluster: cluster,  // Required
      cpu: '512', // Default is 256
      desiredCount: 6,  // Default is 1
      image: ecs.ContainerImage.fromDockerHub('amazon/amazon-ecs-sample'), // Required
      memoryMiB: '2048',  // Default is 512
      publicLoadBalancer: true  // Default is false
    });
  }
}
```

This produces a AWS CloudFormation template of over 600 lines\. We'll show the first and last 25:

```
Resources:
  MyVpcF9F0CA6F:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: MyEcsConstructStack/MyVpc
    Metadata:
      aws:cdk:path: MyEcsConstructStack/MyVpc/Resource
  MyVpcPublicSubnet1SubnetF6608456:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.0.0/19
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-east-2a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: MyEcsConstructStack/MyVpc/PublicSubnet1
        - Key: aws-cdk:subnet-name
...
    Metadata:
      aws:cdk:path: MyEcsConstructStack/MyFargateService/LB/PublicListener/ECSGroup/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: "@aws-cdk/assets=0.19.0,@aws-cdk/assets-docker=0.19.0,@aws-cdk/aws-appl\
        icationautoscaling=0.19.0,@aws-cdk/aws-autoscaling=0.19.0,@aws-cdk/aws-\
        autoscaling-common=0.19.0,@aws-cdk/aws-certificatemanager=0.19.0,@aws-c\
        dk/aws-cloudformation=0.19.0,@aws-cdk/aws-cloudwatch=0.19.0,@aws-cdk/aw\
        s-codedeploy-api=0.19.0,@aws-cdk/aws-codepipeline-api=0.19.0,@aws-cdk/a\
        ws-ec2=0.19.0,@aws-cdk/aws-ecr=0.19.0,@aws-cdk/aws-ecs=0.19.0,@aws-cdk/\
        aws-elasticloadbalancingv2=0.19.0,@aws-cdk/aws-events=0.19.0,@aws-cdk/a\
        ws-iam=0.19.0,@aws-cdk/aws-kms=0.19.0,@aws-cdk/aws-lambda=0.19.0,@aws-c\
        dk/aws-logs=0.19.0,@aws-cdk/aws-route53=0.19.0,@aws-cdk/aws-s3=0.19.0,@\
        aws-cdk/aws-s3-notifications=0.19.0,@aws-cdk/aws-sns=0.19.0,@aws-cdk/aw\
        s-sqs=0.19.0,@aws-cdk/cdk=0.19.0,@aws-cdk/cx-api=0.19.0,my_ecs_construc\
        t=0.1.0"
Outputs:
  MyFargateServiceLoadBalancerDNS704F6391:
    Value:
      Fn::GetAtt:
        - MyFargateServiceLBDE830E97
        - DNSName
    Export:
      Name: MyEcsConstructStack:MyFargateServiceLoadBalancerDNS704F6391
```

Which creates the following AWS resources\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/CDK/latest/userguide/images/MyEcsConstruct.png)

## The Developer Experience<a name="cdk_developer_experience"></a>

To get started see [Installing and Configuring the AWS CDK](cdk_install_config.md)\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/CDK/latest/userguide/images/screencast.gif)

Developers can use one of the supported programming languages to define reusable cloud components called [Constructs](cdk_constructs.md), which are composed together into [Stacks](cdk_stacks.md) and [Apps](cdk_apps.md)\.

**Note**  
Unless otherwise indicated, the code examples in this guide are in TypeScript\.  
To aid you in porting a TypeScript example to a supported programming language, take a look at the example code for your language in the [Reference](https://awslabs.github.io/aws-cdk/reference.html)\.

The [Command\-line Toolkit \(cdk\)](cdk_tools.md) is a command\-line tool for interacting with CDK apps\. It allows developers to synthesize artifacts such as AWS CloudFormation Templates, deploy stacks to development AWS accounts and diff against a deployed stack to understand the impact of a code change\.

The [AWS Construct Library](cdk_aws_construct_lib.md) includes a module for each AWS service with constructs that offer rich APIs that encapsulate the details of how to use AWS\. The AWS Construct Library aims to reduce the complexity and glue\-logic required when integrating various AWS services to achieve your goals on AWS\.

**Note**  
There is no charge for using the AWS CDK, however you may incur AWS charges for creating or using AWS [chargeable resources](http://docs.aws.amazon.com/general/latest/gr/glos-chap.html#chargeable-resources), such as running Amazon EC2 instances or using Amazon S3 storage\. Use the [AWS Simple Monthly Calculator](http://calculator.s3.amazonaws.com/index.html) to estimate charges for the use of various AWS resources\.

## Additional Documentation and Resources<a name="cdk_additional_docs"></a>

In addition to this guide, the following are other resources available to AWS CDK users:
+ [CDK Reference](https://awslabs.github.io/aws-cdk/)
+ [AWS Developer blog](https://aws.amazon.com/blogs/developer)
+ [Gitter Channel](https://gitter.im/awslabs/aws-cdk)
+ [Stack Overflow](https://stackoverflow.com/questions/tagged/aws-cdk)
+ [GitHub repository](https://github.com/awslabs/aws-cdk)
  + [Examples](https://github.com/awslabs/aws-cdk/tree/master/examples)
  + [Documentation source](https://github.com/awslabs/aws-cdk/docs_src)
  + [Issues](https://github.com/awslabs/aws-cdk/issues)
  + [License](https://github.com/awslabs/aws-cdk/blob/master/LICENSE)
+ [AWS CDK Sample for Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-cdk.html)
+ [AWS CloudFormation Concepts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)

## About Amazon Web Services<a name="cdk_about_aws"></a>

Amazon Web Services \(AWS\) is a collection of digital infrastructure services that developers can leverage when developing their applications\. The services include computing, storage, database, and application synchronization \(messaging and queuing\)\.

AWS uses a pay\-as\-you\-go service model\. You are charged only for the services that you — or your applications — use\. Also, to make AWS useful as a platform for prototyping and experimentation, AWS offers a free usage tier, in which services are free below a certain level of usage\. For more information about AWS costs and the free usage tier, see [Test\-Driving AWS in the Free Usage Tier](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-free-tier.html)\.

To obtain an AWS account, go to [aws\.amazon\.com](https://aws.amazon.com) and click **Create an AWS Account**\.