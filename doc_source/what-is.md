--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# What Is the AWS CDK?<a name="what-is"></a>

Welcome to the *AWS CDK \(CDK\) Developer Guide*\. This document provides information about the CDK, which is a software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation\.

AWS CloudFormation enables you to:
+ Create and provision AWS infrastructure deployments predictably and repeatedly\.
+ Leverage AWS products such as Amazon EC2, Amazon Elastic Block Store, Amazon SNS, Elastic Load Balancing, and Auto Scaling\.
+ Build highly reliable, highly scalable, cost\-effective applications in the cloud without worrying about creating and configuring the underlying AWS infrastructure\.
+ Use a template file to create and delete a collection of resources together as a single unit \(a stack\)\.

Use the CDK to define your cloud resources using one of the supported programming languages: C\#/\.NET, Java, JavaScript, or TypeScript\. This document does not supply reference information for the CDK\. You can find that information in the [CDK Reference](https://awslabs.github.io/aws-cdk/)\.

Developers can use one of the supported programming languages to define reusable cloud components known as [Constructs](constructs.md)\. You compose these together into [Stacks](apps_and_stacks.md#stacks) and [Apps](apps_and_stacks.md#apps)\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/CDK/latest/guide/images/AppStacks.png)

## Why Use the CDK?<a name="why_use_cdk"></a>

Let's look at the power of the CDK\. Here is some TypeScript code in a CDK project to create an AWS Fargate service \(this is the code we use in the [Creating an AWS Fargate Service Using the AWS CDK](ecs_example.md)\)\.

```
export class MyEcsConstructStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

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
      image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample"), // Required
      memoryMiB: '2048',  // Default is 512
      publicLoadBalancer: true  // Default is false
    });
```

This produces an AWS CloudFormation template of over 600 lines\. We'll show the first 25 lines and Outputs of a cdk synth command\.

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
          Value: MyEcsConstruct/MyVpc
    Metadata:
      aws:cdk:path: MyEcsConstruct/MyVpc/Resource
  MyVpcPublicSubnet1SubnetF6608456:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.0.0/19
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-west-2a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: MyEcsConstruct/MyVpc/PublicSubnet1
        - Key: aws-cdk:subnet-name
...
  MyFargateServiceLoadBalancerDNS704F6391:
    Value:
      Fn::GetAtt:
        - MyFargateServiceLBDE830E97
        - DNSName
```

Another advantage of IAC \(infrastructure as code\) is that you get code completion within your IDE:

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/CDK/latest/guide/images/CodeCompletion.png)

## Developing With the CDK<a name="developing"></a>

Unless otherwise indicated, the code examples in this guide are in TypeScript\. To aid you in porting a TypeScript example to a supported programming language, take a look at the example code for your language in the [CDK Reference](https://awslabs.github.io/aws-cdk/reference.html)\. The CDK also includes some [TypeScript examples](https://github.com/aws-samples/aws-cdk-examples/tree/master/typescript)\.

The [CDK Toolchain](tools.md) is a command line tool for interacting with CDK apps\. It enables developers to synthesize artifacts such as AWS CloudFormation templates, deploy stacks to development AWS accounts, and diff against a deployed stack to understand the impact of a code change\.

The [AWS Construct Library](aws_construct_lib.md) includes a module for each AWS service with constructs that offer rich APIs that encapsulate the details of how to create resources for an Amazon or AWS service\. The aim of the AWS Construct Library is to reduce the complexity and glue logic required when integrating various AWS services to achieve your goals on AWS\.

**Note**  
There is no charge for using the CDK, however you might incur AWS charges for creating or using AWS [chargeable resources](http://docs.aws.amazon.com/general/latest/gr/glos-chap.html#chargeable-resources), such as running Amazon EC2 instances or using Amazon S3 storage\. Use the [AWS Simple Monthly Calculator](http://calculator.s3.amazonaws.com/index.html) to estimate charges for the use of various AWS resources\.

## Contributing to the CDK<a name="contributing"></a>

Because the AWS CDK is open source, the team encourages you contribute to make it an even better tool\. For details, see [Contributing](https://github.com/awslabs/aws-cdk/blob/master/CONTRIBUTING.md)\.

## Additional Documentation and Resources<a name="additional_docs"></a>

In addition to this guide, the following are other resources available to CDK users:
+ [CDK Reference](https://awslabs.github.io/aws-cdk/)
+ [CDK Demo at re:Invent 2018](https://www.youtube.com/watch?v=Lh-kVC2r2AU)
+ [CDK Workshop](https://cdkworkshop.com/)
+ [CDK Examples](https://github.com/aws-samples/aws-cdk-examples)
+ [AWS Developer Blog](https://aws.amazon.com/blogs/developer)
+ [Gitter Channel](https://gitter.im/awslabs/aws-cdk)
+ [Stack Overflow](https://stackoverflow.com/questions/tagged/aws-cdk)
+ [GitHub Repository](https://github.com/awslabs/aws-cdk)
  + [Issues](https://github.com/awslabs/aws-cdk/issues)
  + [Examples](https://github.com/aws-samples/aws-cdk-examples)
  + [Documentation Source](https://github.com/awsdocs/aws-cdk-user-guide/tree/master/doc_source)
  + [License](https://github.com/awslabs/aws-cdk/blob/master/LICENSE)
  + [Releases](https://github.com/awslabs/aws-cdk/releases)
    + [CDK OpenPGP Key](pgp-keys.md#cdk_pgp_key)
    + [JSII OpenPGP Key](pgp-keys.md#jsii_pgp_key)
+ [AWS CDK Sample for Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-cdk.html)
+ [AWS CloudFormation Concepts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)
+ [AWS Glossary](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html)

## About Amazon Web Services<a name="about_aws"></a>

Amazon Web Services \(AWS\) is a collection of digital infrastructure services that developers can use when developing their applications\. The services include computing, storage, database, and application synchronization \(messaging and queuing\)\.

AWS uses a pay\-as\-you\-go service model\. You are charged only for the services that you — or your applications — use\. Also, to make AWS useful as a platform for prototyping and experimentation, AWS offers a free usage tier, in which services are free below a certain level of usage\. For more information about AWS costs and the free usage tier, see [Test\-Driving AWS in the Free Usage Tier](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-free-tier.html)\.

To obtain an AWS account, go to [aws\.amazon\.com](https://aws.amazon.com), and then choose **Create an AWS Account**\.