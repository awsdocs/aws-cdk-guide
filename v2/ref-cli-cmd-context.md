# cdk context<a name="ref-cli-cmd-context"></a>

Manage cached context values for your AWS CDK application\.

*Context* represents the configuration and environment information that can influence how your stacks are synthesized and deployed\. Use `cdk context` to do the following:
+ View your configured context values\.
+ Set and manage context values\.
+ Remove context values\.

## Usage<a name="ref-cli-cmd-context-usage"></a>

```
$ cdk context <options>
```

## Options<a name="ref-cli-cmd-context-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--clear BOOLEAN`  <a name="ref-cli-cmd-context-options-clear"></a>
Clear all context\.

`--force, -f BOOLEAN`  <a name="ref-cli-cmd-context-options-force"></a>
Ignore missing key error\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-context-options-help"></a>
Show command reference information for the `cdk context` command\.

`--reset, -e STRING`  <a name="ref-cli-cmd-context-options-reset"></a>
The context key, or its index, to reset\.