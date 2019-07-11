# Get a Value from a Context Variable<a name="get_context_var"></a>

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

To get the value of a context variable in your app, use code like the following, which gets the value of the context variable **bucket\_name**\.

```
const bucket_name = this.node.tryGetContext("bucket_name");
```