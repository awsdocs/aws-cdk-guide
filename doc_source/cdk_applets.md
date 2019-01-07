--------

 This documentation is for the developer preview release of the AWS CDK\. Do not use this version of the AWS CDK in production\. Subsequent releases of the AWS CDK will likely include breaking changes\. 

--------

# Applets<a name="cdk_applets"></a>

**Note**  
The AWS CDK only supports applets published as JavaScript modules\.

Applets are YAML files that directly instantiate constructs, without writing any code\. The structure of an applet file looks like the following:

```
applets:
  Applet1:
    type: MODULE[:CLASS]
    properties:
      property1: value1
      property2: value2
      ...
  Applet2:
    type: MODULE[:CLASS]
    properties:
      ...
```

Every applet will be synthesized to its own stack, named after the key used in the applet definition\.

## Specifying the Applet to Load<a name="cdk_applets_specifying"></a>

An applet `type` specification looks like the following:

```
applet: MODULE[:CLASS]
```

**MODULE** can be used to indicate:
+ A local file, such as `./my-module` \(expects `my-module.js` in the same directory\)\.
+ A local module such as `my-dependency` \(expects an NPM package at `node_modules/my-dependency`\)\.
+ A global module, such as `@aws-cdk/aws-s3` \(expects the package to have been globally installed using NPM\)\.
+ An NPM package, such as `npm://some-package@1.2.3` \(the version specifier may be omitted to refer to the latest version of the package\)\.

**CLASS** should reference the name of a class exported by the indicated module\. If the class name is omitted, `Applet` is used as the default class name\.

## Properties<a name="cdk_applets_properties"></a>

Pass properties to the applet by specifying them in the `properties` object\. The properties will be passed to the instantiation of the class in the `type` parameter\.

## Running<a name="cdk_applets_running"></a>

To run an applet, pass its YAML file directly as the `--app` argument to a `cdk` invocation:

```
cdk --app ./my-applet.yaml deploy
```

To avoid needing to specify `--app` for every invocation, make a `cdk.json` file and add in the application in the config as usual:

```
{
  "app": "./my-applet.yaml"
}
```