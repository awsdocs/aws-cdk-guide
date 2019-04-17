--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Get a Value from a Context Variable<a name="get_context_var"></a>

You can specify a context variable either as part of a CDK Toolkit command, or in `cdk.json`\.

To create a command line context variable, use the **\-\-context** \(**\-c**\) option of a CDK Toolkit command, as shown in the following example\.

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

To get the value of a context variable in your app, use code like the following, which gets the value of the context variable **bucket\_name**\.

```
const bucket_name = this.node.getContext("bucket_name");
```