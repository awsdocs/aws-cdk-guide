# Save and retrieve context variable values<a name="get_context_var"></a>

You can specify context variables with the AWS Cloud Development Kit \(AWS CDK\) CLI or in the `cdk.json` file\. Then, use the `TryGetContext` method to retrieve values\.

**Topics**
+ [Specify context variables](#develop-context-specify)
+ [Retrieve context variable values](#develop-context-retrieve)

## Specify context variables<a name="develop-context-specify"></a>

You can specify a context variable either as part of an AWS CDK CLI command, or in `cdk.json`\.

To create a command line context variable, use the **\-\-context** \(**\-c**\) option, as shown in the following example\.

```
cdk synth -c bucket_name=mygroovybucket
```

To specify the same context variable and value in the `cdk.json` file, use the following code\.

```
{
  "context": {
    "bucket_name": "myotherbucket"
  }
}
```

If you specify a context variable using both the AWS CDK CLI and `cdk.json` file, the AWS CDK CLI value takes precedence\.

## Retrieve context variable values<a name="develop-context-retrieve"></a>

To get the value of a context variable in your app, use the `TryGetContext` method in the context of a construct\. \(That is, when `this`, or `self` in Python, is an instance of some construct\.\)

In this example, we retrieve the value of the `bucket_name` context variable\. If the requested value is not defined, `TryGetContext` returns `undefined` \(`None` in Python; `null` in Java and C\#; `nil` in Go\) rather than raising an exception\.

------
#### [ TypeScript ]

```
const bucket_name = this.node.tryGetContext('bucket_name');
```

------
#### [ JavaScript ]

```
const bucket_name = this.node.tryGetContext('bucket_name');
```

------
#### [ Python ]

```
bucket_name = self.node.try_get_context("bucket_name")
```

------
#### [ Java ]

```
String bucketName = (String)this.getNode().tryGetContext("bucket_name");
```

------
#### [ C\# ]

```
var bucketName = this.Node.TryGetContext("bucket_name");
```

------

Outside the context of a construct, you can access the context variable from the app object, like this\.

------
#### [ TypeScript ]

```
const app = new cdk.App();
const bucket_name = app.node.tryGetContext('bucket_name')
```

------
#### [ JavaScript ]

```
const app = new cdk.App();
const bucket_name = app.node.tryGetContext('bucket_name');
```

------
#### [ Python ]

```
app = cdk.App()
bucket_name = app.node.try_get_context("bucket_name")
```

------
#### [ Java ]

```
App app = App();
String bucketName = (String)app.getNode().tryGetContext("bucket_name");
```

------
#### [ C\# ]

```
app = App();
var bucketName = app.Node.TryGetContext("bucket_name");
```

------

For more details on working with context variables, see [Runtime context](context.md)\.