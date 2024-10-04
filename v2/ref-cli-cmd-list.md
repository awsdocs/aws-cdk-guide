# cdk list<a name="ref-cli-cmd-list"></a>

List all AWS CDK stacks and their dependencies from a CDK app\.

## Usage<a name="ref-cli-cmd-list-usage"></a>

```
$ cdk list <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-list-args"></a>

**CDK stack ID**  <a name="ref-cli-cmd-list-args-stack-name"></a>
The construct ID of the CDK stack from your app to perform this command against\.  
*Type*: String  
*Required*: No

## Options<a name="ref-cli-cmd-list-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-list-options-help"></a>
Show command reference information for the `cdk list` command\.

`--long, -l BOOLEAN`  <a name="ref-cli-cmd-list-options-long"></a>
Display AWS environment information for each stack\.  
*Default value*: `false`

`--show-dependencies, -d BOOLEAN`  <a name="ref-cli-cmd-list-options-show-dependencies"></a>
Display stack dependency information for each stack\.  
*Default value*: `false`

## Examples<a name="ref-cli-cmd-list-examples"></a>

### List all stacks in the CDK app ‘node bin/main\.js’<a name="ref-cli-cmd-list-examples-1"></a>

```
$ cdk list --app='node bin/main.js'
Foo
Bar
Baz
```

### List all stacks, including AWS environment details for each stack<a name="ref-cli-cmd-list-examples-"></a>

```
$ cdk list --app='node bin/main.js' --long
-
    name: Foo
    environment:
        name: 000000000000/bermuda-triangle-1
        account: '000000000000'
        region: bermuda-triangle-1
-
    name: Bar
    environment:
        name: 111111111111/bermuda-triangle-2
        account: '111111111111'
        region: bermuda-triangle-2
-
    name: Baz
    environment:
        name: 333333333333/bermuda-triangle-3
        account: '333333333333'
        region: bermuda-triangle-3
```