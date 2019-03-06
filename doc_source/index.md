# AWS Cloud Development Kit (CDK) User Guide

-----
*****Copyright &copy; 2019 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What Is the AWS CDK?](what-is.md)
+ [Installing and Configuring the AWS CDK](install_config.md)
+ [Hello World Tutorial](hello_world_tutorial.md)
+ [AWS CDK Concepts](concepts.md)
   + [CDK Core](core_concepts.md)
      + [Constructs](constructs.md)
      + [Apps and Stacks](apps_and_stacks.md)
      + [Logical IDs](logical_ids.md)
      + [Environments and Run-Time Context](environments_and_context.md)
      + [Assets](assets.md)
   + [AWS Construct Library](aws_construct_lib.md)
      + [AWS CloudFormation Library](cloudformation.md)
      + [Accessing the AWS CloudFormation Layer](cfn_layer.md)
+ [AWS CDK HowTos](how_tos.md)
   + [Get External Values in a CDK Application](how_to_get_ext_values.md)
      + [Get a Value from a Context Variable](get_context_var.md)
      + [Get a Value from an Environment Variable](get_env_var.md)
      + [Get a Value from a Systems Manager Parameter Store Variable](get_ssm_value.md)
      + [Get a Value from AWS Secrets Manager](passing_secrets_manager.md)
      + [Pass in a Value from Another Stack](passing_stack_value.md)
      + [Use an AWS CloudFormation Parameter](get_cfn_param.md)
      + [Use an Existing AWS CloudFormation Template](use_cfn_template.md)
   + [Set a CloudWatch Alarm](how_to_set_cw_alarm.md)
   + [Stack How-Tos](stack_how_to.md)
+ [Advanced Tutorials](advanced_tutorials.md)
   + [Creating a Serverless Application Using the AWS CDK](serverless_tutorial.md)
   + [Creating an AWS Fargate Service Using the AWS CDK](ecs_tutorial.md)
+ [AWS CDK Command Line Toolkit (cdk)](tools.md)
+ [Writing AWS CDK Constructs](writing_constructs.md)
+ [Document History for the AWS CDK User Guide](doc-history.md)