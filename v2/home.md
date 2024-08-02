# What is the AWS CDK?<a name="home"></a>

The AWS Cloud Development Kit \(AWS CDK\) is an open\-source software development framework for defining cloud infrastructure in code and provisioning it through AWS CloudFormation\.

The AWS CDK consists of two primary parts:
+ **[AWS CDK Construct Library](constructs.md)** – A collection of pre\-written modular and reusable pieces of code, called constructs, that you can use, modify, and integrate to develop your infrastructure quickly\. The goal of the AWS CDK Construct Library is to reduce the complexity required to define and integrate AWS services together when building applications on AWS\.
+ **[AWS CDK Command Line Interface \(AWS CDKCLI](cli.md)** – A command line tool for interacting with CDK apps\. Use the CDK CLI to create, manage, and deploy your AWS CDK projects\. The CDK CLI is also referred to as the CDK Toolkit\.

The AWS CDK supports TypeScript, JavaScript, Python, Java, C\#/\.Net, and Go\. You can use any of these supported programming languages to define reusable cloud components known as [constructs](constructs.md)\. You compose these together into [stacks](stacks.md) and [apps](apps.md)\. Then, you deploy your CDK applications to AWS CloudFormation to provision or update your resources\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/v2/guide/images/AppStacks.png)

**Topics**
+ [Benefits of the AWS CDK](#home-benefits)
+ [Example of the AWS CDK](#home-example)
+ [AWS CDK features](#home-features)
+ [Next steps](#home-next)
+ [Learn more](#home-learn)

## Benefits of the AWS CDK<a name="home-benefits"></a>

Use the AWS CDK to develop reliable, scalable, cost\-effective applications in the cloud with the considerable expressive power of a programming language\. This approach yields many benefits, including:

**Develop and manage your infrastructure as code \(IaC\)**  <a name="home-benefits-iac"></a>
Practice *infrastructure as code* to create, deploy, and maintain infrastructure in a programmatic, descriptive, and declarative way\. With IaC, you treat infrastructure the same way developers treat code\. This results in a scalable and structured approach to managing infrastructure\. To learn more about IaC, see [ Infrastructure as code](https://docs.aws.amazon.com/whitepapers/latest/introduction-devops-aws/infrastructure-as-code.html) in the *Introduction to DevOps on AWS Whitepaper*\.  
With the AWS CDK, you can put your infrastructure, application code, and configuration all in one place, ensuring that you have a complete, cloud\-deployable system at every milestone\. Employ software engineering best practices such as code reviews, unit tests, and source control to make your infrastructure more robust\.

**Define your cloud infrastructure using general\-purpose programming languages**  <a name="home-benefits-languages"></a>
With the AWS CDK, you can use any of the following programming languages to define your cloud infrastructure: TypeScript, JavaScript, Python, Java, C\#/\.Net, and Go\. Choose your preferred language and use programming elements like parameters, conditionals, loops, composition, and inheritance to define the desired outcome of your infrastructure\.  
Use the same programming language to define your infrastructure and your application logic\.  
Receive the benefits of developing infrastructure in your preferred IDE \(Integrated Development Environment\), such as syntax highlighting and intelligent code completion\.  

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/v2/guide/images/CodeCompletion.png)

**Deploy infrastructure through AWS CloudFormation**  <a name="home-benefits-cfn"></a>
AWS CDK integrates with AWS CloudFormation to deploy and provision your infrastructure on AWS\. AWS CloudFormation is a managed AWS service that offers extensive support of resource and property configurations for provisioning services on AWS\. With AWS CloudFormation, you can perform infrastructure deployments predictably and repeatedly, with rollback on error\. If you are already familiar with AWS CloudFormation, you don’t have to learn a new IaC management service when getting started with the AWS CDK\.

**Get started developing your application quickly with constructs**  <a name="home-benefits-constructs"></a>
Develop faster by using and sharing reusable components called constructs\. Use low\-level constructs to define individual AWS CloudFormation resources and their properties\. Use high\-level constructs to quickly define larger components of your application, with sensible, secure defaults for your AWS resources, defining more infrastructure with less code\.  
Create your own constructs that are customized for your unique use cases and share them across your organization or even with the public\.

## Example of the AWS CDK<a name="home-example"></a>

The following is an example of using the AWS CDK Constructs Library to create an Amazon Elastic Container Service \(Amazon ECS\) service with AWS Fargate \(Fargate\) launch type\. For more details of this example, see [Creating an AWS Fargate service using the AWS CDK](ecs_example.md)\.

------
#### [ TypeScript ]

```
export class MyEcsConstructStack extends Stack {
  constructor(scope: App, id: string, props?: StackProps) {
    super(scope, id, props);

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
      publicLoadBalancer: true // Default is false
    });
  }
}
```

------
#### [ JavaScript ]

```
class MyEcsConstructStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

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
      publicLoadBalancer: true // Default is false
    });
  }
}

module.exports = { MyEcsConstructStack }
```

------
#### [ Python ]

```
class MyEcsConstructStack(Stack):

    def __init__(self, scope: Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        vpc = ec2.Vpc(self, "MyVpc", max_azs=3)     # default is all AZs in region

        cluster = ecs.Cluster(self, "MyCluster", vpc=vpc)

        ecs_patterns.ApplicationLoadBalancedFargateService(self, "MyFargateService",
            cluster=cluster,            # Required
            cpu=512,                    # Default is 256
            desired_count=6,            # Default is 1
            task_image_options=ecs_patterns.ApplicationLoadBalancedTaskImageOptions(
                image=ecs.ContainerImage.from_registry("amazon/amazon-ecs-sample")),
            memory_limit_mib=2048,      # Default is 512
            public_load_balancer=True)  # Default is False
```

------
#### [ Java ]

```
public class MyEcsConstructStack extends Stack {

    public MyEcsConstructStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyEcsConstructStack(final Construct scope, final String id,
            StackProps props) {
        super(scope, id, props);

        Vpc vpc = Vpc.Builder.create(this, "MyVpc").maxAzs(3).build();

        Cluster cluster = Cluster.Builder.create(this, "MyCluster")
                .vpc(vpc).build();

        ApplicationLoadBalancedFargateService.Builder.create(this, "MyFargateService")
                .cluster(cluster)
                .cpu(512)
                .desiredCount(6)
                .taskImageOptions(
                       ApplicationLoadBalancedTaskImageOptions.builder()
                               .image(ContainerImage
                                       .fromRegistry("amazon/amazon-ecs-sample"))
                               .build()).memoryLimitMiB(2048)
                .publicLoadBalancer(true).build();
    }
}
```

------
#### [ C\# ]

```
public class MyEcsConstructStack : Stack
{
    public MyEcsConstructStack(Construct scope, string id, IStackProps props=null) : base(scope, id, props)
    {
        var vpc = new Vpc(this, "MyVpc", new VpcProps
        {
            MaxAzs = 3
        });

        var cluster = new Cluster(this, "MyCluster", new ClusterProps
        {
            Vpc = vpc
        });

        new ApplicationLoadBalancedFargateService(this, "MyFargateService", 
            new ApplicationLoadBalancedFargateServiceProps
        {
            Cluster = cluster,
            Cpu = 512,
            DesiredCount = 6,
            TaskImageOptions = new ApplicationLoadBalancedTaskImageOptions
            {
                Image = ContainerImage.FromRegistry("amazon/amazon-ecs-sample")
            },
            MemoryLimitMiB = 2048,
            PublicLoadBalancer = true,
        });
    }
}
```

------
#### [ Go ]

```
func NewMyEcsConstructStack(scope constructs.Construct, id string, props *MyEcsConstructStackProps) awscdk.Stack {

	var sprops awscdk.StackProps

	if props != nil {
		sprops = props.StackProps
	}

	stack := awscdk.NewStack(scope, &id, &sprops)

	vpc := awsec2.NewVpc(stack, jsii.String("MyVpc"), &awsec2.VpcProps{
		MaxAzs: jsii.Number(3), // Default is all AZs in region
	})

	cluster := awsecs.NewCluster(stack, jsii.String("MyCluster"), &awsecs.ClusterProps{
		Vpc: vpc,
	})

	awsecspatterns.NewApplicationLoadBalancedFargateService(stack, jsii.String("MyFargateService"),
		&awsecspatterns.ApplicationLoadBalancedFargateServiceProps{
			Cluster:        cluster,           // required
			Cpu:            jsii.Number(512),  // default is 256
			DesiredCount:   jsii.Number(5),    // default is 1
			MemoryLimitMiB: jsii.Number(2048), // Default is 512
			TaskImageOptions: &awsecspatterns.ApplicationLoadBalancedTaskImageOptions{
				Image: awsecs.ContainerImage_FromRegistry(jsii.String("amazon/amazon-ecs-sample"), nil),
			},
			PublicLoadBalancer: jsii.Bool(true), // Default is false
		})

	return stack

}
```

------

This class produces an AWS CloudFormation [template of more than 500 lines](https://github.com/awsdocs/aws-cdk-guide/blob/main/doc_source/my_ecs_construct-stack.yaml)\. Deploying the AWS CDK app produces more than 50 resources of the following types\.
+  [AWS::EC2::EIP](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html) 
+  [AWS::EC2::InternetGateway](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-internetgateway.html) 
+  [AWS::EC2::NatGateway](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-natgateway.html) 
+  [AWS::EC2::Route](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route.html) 
+  [AWS::EC2::RouteTable](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-routetable.html) 
+  [AWS::EC2::SecurityGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) 
+  [AWS::EC2::Subnet](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html) 
+  [AWS::EC2::SubnetRouteTableAssociation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet-route-table-assoc.html) 
+  [AWS::EC2::VPCGatewayAttachment](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc-gateway-attachment.html) 
+  [AWS::EC2::VPC](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html) 
+  [AWS::ECS::Cluster](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-cluster.html) 
+  [AWS::ECS::Service](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-service.html) 
+  [AWS::ECS::TaskDefinition](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ecs-taskdefinition.html) 
+  [AWS::ElasticLoadBalancingV2::Listener](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-listener.html) 
+  [AWS::ElasticLoadBalancingV2::LoadBalancer](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-loadbalancer.html) 
+  [AWS::ElasticLoadBalancingV2::TargetGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticloadbalancingv2-targetgroup.html) 
+  [AWS::IAM::Policy](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-policy.html) 
+  [AWS::IAM::Role](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html) 
+  [AWS::Logs::LogGroup](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-logs-loggroup.html) 

## AWS CDK features<a name="home-features"></a>

### The AWS CDK GitHub repository<a name="home-features-repo"></a>

For the official AWS CDK GitHub repository, see [ aws\-cdk](https://github.com/aws/aws-cdk)\. Here, you can submit [issues](https://github.com/aws/aws-cdk/issues), view our [license](https://github.com/aws/aws-cdk/blob/main/LICENSE), track [releases](https://github.com/aws/aws-cdk/releases), and more\.

Because the AWS CDK is open\-source, the team encourages you to contribute to make it an even better tool\. For details, see [Contributing to the AWS Cloud Development Kit \(AWS CDK\)](https://github.com/aws/aws-cdk/blob/main/CONTRIBUTING.md)\.

### The AWS CDK API reference<a name="home-features-api"></a>

The AWS CDK Construct Library provides APIs to define your CDK application and add CDK constructs to the application\. For more information, see the [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-construct-library.html)\.

### The Construct Programming Model<a name="home-features-cpm"></a>

The Construct Programming Model \(CPM\) extends the concepts behind the AWS CDK into additional domains\. Other tools using the CPM include:
+ [CDK for Terraform](https://www.terraform.io/docs/cdktf/index.html) \(CDKtf\)
+ [CDK for Kubernetes](https://cdk8s.io/) \(CDK8s\)
+ [Projen](https://github.com/projen/projen), for building project configurations

### The Construct Hub<a name="home-features-hub"></a>

The [Construct Hub](https://constructs.dev/) is an online registry where you can find, publish, and share open\-source AWS CDK libraries\.

## Next steps<a name="home-next"></a>

To get started with using the AWS CDK, see [Getting started with the AWS CDK](getting_started.md)\.

## Learn more<a name="home-learn"></a>

To continue learning about the AWS CDK, see the following:
+ **[Learn AWS CDK core concepts](core_concepts.md)** – Important concepts and terms for the AWS CDK\.
+ **[AWS CDK Workshop](https://cdkworkshop.com/)** – Hands\-on workshop to learn and use the AWS CDK\.
+ **[AWS CDK Patterns](https://cdkpatterns.com/)** – Open\-source collection of AWS serverless architecture patterns, built for the AWS CDK by AWS experts\.
+ **[AWS CDK code examples](https://github.com/aws-samples/aws-cdk-examples) ** – GitHub repository of example AWS CDK projects\.
+ **[cdk\.dev](https://cdk.dev/)** – Community\-driven hub for the AWS CDK, including a community Slack workspace\.
+ **[Awesome CDK](https://github.com/kalaiser/awesome-cdk)** – GitHub repository containing a curated list of AWS CDK open\-source projects, guides, blogs, and other resources\.
+ **[AWS Solutions Constructs](https://aws.amazon.com/solutions/constructs/) ** – Vetted, configuration infrastructure as code \(IaC\) patterns that can easily be assembled into production\-ready applications\.
+ **[AWS Developer Tools Blog](https://aws.amazon.com/blogs/developer/category/developer-tools/aws-cloud-development-kit/)** – Blog posts filtered for the AWS CDK\.
+ **[AWS CDK on Stack Overflow](https://stackoverflow.com/questions/tagged/aws-cdk)** – Questions tagged with ** aws\-cdk** on Stack Overflow\.
+ **[AWS CDK tutorial for AWS Cloud9](https://docs.aws.amazon.com/cloud9/latest/user-guide/sample-cdk.html)** – Tutorial on using the AWS CDK with the AWS Cloud9 development environment\.

To learn more about related topics to the AWS CDK, see the following:
+ **[AWS CloudFormation concepts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)** – Since the AWS CDK is built to work with AWS CloudFormation, we recommend that you learn and understand key AWS CloudFormation concepts\.
+ **[AWS Glossary](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html) ** – Definitions of key terms used across AWS\.

To learn more about tools related to the AWS CDK that can be used to simplify serverless application development and deployment, see the following:
+ **[AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/)** – An open\-source developer tool that simplifies and improves the experience of building and running serverless applications on AWS\.
+ **[AWS Chalice](https://github.com/aws/chalice)** – A framework for writing serverless apps in Python\.