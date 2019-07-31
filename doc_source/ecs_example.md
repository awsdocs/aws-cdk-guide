# Creating an AWS Fargate Service Using the AWS CDK<a name="ecs_example"></a>

This example walks you through how to create an AWS Fargate service running on an Amazon Elastic Container Service \(Amazon ECS\) cluster that's fronted by an internet\-facing Application Load Balancer from an image on Amazon ECR\.

Amazon ECS is a highly scalable, fast, container management service that makes it easy to run, stop, and manage Docker containers on a cluster\. You can host your cluster on a serverless infrastructure that's managed by Amazon ECS by launching your services or tasks using the Fargate launch type\. For more control, you can host your tasks on a cluster of Amazon Elastic Compute Cloud \(Amazon EC2\) instances that you manage by using the Amazon EC2 launch type\.

This tutorial shows you how to launch some services using the Fargate launch type\. If you've used the AWS Management Console to create a Fargate service, you know that there are many steps to follow to accomplish that task\. AWS has several tutorials and documentation topics that walk you through creating a Fargate service, including:
+ [How to Deploy Docker Containers \- AWS](https://aws.amazon.com/getting-started/tutorials/deploy-docker-containers)
+ [Setting Up with Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/get-set-up-for-amazon-ecs.html)
+ [Getting Started with Amazon ECS Using Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html)

This example creates a similar Fargate service in AWS CDK code\.

The Amazon ECS construct used in this tutorial helps you use AWS services by providing the following benefits:
+ Automatically configures a load balancer\.
+ Automatically opens a security group for load balancers\. This enables load balancers to communicate with instances without you explicitly creating a security group\.
+ Automatically orders dependency between the service and the load balancer attaching to a target group, where the AWS CDK enforces the correct order of creating the listener before an instance is created\.
+ Automatically configures user data on automatically scaling groups\. This creates the correct configuration to associate a cluster to AMIs\.
+ Validates parameter combinations early\. This exposes AWS CloudFormation issues earlier, thus saving you deployment time\. For example, depending on the task, it's easy to misconfigure the memory settings\. Previously, you would not encounter an error until you deployed your app\. But now the AWS CDK can detect a misconfiguration and emit an error when you synthesize your app\.
+ Automatically adds permissions for Amazon Elastic Container Registry \(Amazon ECR\) if you use an image from Amazon ECR\.
+ Automatically scales\. The AWS CDK supplies a method so you can autoscalinginstances when you use an Amazon EC2 cluster\. This happens automatically when you use an instance in a Fargate cluster\.

  In addition, the AWS CDK prevents an instance from being deleted when automatic scaling tries to kill an instance, but either a task is running or is scheduled on that instance\.

  Previously, you had to create a Lambda function to have this functionality\.
+ Provides asset support, so that you can deploy a source from your machine to Amazon ECS in one step\. Previously, to use an application source you had to perform several manual steps, such as uploading to Amazon ECR and creating a Docker image\.

See [ECS](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-ecs-readme.html) for details\.

## Creating the Directory and Initializing the AWS CDK<a name="ecs_example_initialize"></a>

Let's start by creating a directory to hold the AWS CDK code, and then creating a AWS CDK app in that directory\.

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language typescript
```

Build the app and confirm that it creates an empty stack\.

```
npm run build
cdk synth
```

You should see a stack like the following, where *CDK\-VERSION* is the version of the CDK\.

```
Resources:
  CDKMetadata:
    Type: 'AWS::CDK::Metadata'
    Properties:
      Modules: @aws-cdk/cdk=CDK-VERSION,@aws-cdk/cx-api=CDK-VERSION,my_ecs_construct=0.1.0
```

## Add the Amazon EC2 and Amazon ECS Packages<a name="ecs_example_add_packages"></a>

Install support for Amazon EC2 and Amazon ECS\.

```
npm install @aws-cdk/aws-ec2 @aws-cdk/aws-ecs @aws-cdk/aws-ecs-patterns
```

## Create a Fargate Service<a name="ecs_example_create_fargate_service"></a>

There are two different ways to run your container tasks with Amazon ECS:
+ Use the `Fargate` launch type, where Amazon ECS manages the physical machines that your containers are running on for you\.
+ Use the `EC2` launch type, where you do the managing, such as specifying automatic scaling\.

The following tutorial creates a Fargate service running on an ECS cluster fronted by an internet\-facing Application load Balancer\.

Add the following `import` statements to `lib/my_ecs_construct-stack.ts`\.

```
import ec2 = require("@aws-cdk/aws-ec2");
import ecs = require("@aws-cdk/aws-ecs");
import ecs_patterns = require("@aws-cdk/aws-ecs-patterns");
```

Replace the comment at the end of the constructor with the following code\.

```
    const vpc = new ec2.Vpc(this, "MyVpc", {
      maxAzs: 3 // Default is all AZs in region
    });

    const cluster = new ecs.Cluster(this, "MyCluster", {
      vpc: vpc
    });

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.LoadBalancedFargateService(this, "MyFargateService", {
      cluster: cluster, // Required
      cpu: 512, // Default is 256
      desiredCount: 6, // Default is 1
      image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample"), // Required
      memoryLimitMiB: 2048, // Default is 512
      publicLoadBalancer: true // Default is false
    });
```

Save it and make sure it builds and creates a stack\.

```
npm run build
cdk synth
```

The stack is hundreds of lines, so we don't show it here\. The stack should contain one default instance, a private subnet and a public subnet for the three Availability Zones, and a security group\.

Deploy the stack\.

```
cdk deploy
```

AWS CloudFormation displays information about the dozens of steps that it takes as it deploys your app\.

That's how easy it is to create a Fargate service to run a Docker image\.