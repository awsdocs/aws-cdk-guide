# Resilience for the AWS Cloud Development Kit \(AWS CDK\)<a name="disaster-recovery-resiliency"></a>

The Amazon Web Services \(AWS\) global infrastructure is built around AWS Regions and Availability Zones\.

AWS Regions provide multiple physically separated and isolated Availability Zones, which are connected with low\-latency, high\-throughput, and highly redundant networking\.

With Availability Zones, you can design and operate applications and databases that automatically fail over between Availability Zones without interruption\. Availability Zones are more highly available, fault tolerant, and scalable than traditional single or multiple data center infrastructures\.

For more information about AWS Regions and Availability Zones, see [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)\.

The AWS CDK follows the [shared responsibility model](https://aws.amazon.com/compliance/shared-responsibility-model/) through the specific Amazon Web Services \(AWS\) services it supports\. For AWS service security information, see the [AWS service security documentation page](https://docs.aws.amazon.com/security/?id=docs_gateway#aws-security) and [AWS services that are in scope of AWS compliance efforts by compliance program](https://aws.amazon.com/compliance/services-in-scope/)\.