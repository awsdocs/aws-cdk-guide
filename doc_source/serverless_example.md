--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Creating a Serverless Application Using the AWS CDK<a name="serverless_example"></a>

This example walks you through creating the resources for a simple widget dispensing service\. It includes:
+ An AWS Lambda function
+ An API Gateway API to call our Lambda function
+ An Amazon S3 bucket that contains our Lambda function code

## Overview<a name="serverless_example_overview"></a>

This example contains the following steps\.

1. Create a AWS CDK app

1. Create a Lambda function that gets a list of widgets with: GET /

1. Create the service that calls the Lambda function

1. Add the service to the AWS CDK app

1. Test the app

1. Add Lambda functions to:
   + create an widget based with: POST /\{name\}
   + get an widget by name with: GET /\{name\}
   + delete an widget by name with: DELETE /\{name\}

## Create an AWS CDK App<a name="serverless_example_create_app"></a>

Create the TypeScript app **MyWidgetService** in in the current folder\.

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language typescript
```

This creates `my_widget_service.ts` in the `bin` directory and `my_widget_service-stack.ts` in the `lib` directory\.

Build it and note that it creates an empty stack\.

```
npm run build
cdk synth
```

You should see a stack like the following, where *CDK\-VERSION* is the version of the CDK\.

```
Resources:
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: "@aws-cdk/cdk=CDK-VERSION,@aws-cdk/cx-api=CDK-VERSION,my_widget_service=0.1.0"
```

## Create a Lambda Function to List all Widgets<a name="serverless_example_create_iam_function"></a>

The next step is to create a Lambda function to list all of the widgets in our Amazon S3 bucket\.

Create the directory `resources` at the same level as the bin directory\.

```
mkdir resources
```

Create the following Javascript file, `widgets.js`, in the `resources` directory\.

```
const AWS = require('aws-sdk');
const S3 = new AWS.S3();

const bucketName = process.env.BUCKET;

exports.main = async function(event, context) {
  try {
    var method = event.httpMethod;

    if (method === "GET") {
      if (event.path === "/") {
        const data = await S3.listObjectsV2({ Bucket: bucketName }).promise();
        var body = {
          widgets: data.Contents.map(function(e) { return e.Key })
        };
        return {
          statusCode: 200,
          headers: {},
          body: JSON.stringify(body)
        };
      }
    }

    // We only accept GET for now
    return {
      statusCode: 400,
      headers: {},
      body: "We only accept GET /"
    };
  } catch(error) {
    var body = error.stack || JSON.stringify(error, null, 2);
    return {
      statusCode: 400,
        headers: {},
        body: JSON.stringify(body)
    }
  }
}
```

Save it and make sure it builds and creates an empty stack\. Note that since we haven't wired the function to our app, the Lambda file does not appear in the output\.

```
npm run build
cdk synth
```

## Creating a Widget Service<a name="serverless_example_create_widget_service"></a>

Add the API Gateway, Lambda, and Amazon S3 packages to our app\.

```
npm install @aws-cdk/aws-apigateway @aws-cdk/aws-lambda @aws-cdk/aws-s3
```

Create the Typescript file `widget_service.ts` in the `lib` directory\.

```
import cdk = require('@aws-cdk/cdk');
import apigateway = require('@aws-cdk/aws-apigateway');
import lambda = require('@aws-cdk/aws-lambda');
import s3 = require('@aws-cdk/aws-s3');

export class WidgetService extends cdk.Construct {
  constructor(parent: cdk.Construct, name: string) {
    super(parent, name);

    // Use S3 bucket to store our widgets
    const bucket = new s3.Bucket(this, 'WidgetStore');

    // Create a handler that calls the function main
    // in the source file widgets(.js) in the resources directory
    // to handle requests through API Gateway
    const handler = new lambda.Function(this, 'WidgetHandler', {
      runtime: lambda.Runtime.NodeJS810,
      code: lambda.Code.directory('resources'),
      handler: 'widgets.main',
      environment: {
        BUCKET: bucket.bucketName // So runtime has the bucket name
      }
    });

    bucket.grantReadWrite(handler.role);

    // Create an API Gateway REST API
    const api = new apigateway.RestApi(this, 'widgets-api', {
      restApiName: 'Widget Service',
      description: 'This service serves widgets.'
    });

    // Pass the request to the handler
    const getWidgetsIntegration = new apigateway.LambdaIntegration(handler);

    // Use the getWidgetsIntegration when there is a GET request
    api.root.addMethod('GET', getWidgetsIntegration);   // GET /
  }
}
```

Save it and make sure it builds and creates a \(still empty\) stack\.

```
npm run build
cdk synth
```

## Add the Service to the App<a name="serverless_example_add_service"></a>

To add the service to our app, we need to first modify `my_widget_service-stack.ts`\. Add the following line of code after the existing **import** statement\.

```
import widget_service = require('../lib/widget_service');
```

Replace the comment in the constructor with the following line of code\.

```
new widget_service.WidgetService(this, 'Widgets');
```

Make sure it builds and creates a stack \(we don't show the stack as it's over 250 lines\)\.

```
npm run build
cdk synth
```

## Deploy and Test the App<a name="serverless_example_deploy_and_test"></a>

Before you can deploy your first AWS CDK app, you must bootstrap your deployment, which creates some AWS infracture that the AWS CDK needs\. See the **bootstrap** section of [Command\-line Toolkit \(cdk\)](tools.md) for details \(you'll get a warning and nothing changes if you have already bootstrapped an AWS CDK app\)\.

```
cdk bootstrap
```

Deploy your app:

```
cdk deploy
```

If the deployment is successfull, save the URL for your server, which appears in one of the last lines in the window, where *GUID* is an alpha\-numeric GUID and *REGION* is your region\.

```
https://GUID.execute-REGION.amazonaws.com/prod/
```

You can test your app by getting the list of widgets \(currently empty\) by navigating to this URL in a browser or use the following command\.

```
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
```

You can also test the app by:
+ Opening the AWS Management Console
+ Navigating to the API Gateway service
+ Finding **Widget Service** in the list
+ Selecting **GET** and **Test** to test the function\.

Since we haven't stored any widgets yet, the output should be similar to the following\.

```
{ "widgets": [] }
```

## Add the Individual Widget Functions<a name="serverless_example_add_widget_functions"></a>

The next step is to create Lambda functions to create, show, and delete individual widgets\. Replace the existing `exports.main` function in `widgets.js` with the following code\.

```
exports.main = async function(event, context) {
  try {
    var method = event.httpMethod;
    // Get name, if present
    var widgetName = event.path.startsWith('/') ? event.path.substring(1) : event.path;

    if (method === "GET") {
      // GET / to get the names of all widgets
      if (event.path === "/") {
        const data = await S3.listObjectsV2({ Bucket: bucketName }).promise();
        var body = {
          widgets: data.Contents.map(function(e) { return e.Key })
        };
        return {
          statusCode: 200,
          headers: {},
          body: JSON.stringify(body)
        };
      }

      if (widgetName) {
        // GET /name to get info on widget name
        const data = await S3.getObject({ Bucket: bucketName, Key: widgetName}).promise();
        var body = data.Body.toString('utf-8');

        return {
          statusCode: 200,
          headers: {},
          body: JSON.stringify(body)
        };
      }
    }

    if (method === "POST") {
      // POST /name
      // Return error if we do not have a name
      if (!widgetName) {
        return {
          statusCode: 400,
          headers: {},
          body: "Widget name missing"
        };
      }

      // Create some dummy data to populate object
      const now = new Date();
      var data = widgetName + " created: " + now;

      var base64data = new Buffer(data, 'binary');

      await S3.putObject({
        Bucket: bucketName,
        Key: widgetName,
        Body: base64data,
        ContentType: 'application/json'
      }).promise();

      return {
        statusCode: 200,
        headers: {},
        body: JSON.stringify(event.widgets)
      };
    }

    if (method === "DELETE") {
      // DELETE /name
      // Return an error if we do not have a name
      if (!widgetName) {
        return {
          statusCode: 400,
          headers: {},
          body: "Widget name missing"
        };
      }

      await S3.deleteObject({
        Bucket: bucketName, Key: widgetName
      }).promise();

      return {
        statusCode: 200,
        headers: {},
        body: "Successfully deleted widget " + widgetName
      };
    }

    // We got something besides a GET, POST, or DELETE
    return {
      statusCode: 400,
      headers: {},
      body: "We only accept GET, POST, and DELETE, not " + method
    };
  } catch(error) {
    var body = error.stack || JSON.stringify(error, null, 2);
    return {
      statusCode: 400,
      headers: {},
      body: body
    }
  }
}
```

Wire these functions up to our API Gateway code in `widget_service.ts` by adding the following code at the end of the constructor\.

```
const widget = api.root.addResource('{name}');

// Add new widget to bucket with: POST /{name}
const postWidgetIntegration = new apigateway.LambdaIntegration(handler);

// Get a specific widget from bucket with: GET /{name}
const getWidgetIntegration = new apigateway.LambdaIntegration(handler);

// Remove a specific widget from the bucket with: DELETE /{name}
const deleteWidgetIntegration = new apigateway.LambdaIntegration(handler);

widget.addMethod('POST', postWidgetIntegration);    // POST /{name}
widget.addMethod('GET', getWidgetIntegration);       // GET /{name}
widget.addMethod('DELETE', deleteWidgetIntegration); // DELETE /{name}
```

Save, build, and deploy the app\.

```
npm run build
cdk deploy
```

We can now store, show, or delete an individual widget\. Use the following commands to list the widgets, create the widget **dummy**, list all of the widgets, show the contents of **dummy** \(it should show today's date\), and delete **dummy**, and again show the list of widgets\.

```
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
curl -X POST 'https://GUID.execute-REGION.amazonaws.com/prod/dummy'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod/dummy'
curl -X DELETE 'https://GUID.execute-REGION.amazonaws.com/prod/dummy'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
```

You can also use the API Gateway console to test these functions\. You'll have to set the **name** entry to the name of an widget, such as **dummy**\.