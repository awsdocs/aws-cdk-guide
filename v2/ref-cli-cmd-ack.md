# cdk acknowledge<a name="ref-cli-cmd-ack"></a>

Acknowledge a notice by issue number and hide it from displaying again\.

This is useful to hide notices that have been addressed or do not apply to you\.

Acknowledgements are saved at a CDK project level\. If you acknowledge a notice in one CDK project, it will still display in other projects until acknowledged there\.

## Usage<a name="ref-cli-cmd-ack-usage"></a>

```
$ cdk acknowledge <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-ack-args"></a>

**Notice ID**  <a name="ref-cli-cmd-ack-args-notice-id"></a>
The ID of the notice\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-ack-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-ack-options-help"></a>
Show command reference information for the `cdk acknowledge` command\.

## Examples<a name="ref-cli-cmd-ack-examples"></a>

### Acknowledge and hide a notice that displays when running another CDK CLI command<a name="ref-cli-cmd-ack-examples-1"></a>

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
        
$ cdk acknowledge 16603
```