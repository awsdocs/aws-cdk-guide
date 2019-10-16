# AWS CDK in Other Languages<a name="multiple_languages"></a>

In some cases the example code in the AWS CDK documentation is available only in TypeScript\. This topic describes how to read TypeScript code and translate it into Python\. This is currently the only other [stable](reference.md#aws_construct_lib_versioning_binding) programming language that the AWS CDK supports \(the AWS CDK supports a developer preview version of Java and C\#/\.NET\)\. See [Hello World Tutorial](getting_started.md#hello_world_tutorial) for an example of creating an AWS CDK app in a supported language\.

## Importing a Module<a name="multiple_languages_import"></a>

Both TypeScript and Python support namespaced module imports and selective imports\. Module names in Python look like **aws\_cdk\.***xxx*, where *xxx* represents an AWS service name, such as **s3** for Amazon S3 \(we'll use Amazon S3 for our examples\)\. Replace the dashes in the TypeScript module name with underscores to get the Python module name\. 

The following is how you import the entire Amazon S3 module or just a `Stack` class in both languages\.


| TypeScript | Python | 
| --- |--- |
| <pre>// Import entire module<br />import s3 = require('@aws-cdk/aws-s3')<br /><br />// Selective import<br />import { Stack } from '@aws-cdk/core';</pre> | <pre># Import entire module<br />import aws_cdk.aws_s3 as s3<br /><br /># Selective import<br />from aws_cdk.core import Stack</pre>  | 

## Instantiating a Class<a name="multiple_languages_class"></a>

Classes have the same name in TypeScript and in Python\. TypeScript uses **new** to instantiate classes, whereas in Python you call the class object directly, like a function\. Props objects at the end of an argument list become keyword\-only arguments in Python, and their names become `snake_case`\. The keyword **this** in TypeScript translates to **self** in Python\. 

The following table shows how you can translate TypeScript class instantiations to Python class instantiations\.


| TypeScript | Python | 
| --- |--- |
| <pre>// Instantiate Bucket class<br />const bucket = new s3.Bucket(this, 'Bucket');</pre> | <pre># Instantiate Bucket class<br />bucket = s3.Bucket(self, 'Bucket')</pre> | 
| <pre>// Instantiate Bucket with props<br />const bucket = new s3.Bucket(this, 'Bucket', {<br />  bucketName: 'my-bucket',<br />   versioned: true,<br />});</pre> | <pre># Instantiate Bucket with props<br />bucket = s3.Bucket(self, 'Bucket', <br />  bucket_name='my-bucket',<br />  versioned=True)</pre> | 

## Methods<a name="multiple_languages_methods"></a>

Methods names and argument names in TypeScript are `camelCased`, whereas in Python they are `snake_cased`\. Props objects at the end of an argument list in TypeScript are translated into keyword\-only arguments in Python, and their names become `snake_case`\.

The following table shows how you can translate TypeScript methods to Python methods\.


| TypeScript | Python | 
| --- |--- |
| <pre>// Call method<br />bucket.addCorsRule({<br />  allowedOrigins: ['*'],<br />  allowedMethods: [],<br />});</pre> | <pre># Call method<br />bucket.add_cors_rule(<br />  allowed_origins=['*'],<br />  allowed_methods=[]<br />)</pre> | 

## Enum Constants<a name="multiple_languages_enums"></a>

Enum constants are scoped to a class, and have uppercase names with underscores in both languages \(sometimes referred to as `SCREAMING_SNAKE_CASE`\)\. TypeScript enum constants and Python enum constants are identical\.


| TypeScript | Python | 
| --- |--- |
| <pre>s3.BucketEncryption.KMS_MANAGED</pre> | <pre>s3.BucketEncryption.KMS_MANAGED</pre> | 

## Defining Constructs<a name="multiple_languages_constructs"></a>

In TypeScript, a construct's props are defined with a shape\-based interface, whereas Python takes keyword \(or keyword\-only, see [PEP3102](https://www.python.org/dev/peps/pep-3102/)\) arguments\. 

The following table shows how TypeScript construct definitions translate to Python construct definitions\.


| TypeScript | Python | 
| --- |--- |
| <pre>interface MyConstructProps {<br />  prop1: number;<br />  prop2?: number;<br />}<br /><br />class MyConstruct extends Construct {<br />  constructor(scope: Construct, id: string, props: MyConstructProps) {<br />     super(scope, id);<br />     const prop2 = props.prop2 !== undefined ? props.prop2 : 10;<br /><br />    // Construct contents here<br /><br />  }<br />}</pre> | <pre>class MyConstruct(Construct):<br /><br />  def __init__(scope, id, *, prop1, prop2=10):<br />    super().__init__(scope, id)<br /><br />    # Construct contents here</pre>  | 

## Structs \(Interfaces\)<a name="multiple_languages_structs"></a>

Structs are TypeScript interfaces that represent a set of values\. You can recognize them because their name doesn't start with an `I`, and all of their fields are **read\-only**\.

In TypeScript, structs are passed as object literals\. In Python, if the struct is the last argument to a method, its fields are lifted into the method call itself\. If the argument list contains nested structs, wrap them in a class named after the struct\.

The following table shows how to call a method with two levels of structs\.


| TypeScript | Python | 
| --- |--- |
| <pre>bucket.addLifecycleRule({<br />  transitions: [<br />    {<br />      storageClass: StorageClass.GLACIER,<br />      transitionAfter: Duration.days(10)<br />    }<br />  ]<br />});<br />          </pre> | <pre>bucket.add_lifecycle_rule(<br />  transitions=[<br />    Transition(<br />      storage_class=StorageClass.GLACIER,<br />      transition_after=Duration.days(10)<br />    )<br />  ]<br />)</pre> | 

## Object Interfaces<a name="multiple_languages_object"></a>

The AWS CDK uses TypeScript object interfaces to indicate that a class implements an expected set of methods and properties\. You can recognize an object interface because its name starts with `I`\.

Typically, Python users don't explicitly indicate that a class implements an interface\. However, for the AWS CDK you can do this by decorating your class with `@jsii.implements(interface)`\. 


| TypeScript | Python | 
| --- |--- |
| <pre>import {IAspect, IConstruct } from '@aws-cdk/core';<br /><br />class MyAspect implements IAspect {<br />  public visit(node: IConstruct) {<br />    console.log('Visited', node.node.path);<br />  }<br />}</pre> | <pre>from aws_cdk.core import IAspect, IConstruct<br /><br />@jsii.implements(IAspect)<br />class MyAspect():<br />  def visit(self, node: IConstruct) -> None:<br />    print("Visited", node.node.path)</pre> | 