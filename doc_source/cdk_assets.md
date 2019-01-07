--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Assets<a name="cdk_assets"></a>

Assets are local files, directories, or Docker images which can be bundled into CDK constructs and apps\. A common example is a directory which contains the handler code for a Lambda function, but assets can represent any artifact that is needed for the app’s operation\.

When deploying an AWS CDK app that includes constructs with assets, the toolkit first prepares and publishes them to Amazon S3 or Amazon ECR, and only then deploy the stacks\. The locations of the published assets are passed in as AWS CloudFormation parameters to the relevant stacks\.