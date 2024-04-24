# cdk docs<a name="ref-cli-cmd-docs"></a>

Open AWS CDK documentation in your browser\.

## Usage<a name="ref-cli-cmd-docs-usage"></a>

```
$ cdk docs <options>
```

## Options<a name="ref-cli-cmd-docs-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--browser, -b STRING`  <a name="ref-cli-cmd-docs-options-browser"></a>
The command to use to open the browser, using `%u` as a placeholder for the path of the file to open\.  
*Default value*: `open %u`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-docs-options-help"></a>
Show command reference information for the `cdk docs` command\.

## Examples<a name="ref-cli-cmd-docs-examples"></a>

### Open AWS CDK documentation in Google Chrome<a name="ref-cli-cmd-docs-examples-1"></a>

```
$ cdk docs --browser='chrome %u'
```