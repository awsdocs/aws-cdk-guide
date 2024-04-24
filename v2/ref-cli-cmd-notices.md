# cdk notices<a name="ref-cli-cmd-notices"></a>

Display notices for your CDK application\.

Notices can include important messages regarding security vulnerabilities, regressions, and usage of unsupported versions\.

This command displays relevant notices, regardless of whether they have been acknowledged or not\. Relevant notices may also appear after every command by default\.

You can suppress notices in the following ways:
+ Through command options\. The following is an example:

  ```
  $ cdk deploy --no-notices
  ```
+ Suppress all notices indefinitely through context in the project’s `cdk.json` file:

  ```
  {
    "notices": false,
    "context": {
      // ...
    }
  }
  ```
+ Acknowledge each notice with the `cdk acknowledge` command\.

## Usage<a name="ref-cli-cmd-notices-usage"></a>

```
$ cdk notices <options>
```

## Options<a name="ref-cli-cmd-notices-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-notices-options-help"></a>
Show command reference information for the `cdk notices` command\.

## Examples<a name="ref-cli-cmd-notices-examples"></a>

### Example of a default notice that displays after running the cdk deploy command<a name="ref-cli-cmd-notices-examples-1"></a>

```
$ cdk deploy

... # Normal output of the command

NOTICES

16603   Toggling off auto_delete_objects for Bucket empties the bucket

        Overview: If a stack is deployed with an S3 bucket with
                  auto_delete_objects=True, and then re-deployed with
                  auto_delete_objects=False, all the objects in the bucket
                  will be deleted.

        Affected versions: <1.126.0.

        More information at: https://github.com/aws/aws-cdk/issues/16603


17061   Error when building EKS cluster with monocdk import

        Overview: When using monocdk/aws-eks to build a stack containing
                  an EKS cluster, error is thrown about missing
                  lambda-layer-node-proxy-agent/layer/package.json.

        Affected versions: >=1.126.0 <=1.130.0.

        More information at: https://github.com/aws/aws-cdk/issues/17061


If you don’t want to see an notice anymore, use "cdk acknowledge ID". For example, "cdk acknowledge 16603"
```

### Simple example of running the cdk notices command<a name="ref-cli-cmd-notices-examples-1"></a>

```
$ cdk notices

NOTICES

16603   Toggling off auto_delete_objects for Bucket empties the bucket

        Overview: if a stack is deployed with an S3 bucket with
                  auto_delete_objects=True, and then re-deployed with
                  auto_delete_objects=False, all the objects in the bucket
                  will be deleted.

        Affected versions: framework: <=2.15.0 >=2.10.0

        More information at: https://github.com/aws/aws-cdk/issues/16603


If you don’t want to see a notice anymore, use "cdk acknowledge <id>". For example, "cdk acknowledge 16603"
```