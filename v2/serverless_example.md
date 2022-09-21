# Creating a serverless application using the AWS CDK<a name="serverless_example"></a>

This example walks you through creating the resources for a simple widget dispensing service\. \(For the purpose of this example, a widget is just a name or identifier that can be added to, retrieved from, and deleted from a collection\.\) The example includes:
+ An AWS Lambda function\.
+ An Amazon API Gateway API to call the Lambda function\.
+ An Amazon S3 bucket that holds the widgets\.

This tutorial contains the following steps\.

1. Create an AWS CDK app

1. Create a Lambda function that gets a list of widgets with HTTP GET /

1. Create the service that calls the Lambda function

1. Add the service to the AWS CDK app

1. Test the app

1. Add Lambda functions to do the following:
   + Create a widget with POST /\{name\}
   + Get a widget by name with GET /\{name\}
   + Delete a widget by name with DELETE /\{name\}

1. Tear everything down when you're finished

## Create an AWS CDK app<a name="serverless_example_create_app"></a>

Create the app **MyWidgetService** in the current folder\.

------
#### [ TypeScript ]

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language typescript
```

------
#### [ JavaScript ]

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language javascript
```

------
#### [ Python ]

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language python
source .venv/bin/activate
pip install -r requirements.txt
```

------
#### [ Java ]

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language java
```

You may now import the Maven project into your IDE\.

------
#### [ C\# ]

```
mkdir MyWidgetService
cd MyWidgetService
cdk init --language csharp
```

You may now open `src/MyWidgetService.sln` in Visual Studio\.

------

**Note**  
The CDK names source files and classes based on the name of the project directory\. If you don't use the name `MyWidgetService` as shown above, you'll have trouble following the rest of the steps because some of the files the instructions tell you to modify aren't there \(they'll have different names\)\.

The important files in the blank project are as follows\. \(We will also be adding a couple of new files\.\)

------
#### [ TypeScript ]
+ `bin/my_widget_service.ts` – Main entry point for the application
+ `lib/my_widget_service-stack.ts` – Defines the widget service stack

------
#### [ JavaScript ]
+ `bin/my_widget_service.js` – Main entry point for the application
+ `lib/my_widget_service-stack.js` – Defines the widget service stack

------
#### [ Python ]
+ `app.py` – Main entry point for the application
+ `my_widget_service/my_widget_service_stack.py` – Defines the widget service stack

------
#### [ Java ]
+ `src/main/java/com/myorg/MyWidgetServiceApp.java` – Main entry point for the application
+ `src/main/java/com/myorg/MyWidgetServiceStack.java` – Defines the widget service stack

------
#### [ C\# ]
+ `src/MyWidgetService/Program.cs` – Main entry point for the application
+ `src/MyWidgetService/MyWidgetServiceStack.cs` – Defines the widget service stack

------

Run the app and note that it synthesizes an empty stack\.

```
cdk synth
```

You should see output beginning with YAML code like the following\.

```
Resources:
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      ...
```

## Create a Lambda function to list all widgets<a name="serverless_example_create_iam_function"></a>

The next step is to create a Lambda function to list all of the widgets in our Amazon S3 bucket\. We will provide the Lambda function's code in JavaScript\.

Create the `resources` directory in the project's main directory\.

```
mkdir resources
```

Create the following JavaScript file, `widgets.js`, in the `resources` directory\.

```
/* 
This code uses callbacks to handle asynchronous function responses.
It currently demonstrates using an async-await pattern. 
AWS supports both the async-await and promises patterns.
For more information, see the following: 
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/calling-services-asynchronously.html
https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html 
*/
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

Save it and be sure the project still results in an empty stack\. We haven't yet wired the Lambda function to the AWS CDK app, so the Lambda asset doesn't appear in the output\.

```
cdk synth
```

## Create a widget service<a name="serverless_example_create_widget_service"></a>

Create a new source file to define the widget service with the source code shown below\.

------
#### [ TypeScript ]

File: `lib/widget_service.ts`

```
import * as cdk from "aws-cdk-lib";
import { Construct } from "constructs";
import * as apigateway from "aws-cdk-lib/aws-apigateway";
import * as lambda from "aws-cdk-lib/aws-lambda";
import * as s3 from "aws-cdk-lib/aws-s3";

export class WidgetService extends Construct {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    const bucket = new s3.Bucket(this, "WidgetStore");

    const handler = new lambda.Function(this, "WidgetHandler", {
      runtime: lambda.Runtime.NODEJS_14_X, // So we can use async in widget.js
      code: lambda.Code.fromAsset("resources"),
      handler: "widgets.main",
      environment: {
        BUCKET: bucket.bucketName
      }
    });

    bucket.grantReadWrite(handler); // was: handler.role);

    const api = new apigateway.RestApi(this, "widgets-api", {
      restApiName: "Widget Service",
      description: "This service serves widgets."
    });

    const getWidgetsIntegration = new apigateway.LambdaIntegration(handler, {
      requestTemplates: { "application/json": '{ "statusCode": "200" }' }
    });

    api.root.addMethod("GET", getWidgetsIntegration); // GET /
  }
}
```

------
#### [ JavaScript ]

File: `lib/widget_service.js`

```
const cdk = require("aws-cdk-lib");
const { Construct } = require("constructs");
const apigateway = require("aws-cdk-lib/aws-apigateway");
const lambda = require("aws-cdk-lib/aws-lambda");
const s3 = require("aws-cdk-lib/aws-s3");

class WidgetService extends Construct {
  constructor(scope, id) {
    super(scope, id);

    const bucket = new s3.Bucket(this, "WidgetStore");

    const handler = new lambda.Function(this, "WidgetHandler", {
      runtime: lambda.Runtime.NODEJS_14_X, // So we can use async in widget.js
      code: lambda.Code.fromAsset("resources"),
      handler: "widgets.main",
      environment: {
        BUCKET: bucket.bucketName
      }
    });

    bucket.grantReadWrite(handler); // was: handler.role);

    const api = new apigateway.RestApi(this, "widgets-api", {
      restApiName: "Widget Service",
      description: "This service serves widgets."
    });

    const getWidgetsIntegration = new apigateway.LambdaIntegration(handler, {
      requestTemplates: { "application/json": '{ "statusCode": "200" }' }
    });

    api.root.addMethod("GET", getWidgetsIntegration); // GET /
  }
}

module.exports = { WidgetService }
```

------
#### [ Python ]

File: `my_widget_service/widget_service.py`

```
import aws_cdk as cdk
from constructs import Construct
from aws_cdk import (aws_apigateway as apigateway,
                     aws_s3 as s3,
                     aws_lambda as lambda_)

class WidgetService(Construct):
    def __init__(self, scope: Construct, id: str):
        super().__init__(scope, id)

        bucket = s3.Bucket(self, "WidgetStore")

        handler = lambda_.Function(self, "WidgetHandler",
                    runtime=lambda_.Runtime.NODEJS_14_X,
                    code=lambda_.Code.from_asset("resources"),
                    handler="widgets.main",
                    environment=dict(
                    BUCKET=bucket.bucket_name)
                    )

        bucket.grant_read_write(handler)

        api = apigateway.RestApi(self, "widgets-api",
                  rest_api_name="Widget Service",
                  description="This service serves widgets.")

        get_widgets_integration = apigateway.LambdaIntegration(handler,
                request_templates={"application/json": '{ "statusCode": "200" }'})

        api.root.add_method("GET", get_widgets_integration)   # GET /
```

------
#### [ Java ]

File: `src/src/main/java/com/myorg/WidgetService.java`

```
package com.myorg;

import java.util.HashMap;

import software.constructs.Construct;
import software.amazon.awscdk.services.apigateway.LambdaIntegration;
import software.amazon.awscdk.services.apigateway.Resource;
import software.amazon.awscdk.services.apigateway.RestApi;
import software.amazon.awscdk.services.lambda.Code;
import software.amazon.awscdk.services.lambda.Function;
import software.amazon.awscdk.services.lambda.Runtime;
import software.amazon.awscdk.services.s3.Bucket;

public class WidgetService extends Construct {

    @SuppressWarnings("serial")
    public WidgetService(Construct scope, String id) {
        super(scope, id);

        Bucket bucket = new Bucket(this, "WidgetStore");

        Function handler = Function.Builder.create(this, "WidgetHandler")
            .runtime(Runtime.NODEJS_14_X)
            .code(Code.fromAsset("resources"))
            .handler("widgets.main")
            .environment(java.util.Map.of(   // Java 9 or later
               "BUCKET", bucket.getBucketName()) 
            .build();

        bucket.grantReadWrite(handler);
        
        RestApi api = RestApi.Builder.create(this, "Widgets-API")
                .restApiName("Widget Service").description("This service services widgets.")
                .build();

        LambdaIntegration getWidgetsIntegration = LambdaIntegration.Builder.create(handler) 
                .requestTemplates(java.util.Map.of(   // Map.of is Java 9 or later
                    "application/json", "{ \"statusCode\": \"200\" }"))
                .build();

        api.getRoot().addMethod("GET", getWidgetsIntegration);    
    }
}
```

------
#### [ C\# ]

File: `src/MyWidgetService/WidgetService.cs`

```
using Amazon.CDK;
using Amazon.CDK.AWS.APIGateway;
using Amazon.CDK.AWS.Lambda;
using Amazon.CDK.AWS.S3;
using System.Collections.Generic;
using constructs;

namespace MyWidgetService
{

    public class WidgetService : Construct
    {
        public WidgetService(Construct scope, string id) : base(scope, id)
        {
            var bucket = new Bucket(this, "WidgetStore");

            var handler = new Function(this, "WidgetHandler", new FunctionProps
            {
                Runtime = Runtime.NODEJS_14_X,
                Code = Code.FromAsset("resources"),
                Handler = "widgets.main",
                Environment = new Dictionary<string, string>
                {
                    ["BUCKET"] = bucket.BucketName
                }
            });

            bucket.GrantReadWrite(handler);

            var api = new RestApi(this, "Widgets-API", new RestApiProps
            {
                RestApiName = "Widget Service",
                Description = "This service services widgets."
            });

            var getWidgetsIntegration = new LambdaIntegration(handler, new LambdaIntegrationOptions
            {
                RequestTemplates = new Dictionary<string, string>
                {
                    ["application/json"] = "{ \"statusCode\": \"200\" }"
                }
            });

            api.Root.AddMethod("GET", getWidgetsIntegration);

        }
    }
}
```

------

**Tip**  
We're using a `[lambda\.Function](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda.Function.html)` in to deploy this function because it supports a wide variety of programming languages\. For JavaScript and TypeScript specifically, you might consider a `[lambda\-nodejs\.NodejsFunction](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_lambda_nodejs.NodejsFunction.html)`\. The latter uses esbuild to bundle up the script and converts code written in TypeScript automatically\.

Save the app and make sure it still synthesizes an empty stack\.

```
cdk synth
```

## Add the service to the app<a name="serverless_example_add_service"></a>

To add the widget service to our AWS CDK app, we'll need to modify the source file that defines the stack to instantiate the service construct\.

------
#### [ TypeScript ]

File: `lib/my_widget_service-stack.ts`

Add the following line of code after the existing `import` statement\.

```
import * as widget_service from '../lib/widget_service';
```

Replace the comment in the constructor with the following line of code\.

```
    new widget_service.WidgetService(this, 'Widgets');
```

------
#### [ JavaScript ]

File: `lib/my_widget_service-stack.js`

Add the following line of code after the existing `require()` line\.

```
const widget_service = require('../lib/widget_service');
```

Replace the comment in the constructor with the following line of code\.

```
    new widget_service.WidgetService(this, 'Widgets');
```

------
#### [ Python ]

File: `my_widget_service/my_widget_service_stack.py`

Add the following line of code after the existing `import` statement\.

```
from . import widget_service
```

Replace the comment in the constructor with the following line of code\.

```
        widget_service.WidgetService(self, "Widgets")
```

------
#### [ Java ]

File: `src/src/main/java/com/myorg/MyWidgetServiceStack.java`

Replace the comment in the constructor with the following line of code\.

```
new WidgetService(this, "Widgets");
```

------
#### [ C\# ]

File: `src/MyWidgetService/MyWidgetServiceStack.cs`

Replace the comment in the constructor with the following line of code\.

```
new WidgetService(this, "Widgets");
```

------

Be sure the app runs and synthesizes a stack \(we won't show the stack here: it's over 250 lines\)\.

```
cdk synth
```

## Deploy and test the app<a name="serverless_example_deploy_and_test"></a>

Before you can deploy your first AWS CDK app, you must bootstrap your AWS environment\. This creates \(among other resources\) a staging bucket that the AWS CDK uses to deploy stacks containing assets\. For details, see [Bootstrapping your AWS environment](cli.md#cli-bootstrap)\. If you've already bootstrapped, you'll get a warning and nothing will change\.

```
cdk bootstrap aws://ACCOUNT-NUMBER/REGION
```

Now we're ready to deploy the app as follows\.

```
cdk deploy
```

If the deployment succeeds, save the URL for your server\. This URL appears in one of the last lines in the window, where *GUID* is an alphanumeric GUID and *REGION* is your AWS Region\.

```
https://GUID.execute-api-REGION.amazonaws.com/prod/
```

Test your app by getting the list of widgets \(currently empty\) by navigating to this URL in a browser, or use the following command\.

```
curl -X GET 'https://GUID.execute-api.REGION.amazonaws.com/prod'
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

## Add the individual widget functions<a name="serverless_example_add_widget_functions"></a>

The next step is to create Lambda functions to create, show, and delete individual widgets\. 

Replace the code in `widgets.js` \(in `resources`\) with the following\.

```
const AWS = require('aws-sdk');
const S3 = new AWS.S3();

const bucketName = process.env.BUCKET;

/* 
This code uses callbacks to handle asynchronous function responses.
It currently demonstrates using an async-await pattern. 
AWS supports both the async-await and promises patterns.
For more information, see the following: 
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises
https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/calling-services-asynchronously.html
https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html 
*/
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
        body: data
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

Wire up these functions to your API Gateway code at the end of the `WidgetService` constructor\.

------
#### [ TypeScript ]

File: `lib/widget_service.ts`

```
    const widget = api.root.addResource("{id}");

    // Add new widget to bucket with: POST /{id}
    const postWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Get a specific widget from bucket with: GET /{id}
    const getWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Remove a specific widget from the bucket with: DELETE /{id}
    const deleteWidgetIntegration = new apigateway.LambdaIntegration(handler);

    widget.addMethod("POST", postWidgetIntegration); // POST /{id}
    widget.addMethod("GET", getWidgetIntegration); // GET /{id}
    widget.addMethod("DELETE", deleteWidgetIntegration); // DELETE /{id}
```

------
#### [ JavaScript ]

File: `lib/widget_service.js`

```
    const widget = api.root.addResource("{id}");

    // Add new widget to bucket with: POST /{id}
    const postWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Get a specific widget from bucket with: GET /{id}
    const getWidgetIntegration = new apigateway.LambdaIntegration(handler);

    // Remove a specific widget from the bucket with: DELETE /{id}
    const deleteWidgetIntegration = new apigateway.LambdaIntegration(handler);

    widget.addMethod("POST", postWidgetIntegration); // POST /{id}
    widget.addMethod("GET", getWidgetIntegration); // GET /{id}
    widget.addMethod("DELETE", deleteWidgetIntegration); // DELETE /{id}
```

------
#### [ Python ]

File: `my_widget_service/widget_service.py`

```
        widget = api.root.add_resource("{id}")

        # Add new widget to bucket with: POST /{id}
        post_widget_integration = apigateway.LambdaIntegration(handler)

        # Get a specific widget from bucket with: GET /{id}
        get_widget_integration = apigateway.LambdaIntegration(handler)

        # Remove a specific widget from the bucket with: DELETE /{id}
        delete_widget_integration = apigateway.LambdaIntegration(handler)

        widget.add_method("POST", post_widget_integration);     # POST /{id}
        widget.add_method("GET", get_widget_integration);       # GET /{id}
        widget.add_method("DELETE", delete_widget_integration); # DELETE /{id}
```

------
#### [ Java ]

File: `src/src/main/java/com/myorg/WidgetService.java`

```
        Resource widget = api.getRoot().addResource("{id}");

        // Add new widget to bucket with: POST /{id}
        LambdaIntegration postWidgetIntegration = new LambdaIntegration(handler);

        // Get a specific widget from bucket with: GET /{id}
        LambdaIntegration getWidgetIntegration = new LambdaIntegration(handler);

        // Remove a specific widget from the bucket with: DELETE /{id}
        LambdaIntegration deleteWidgetIntegration = new LambdaIntegration(handler);

        widget.addMethod("POST", postWidgetIntegration);     // POST /{id}
        widget.addMethod("GET", getWidgetIntegration);       // GET /{id}
        widget.addMethod("DELETE", deleteWidgetIntegration); // DELETE /{id}
```

------
#### [ C\# ]

File: `src/MyWidgetService/WidgetService.cs`

```
            var widget = api.Root.AddResource("{id}");

            // Add new widget to bucket with: POST /{id}
            var postWidgetIntegration = new LambdaIntegration(handler);

            // Get a specific widget from bucket with: GET /{id}
            var getWidgetIntegration = new LambdaIntegration(handler);

            // Remove a specific widget from the bucket with: DELETE /{id}
            var deleteWidgetIntegration = new LambdaIntegration(handler);

            widget.AddMethod("POST", postWidgetIntegration);        // POST /{id}
            widget.AddMethod("GET", getWidgetIntegration);          // GET /{id}
            widget.AdMethod("DELETE", deleteWidgetIntegration);    // DELETE /{id}
```

------

Save and deploy the app\.

```
cdk deploy
```

We can now store, show, or delete an individual widget\. Use the following commands to list the widgets, create the widget **example**, list all of the widgets, show the contents of **example** \(it should show today's date\), delete **example**, and then show the list of widgets again\.

```
curl -X GET 'https://GUID.execute-api.REGION.amazonaws.com/prod'
curl -X POST 'https://GUID.execute-api.REGION.amazonaws.com/prod/example'
curl -X GET 'https://GUID.execute-api.REGION.amazonaws.com/prod'
curl -X GET 'https://GUID.execute-api.REGION.amazonaws.com/prod/example'
curl -X DELETE 'https://GUID.execute-api.REGION.amazonaws.com/prod/example'
curl -X GET 'https://GUID.execute-api.REGION.amazonaws.com/prod'
```

You can also use the API Gateway console to test these functions\. Set the **name** value to the name of a widget, such as **example**\.

## Clean up<a name="serverless_example_destroy"></a>

To avoid unexpected AWS charges, destroy your AWS CDK stack after you're done with this exercise\.

```
cdk destroy
```