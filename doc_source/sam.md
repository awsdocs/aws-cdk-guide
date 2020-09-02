# SAM CLI<a name="sam"></a>

This topic describes how to use the AWS SAM CLI with the AWS CDK to test a Lambda function locally\. For further information, see [Invoking Functions Locally](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-invoke.html)\. To install the SAM CLI, see [Installing the AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)\.

1. The first step is to create a AWS CDK application and add the Lambda package\.

   ```
   mkdir cdk-sam-example
   cd cdk-sam-example
   cdk init app --language typescript
   npm install @aws-cdk/aws-lambda
   ```

1. Add a Lambda reference to `lib/cdk-sam-example-stack.ts`:

   ```
   import * as lambda from '@aws-cdk/aws-lambda';
   ```

1. Replace the comment in `lib/cdk-sam-example-stack.ts` with the following Lambda function:

   ```
   new lambda.Function(this, 'MyFunction', {
     runtime: lambda.Runtime.PYTHON_3_7,
     handler: 'app.lambda_handler',
     code:    lambda.Code.asset('./my_function'),
   });
   ```

1. Create the directory `my_function`

   ```
   mkdir my_function
   ```

1. Create the file `app.py` in `my_function` with the following content:

   ```
   def lambda_handler(event, context):
     return "This is a Lambda Function defined through CDK"
   ```

1. Run your AWS CDK app and create a AWS CloudFormation template

   ```
   cdk synth --no-staging > template.yaml
   ```

1. Find the logical ID for your Lambda function in `template.yaml`\. It will look like `MyFunction`*12345678*, where *12345678* represents an 8\-character unique ID that the AWS CDK generates for all resources\. The line right after it should look like:

   ```
   Type: AWS::Lambda::Function
   ```

1. Run the function by executing:

   ```
   sam local invoke MyFunction12345678 --no-event
   ```

   The output should look something like the following\.

   ```
   2019-04-01 12:22:41 Found credentials in shared credentials file: ~/.aws/credentials
   2019-04-01 12:22:41 Invoking app.lambda_handler (python3.7)
   
   Fetching lambci/lambda:python3.7 Docker container image......
   2019-04-01 12:22:43 Mounting D:\cdk-sam-example\.cdk.staging\a57f59883918e662ab3c46b964d2faa5 as /var/task:ro,delegated inside runtime container
   START RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72 Version: $LATEST
   END RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72
   REPORT RequestId: 52fdfc07-2182-154f-163f-5f0f9a621d72     Duration: 3.70 ms       Billed Duration: 100 ms Memory Size: 128 MB     Max Memory Used: 22 MB
   
   "This is a Lambda Function defined through CDK"
   ```