--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Assets<a name="assets"></a>

Assets are local files, directories, or Docker images that can be bundled into AWS CDK constructs and apps\. A common example is a directory that contains the handler code for an AWS Lambda function, however, assets can represent any artifact that the app needs to operate\.

When deploying a AWS CDK app that includes constructs with assets, the AWS CDK Toolkit first prepares and publishes them to Amazon Simple Storage Service \(Amazon S3\) or Amazon Elastic Container Registry \(Amazon ECR\), and only then deploys the stacks\. The locations of the published assets are passed in as AWS CloudFormation parameters to the relevant stacks\.