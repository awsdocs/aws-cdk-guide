--------

This documentation is for the developer preview release \(public beta\) of the AWS Cloud Development Kit \(AWS CDK\)\. Releases might lack important features and might have future breaking changes\.

--------

# Get a Value from an Environment Variable<a name="get_env_var"></a>

To get the value of an environment variable, use code such as the following\. This code gets the value of the `MYBUCKET` environment variable\.

```
const bucket_name = process.env.MYBUCKET;
```