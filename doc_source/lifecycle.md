--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# AWS CDK Lifecycle<a name="lifecycle"></a>

The following illustration shows the phases that the CDK goes through when you call cdk deploy to create the resources defined by your application\.

![\[Image NOT FOUND\]](http://docs.aws.amazon.com/cdk/latest/guide/images/Lifecycle.png)

There are three actors at play to create the resources that your CDK application defines\.
+ Your CDK app\.
+ The CDK Toolkit\.
+ AWS CloudFormation, which the CDK Toolkit calls to deploy your application and create the resources\.

After you use the toolkit to deploy a CDK application, the application goes through the following phases\.

Construction  
Your code instantiates all desired application constructs and links them together\.

Preparation  
All constructs that have implemented the `prepare` method participate in a final round of modifications, to set up any final state they want to\. The preparation phase happens automatically and users do not see any feedback from this phase\.

Validation  
All constructs that have implemented their `validate` method can validate themselves to make sure they've ended up in a state that will correctly deploy\. Users see any validation failures that are detected during this phase\.

Synthesis  
All constructs render themselves to a set of artifacts, representing AWS CloudFormation templates, AWS Lambda application bundles, and other deployment artifacts\. Users do not see any feedback from this phase\.

Deployment  
The toolkit takes the artifacts produced by the synthesis step, uploads them to Amazon S3 or wherever they need to go, and starts an AWS CloudFormation deployment to deploy the application and create the resources\.

Note that your CDK app has already finished and exited by the time the AWS CloudFormation deployment starts\. This has the following implications\.
+ Your CDK app cannot respond to events that happen during deployment, such as a resource being created or the whole deployment finishing\. To run code during the deployment phase, you have to inject it into the AWS CloudFormation template as a Custom Resource\.
+ Your CDK app might have to work with values that cannot be known at the time it executes\. For example, if your CDK application defines an Amazon S3 Bucket with an automatically generated name, and you retrieve the `bucket.bucketName` attribute, that value is not the name of the deployed bucket\. Instead, the value of the `bucketName` attribute is a symbolic value, looking like *$\{TOKEN\[Bucket\.Name\.1234\]\}*\. You can pass this value to constructs, or append it to other strings, and the CDK framework will translate that value\. You cannot examine the value and make decisions based on the deployed bucket name, because the bucket name not available until AWS CloudFormation is done deploying, and by that time your program is no longer running\. Call `cdk.unresolved(value)`, which returns `true` if `value` not known until deployment time\.