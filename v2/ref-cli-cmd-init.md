# cdk init<a name="ref-cli-cmd-init"></a>

Create a new AWS CDK project from a template\.

## Usage<a name="ref-cli-cmd-init-usage"></a>

```
$ cdk init <arguments> <options>
```

## Arguments<a name="ref-cli-cmd-init-args"></a>

**Template type**  <a name="ref-cli-cmd-init-args-template-type"></a>
The CDK template type to initialize a new CDK project from\.  
+ `app` – Template for a CDK application\.
+ `lib` – Template for an AWS Construct Library\.
+ `sample-app` – Example CDK application that includes some constructs\.
*Valid values*: `app`, `lib`, `sample-app`

## Options<a name="ref-cli-cmd-init-options"></a>

For a list of global options that work with all CDK CLI commands, see [Global options](ref-cli-cmd.md#ref-cli-cmd-options)\.

`--generate-only BOOLEAN`  <a name="ref-cli-cmd-init-options-generate-only"></a>
Specify this option to generate project files without initiating additional operations such as setting up a git repository, installing dependencies, or compiling the project\.  
*Default value*: `false`

`--help, -h BOOLEAN`  <a name="ref-cli-cmd-init-options-help"></a>
Show command reference information for the `cdk init command`\.

`--language, -l STRING`  <a name="ref-cli-cmd-init-options-language"></a>
The language to be used for the new project\. This option can be configured in the project’s `cdk.json` configuration file or at `~/.cdk.json` on your local development machine\.  
*Valid values*: `csharp`, `fsharp`, `go`, `java`, `javascript`, `python`, `typescript`

`--list BOOLEAN`  <a name="ref-cli-cmd-init-options-list"></a>
List the available template types and languages\.

## Examples<a name="ref-cli-cmd-init-examples"></a>

### List the available template types and languages<a name="ref-cli-cmd-init-examples-1"></a>

```
$ cdk init --list
Available templates:
* app: Template for a CDK Application
   └─ cdk init app --language=[csharp|fsharp|go|java|javascript|python|typescript]
* lib: Template for a CDK Construct Library
   └─ cdk init lib --language=typescript
* sample-app: Example CDK Application with some constructs
   └─ cdk init sample-app --language=[csharp|fsharp|go|java|javascript|python|typescript]
```

### Create a new CDK app in TypeScript from the library template<a name="ref-cli-cmd-init-examples-2"></a>

```
$ cdk init lib --language=typescript
```