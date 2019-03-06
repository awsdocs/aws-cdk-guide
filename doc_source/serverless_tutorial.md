--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Creating a Serverless Application Using the AWS CDK<a name="serverless_tutorial"></a>

This tutorial walks you through how to create the resources for a simple widget dispensing service\. It includes:
+ An AWS Lambda function\.
+ An Amazon API Gateway API to call the Lambda function\.
+ An Amazon S3 bucket that contains the Lambda function code\.

This tutorial contains the following steps\.

1. Creates a CDK app

1. Creates a Lambda function that gets a list of widgets with: GET /

1. Creates the service that calls the Lambda function

1. Adds the service to the CDK app

1. Tests the app

1. Add Lambda functions to do the following:
   + Create a widget with POST /\{name\}
   + Get a widget by name with GET /\{name\}
   + Delete a widget by name with DELETE /\{name\}

## Create a CDK App<a name="serverless_tutorial_create_app"></a>

Create the TypeScript app **MyWidgetService** in the current folder\.

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language typescript
```

This creates `my_widget_service.ts` in the `bin` directory, and `my_widget_service-stack.ts` in the `lib` directory\.

Build the app and notice that it creates an empty stack\.

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

## Create a Lambda Function to List All Widgets<a name="serverless_tutorial_create_iam_function"></a>

The next step is to create a Lambda function to list all of the widgets in our Amazon S3 bucket\.

Create the `resources` directory at the same level as the `bin` directory\.

```
mkdir resources
```

Create the following JavaScript file, `widgets.js`, in the `resources` directory\.

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

Save it and be sure it builds and creates an empty stack\. Because we haven't wired the function to the app, the Lambda file doesn't appear in the output\.

```
npm run build
cdk synth
```

## Creating a Widget Service<a name="serverless_tutorial_create_widget_service"></a>

Add the API Gateway, Lambda, and Amazon S3 packages to the app\.

```
npm install @aws-cdk/aws-apigateway @aws-cdk/aws-lambda @aws-cdk/aws-s3
```

Create the TypeScript file `widget_service.ts` in the `lib` directory\.

```
import cdk = require('@aws-cdk/cdk');
import apigateway = require('@aws-cdk/aws-apigateway');
import lambda = require('@aws-cdk/aws-lambda');
import s3 = require('@aws-cdk/aws-s3');

export class WidgetService extends cdk.Construct {
  constructor(scope: cdk.Construct, id: string) {
    super(scope, id);

    const bucket = new s3.Bucket(this, 'WidgetStore');

    const handler = new lambda.Function(this, 'WidgetHandler', {
      runtime: lambda.Runtime.NodeJS810,  // So we can use async in widget.js
      code: lambda.Code.directory('resources'),
      handler: 'widgets.main',
      environment: {
        BUCKET: bucket.bucketName
      }
    });

    bucket.grantReadWrite(handler.role);

    const api = new apigateway.RestApi(this, 'widgets-api', {
      restApiName: 'Widget Service',
      description: 'This service serves widgets.'
    });

    const getWidgetsIntegration = new apigateway.LambdaIntegration(handler, {
      requestTemplates:  { "application/json": '{ "statusCode": "200" }' }
    });

    api.root.addMethod('GET', getWidgetsIntegration);   // GET /

    const widget = api.root.addResource('{id}');

    // Add new widget to bucket with: POST /{id}
    const postWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Get a specific widget from bucket with: GET /{id}
    const getWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Remove a specific widget from the bucket with: DELETE /{id}
    const deleteWidgetIntegration = new apigateway.LambdaIntegration(handler);

    widget.addMethod('POST', postWidgetIntegration);    // POST /{id}
    widget.addMethod('GET', getWidgetIntegration);       // GET /{id}
    widget.addMethod('DELETE', deleteWidgetIntegration); // DELETE /{id}
  }
}
```

Save the app and be sure it builds and creates a \(still empty\) stack\.

```
npm run build
cdk synth
```

## Add the Service to the App<a name="serverless_tutorial_add_service"></a>

To add the service to the app, first modify `my_widget_service-stack.ts`\. Add the following line of code after the existing `import` statement\.

```
import widget_service = require('../lib/widget_service');
```

Replace the comment in the constructor with the following line of code\.

```
    new widget_service.WidgetService(this, 'Widgets');
```

Be sure the app builds and creates a stack \(we don't show the stack because it's over 250 lines\)\.

```
npm run build
cdk synth
```

## Deploy and Test the App<a name="serverless_tutorial_deploy_and_test"></a>

Before you can deploy your first CDK app, you must bootstrap your deployment\. This creates some AWS infrastructure that the CDK needs\. For details, see the **bootstrap** section of the [AWS CDK Command Line Toolkit \(cdk\)](tools.md)\(if you've already bootstrapped a CDK app, you'll get a warning and nothing will change\)\.

```
cdk bootstrap
```

Deploy your app, as follows\.

```
cdk deploy
```

If the deployment succeeds, save the URL for your server\. This URL appears in one of the last lines in the window, where *GUID* is an alphanumeric GUID and *REGION* is your AWS Region\.

```
https://GUID.execute-REGION.amazonaws.com/prod/
```

Test your app by getting the list of widgets \(currently empty\) by navigating to this URL in a browser, or use the following command\.

```
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
```

You can also test the app by:

1. Opening the AWS Management Console\.

1. Navigating to the API Gateway service\.

1. Finding **Widget Service** in the list\.

1. Selecting **GET** and **Test** to test the function\.

Because we haven't stored any widgets yet, the output should be similar to the following\.

```
{ "widgets": [] }
```

## Add the Individual Widget Functions<a name="serverless_tutorial_add_widget_functions"></a>

The next step is to create Lambda functions to create, show, and delete individual widgets\. 

Replace the existing `exports.main` function in `widgets.js` with the following code\.

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

Wire up these functions to your API Gateway code in `widget_service.ts` by adding the following code at the end of the constructor\.

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

We can now store, show, or delete an individual widget\. Use the following commands to list the widgets, create the widget **example**, list all of the widgets, show the contents of **example** \(it should show today's date\), delete **example**, and then show the list of widgets again\.

```
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
curl -X POST 'https://GUID.execute-REGION.amazonaws.com/prod/example'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod/example'
curl -X DELETE 'https://GUID.execute-REGION.amazonaws.com/prod/example'
curl -X GET 'https://GUID.execute-REGION.amazonaws.com/prod'
```

You can also use the API Gateway console to test these functions\. You have to set the **name** value to the name of a widget, such as **example**\.