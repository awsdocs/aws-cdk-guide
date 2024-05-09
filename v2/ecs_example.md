# Creating an AWS Fargate service using the AWS CDK<a name="ecs_example"></a>

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

See [ECS](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_ecs-readme.html) for details\.

**Important**  
The `ApplicationLoadBalancedFargateService` constructs we'll be using includes numerous AWS components, some of which have non\-trivial costs if left provisioned in your AWS account, even if you don't use them\. Be sure to clean up \(cdk destroy\) after completing this example\.

## Creating the directory and initializing the AWS CDK<a name="ecs_example_initialize"></a>

Let's start by creating a directory to hold the AWS CDK code, and then creating a AWS CDK app in that directory\.

------
#### [ TypeScript ]

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language typescript
```

------
#### [ JavaScript ]

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language javascript
```

------
#### [ Python ]

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language python
source .venv/bin/activate # On Windows, run '.\venv\Scripts\activate' instead
pip install -r requirements.txt
```

------
#### [ Java ]

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language java
```

You may now import the Maven project into your IDE\.

------
#### [ C\# ]

```
mkdir MyEcsConstruct
cd MyEcsConstruct
cdk init --language csharp
```

You may now open `src/MyEcsConstruct.sln` in Visual Studio\.

------

Run the app and confirm that it creates an empty stack\.

```
cdk synth
```

## Create a Fargate service<a name="ecs_example_create_fargate_service"></a>

There are two different ways to run your container tasks with Amazon ECS:
+ Use the `Fargate` launch type, where Amazon ECS manages the physical machines that your containers are running on for you\.
+ Use the `EC2` launch type, where you do the managing, such as specifying automatic scaling\.

For this example, we'll create a Fargate service running on an ECS cluster fronted by an internet\-facing Application Load Balancer\.

Add the following AWS Construct Library module imports to the indicated file\.

------
#### [ TypeScript ]

File: `lib/my_ecs_construct-stack.ts`

```
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as ecs from "aws-cdk-lib/aws-ecs";
import * as ecs_patterns from "aws-cdk-lib/aws-ecs-patterns";
```

------
#### [ JavaScript ]

File: `lib/my_ecs_construct-stack.js`

```
const ec2 = require("aws-cdk-lib/aws-ec2");
const ecs = require("aws-cdk-lib/aws-ecs");
const ecs_patterns = require("aws-cdk-lib/aws-ecs-patterns");
```

------
#### [ Python ]

File: `my_ecs_construct/my_ecs_construct_stack.py`

```
from aws_cdk import (aws_ec2 as ec2, aws_ecs as ecs,
                     aws_ecs_patterns as ecs_patterns)
```

------
#### [ Java ]

File: `src/main/java/com/myorg/MyEcsConstructStack.java`

```
import software.amazon.awscdk.services.ec2.*;
import software.amazon.awscdk.services.ecs.*;
import software.amazon.awscdk.services.ecs.patterns.*;
```

------
#### [ C\# ]

File: `src/MyEcsConstruct/MyEcsConstructStack.cs`

```
using Amazon.CDK.AWS.EC2;
using Amazon.CDK.AWS.ECS;
using Amazon.CDK.AWS.ECS.Patterns;
```

------

Replace the comment at the end of the constructor with the following code\.

------
#### [ TypeScript ]

```
    const vpc = new ec2.Vpc(this, "MyVpc", {
      maxAzs: 3 // Default is all AZs in region
    });

    const cluster = new ecs.Cluster(this, "MyCluster", {
      vpc: vpc
    });

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.ApplicationLoadBalancedFargateService(this, "MyFargateService", {
      cluster: cluster, // Required
      cpu: 512, // Default is 256
      desiredCount: 6, // Default is 1
      taskImageOptions: { image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample") },
      memoryLimitMiB: 2048, // Default is 512
      publicLoadBalancer: true // Default is true
    });
```

------
#### [ JavaScript ]

```
    const vpc = new ec2.Vpc(this, "MyVpc", {
      maxAzs: 3 // Default is all AZs in region
    });

    const cluster = new ecs.Cluster(this, "MyCluster", {
      vpc: vpc
    });

    // Create a load-balanced Fargate service and make it public
    new ecs_patterns.ApplicationLoadBalancedFargateService(this, "MyFargateService", {
      cluster: cluster, // Required
      cpu: 512, // Default is 256
      desiredCount: 6, // Default is 1
      taskImageOptions: { image: ecs.ContainerImage.fromRegistry("amazon/amazon-ecs-sample") },
      memoryLimitMiB: 2048, // Default is 512
      publicLoadBalancer: true // Default is true
    });
```

------
#### [ Python ]

```
        vpc = ec2.Vpc(self, "MyVpc", max_azs=3)     # default is all AZs in region

        cluster = ecs.Cluster(self, "MyCluster", vpc=vpc)

        ecs_patterns.ApplicationLoadBalancedFargateService(self, "MyFargateService",
            cluster=cluster,            # Required
            cpu=512,                    # Default is 256
            desired_count=6,            # Default is 1
            task_image_options=ecs_patterns.ApplicationLoadBalancedTaskImageOptions(
                image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample")),
            memory_limit_mib=2048,      # Default is 512
            public_load_balancer=True)  # Default is True
```

------
#### [ Java ]

```
        Vpc vpc = Vpc.Builder.create(this, "MyVpc")
                            .maxAzs(3)  // Default is all AZs in region
                            .build();

        Cluster cluster = Cluster.Builder.create(this, "MyCluster")
                            .vpc(vpc).build();

        // Create a load-balanced Fargate service and make it public
        ApplicationLoadBalancedFargateService.Builder.create(this, "MyFargateService")
                    .cluster(cluster)           // Required
                    .cpu(512)                   // Default is 256
                     .desiredCount(6)            // Default is 1
                     .taskImageOptions(
                             ApplicationLoadBalancedTaskImageOptions.builder()
                                     .image(ContainerImage.fromRegistry("amazon/amazon-ecs-sample"))
                                     .build())
                     .memoryLimitMiB(2048)       // Default is 512
                     .publicLoadBalancer(true)   // Default is true
                     .build();
```

------
#### [ C\# ]

```
            var vpc = new Vpc(this, "MyVpc", new VpcProps
            {
                MaxAzs = 3 // Default is all AZs in region
            });

            var cluster = new Cluster(this, "MyCluster", new ClusterProps
            {
                Vpc = vpc
            });

            // Create a load-balanced Fargate service and make it public
            new ApplicationLoadBalancedFargateService(this, "MyFargateService",
                new ApplicationLoadBalancedFargateServiceProps
                {
                    Cluster = cluster,          // Required
                    DesiredCount = 6,           // Default is 1
                    TaskImageOptions = new ApplicationLoadBalancedTaskImageOptions
                    {
                        Image = ContainerImage.FromRegistry("amazon/amazon-ecs-sample")
                    },
                    MemoryLimitMiB = 2048,      // Default is 256
                    PublicLoadBalancer = true   // Default is true
                }
            );
```

------

Save it and make sure it runs and creates a stack\.

```
cdk synth
```

The stack is hundreds of lines, so we won't show it here\. The stack should contain one default instance, a private subnet and a public subnet for the three Availability Zones, and a security group\.

Deploy the stack\.

```
cdk deploy
```

AWS CloudFormation displays information about the dozens of steps that it takes as it deploys your app\.

That's how easy it is to create a Fargate\-powered Amazon ECS service to run a Docker image\.

## Clean up<a name="ecs_example_destroy"></a>

To avoid unexpected AWS charges, destroy your AWS CDK stack after you're done with this exercise\.

```
cdk destroy
```