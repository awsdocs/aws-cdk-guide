# AWS CDK in Other Languages<a name="multiple_languages"></a>

In some cases the example code in the AWS CDK documentation is available only in TypeScript\. This topic describes how to read TypeScript code and translate it into Python\. This is currently the only other [stable](reference.md#aws_construct_lib_versioning_binding) programming language that the AWS CDK supports \(the AWS CDK supports a developer preview version of Java and C\#/\.NET\)\. See [Hello World Tutorial](getting_started.md#hello_world_tutorial) for an example of creating an AWS CDK app in a supported language\.

## Importing a Module<a name="multiple_languages_import"></a>

Both TypeScript and Python support namespaced module imports and selective imports\. Module names in Python look like **aws\_cdk\.***xxx*, where *xxx* represents an AWS service name, such as **s3** for Amazon S3 \(we'll use Amazon S3 for our examples\)\. Replace the dashes in the TypeScript module name with underscores to get the Python module name\. 

The following is how you import the entire Amazon S3 module or just a `Stack` class in both languages\.


| TypeScript | Python | 
| --- |--- |
| // Import entire module | \# Import entire module | 
| import s3 = require\('@aws\-cdk/aws\-s3'\) | import aws\_cdk\.aws\_s3 as s3 | 
|  |  | 
| // Selective import | \# Selective import | 
| import \{ Stack \} from '@aws\-cdk/core'; | from aws\_cdk\.core import Stack | 

## Instantiating a Class<a name="multiple_languages_class"></a>

Classes have the same name in TypeScript and in Python\. TypeScript uses **new** to instantiate classes, whereas in Python you call the class object directly\. The keyword **this** in TypeScript translates to **self** in Python\. 

The following table shows how you can translate TypeScript class instantiations to Python class instantiations\.


| TypeScript | Python | 
| --- |--- |
| // Instantiate Bucket class | \# Instantiate Bucket class | 
| new s3\.Bucket\(this, 'Bucket'\); | s3\.Bucket\(self, 'Bucket'\) | 

## Methods<a name="multiple_languages_methods"></a>

Methods names and argument names in TypeScript are `camelCased`, whereas in Python they are `snake_cased`\. Props objects at the end of an argument list in TypeScript are translated into keyword\-only arguments in Python\. 

The following table shows how you can translate TypeScript methods to Python methods\.


| TypeScript | Python | 
| --- |--- |
| // Instantiate Bucket with props | \# Instantiate Bucket with props | 
| const bucket = new s3\.Bucket\(this, 'Bucket', \{ | bucket = s3\.Bucket\(self, 'Bucket',  | 
|   bucketName: 'my\-bucket', |   bucketName='my\-bucket',  | 
|   versioned: true, |   versioned=True\) | 
| \}\); |  | 
|  |  | 
| // Call method | \# Call method | 
| bucket\.addCorsRule\(\{ | bucket\.add\_cors\_rule\( | 
|   allowedOrigins: \['\*'\], |   allowed\_origins=\['\*'\], | 
|   allowedMethods: \[\], |   allowed\_methods=\[\] | 
| \}\); | \) | 

## Enum Constants<a name="multiple_languages_enums"></a>

Enum constants are scoped to a class, and have uppercase names in both languages\. 

The following table shows how you can translate TypeScript enum constants to Python enum constants\.


| TypeScript | Python | 
| --- |--- |
| s3\.BucketEncryption\.KMS\_MANAGED | s3\.BucketEncryption\.KMS\_MANAGED | 

## Defining Constructs<a name="multiple_languages_constructs"></a>

In TypeScript a construct’s props are defined with an interface, whereas in Python you take keyword \(or keyword\-only, see [PEP3102](https://www.python.org/dev/peps/pep-3102/)\) arguments\. 

The following table shows how you can translate TypeScript construct definitions to Python construct definitions\.


| TypeScript | Python | 
| --- |--- |
| interface MyConstructProps \{ |  | 
|   prop1: number; |  | 
|   prop2?: number; |  | 
| \} |  | 
|  |  | 
| class MyConstruct extends Construct \{ | class MyConstruct\(Construct\): | 
|   constructor\(scope: Construct, id: string, props: MyConstructProps\) \{ |   def \_\_init\_\_\(scope, id, \*, prop1, prop2=10\): | 
|     super\(scope, id\); |   super\(\)\.\_\_init\_\_\(scope, id\) | 
|  |  | 
|     const prop2 = props\.prop2 \!== undefined ? props\.prop2 : 10; |  | 
|  |  | 
|     // Construct contents here |   \# Construct contents here | 

## Structs \(Interfaces\)<a name="multiple_languages_structs"></a>

Structs are TypeScript interfaces that represent a set of values\. You can recognize them because their name doesn't start with an `I`, and all of their fields are **read\-only**\.

In TypeScript, structs are passed as object literals\. In Python, if the struct is the last argument to a method, its fields are lifted into the method call itself\. If the argument list contains nested structs, wrap them in a class named after the struct\.

The following table shows how to call a method with two levels of structs\.


| TypeScript | Python | 
| --- |--- |
| bucket\.addLifecycleRule\(\{ | bucket\.add\_lifecycle\_rule\( | 
|   transitions: \[ |   transitions=\[ | 
|     \{ |     Transition\( | 
|       storageClass: StorageClass\.GLACIER, |       storage\_class=StorageClass\.GLACIER, | 
|       transitionAfter: Duration\.days\(10\) |       transition\_after=Duration\.days\(10\) | 
|     \} |     \) | 
|   \] |   \] | 
| \}\); | \) | 

## Object Interfaces<a name="multiple_languages_object"></a>

The AWS CDK uses TypeScript object interfaces to indicate that a class implements an expected set of methods and properties\. You can recognize an object interface because its name starts with `I`\.

Typically, Python users don’t explicitly indicate that a class implements an interface\. However, for the AWS CDK you can do this by decorating your class with `@jsii.implements(interface)`\. 


| TypeScript | Python | 
| --- |--- |
| import \{IAspect, IConstruct \} from ‘@aws\-cdk/core’; | from aws\_cdk\.core import IAspect, IConstruct | 
|  |  | 
|  | @jsii\.implements\(IAspect\) | 
| class MyAspect implements IAspect \{ | class MyAspect\(\): | 
|   public visit\(node: IConstruct\) \{ |   def visit\(self, node: IConstruct\) \-> None: | 
|     console\.log\(‘Visited’, node\.node\.path\); |     print\("Visited”, node\.node\.path\) | 
|   \} |  | 
| \} |  | 