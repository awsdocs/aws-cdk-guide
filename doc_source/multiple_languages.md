# AWS CDK in Other Languages<a name="multiple_languages"></a>

In some cases the example code in the AWS CDK documentation is available only in TypeScript\. This topic describes how to read TypeScript code and translate it into other languages\. See [Hello World Tutorial](getting_started.md#hello_world_tutorial) for an example of creating an AWS CDK app in a supported language\.

## Importing a Module (Package)<a name="multiple_languages_import"></a>

All TypeScript, Python and Java support namespaced module (or package) imports and selective imports\. Module names in Python look like **aws\_cdk\.***xxx*, where *xxx* represents an AWS service name, such as **s3** for Amazon S3 \(we'll use Amazon S3 for our examples\)\. Replace the dashes in the TypeScript module name with underscores to get the Python module name\. Replace the `@aws-cdk` in the TypeScript module name with `software.amazon.awscdk` to get the Java package name.

The following is how you import the entire Amazon S3 module or just a `Stack` class in both languages\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>// Import entire module<br />import s3 = require('@aws-cdk/aws-s3')<br /><br />// Selective import<br />import { Stack } from '@aws-cdk/core';</pre> | <pre># Import entire module<br />import aws_cdk.aws_s3 as s3<br /><br /># Selective import<br />from aws_cdk.core import Stack</pre>  | <pre>// Import entire package<br />import software.amazon.awscdk.services.s3.*;<br /><br />// Selective import<br />import software.amazon.awscdk.core.Stack;<br /></pre> |

## Instantiating a Class<a name="multiple_languages_class"></a>

Classes have the same name in TypeScript, in Python and in Java\. TypeScript and Java uses **new** to instantiate classes, whereas in Python you call the class object directly, like a function\. Props objects at the end of an argument list become keyword\-only arguments in Python, and their names become `snake_case`\. The keyword **this** in TypeScript translates to **self** in Python\. 

The following table shows how you can translate TypeScript class instantiations to Python or Java class instantiations\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>// Instantiate Bucket class<br />const bucket = new s3.Bucket(this, 'Bucket');</pre> | <pre># Instantiate Bucket class<br />bucket = s3.Bucket(self, 'Bucket')</pre> | <pre>// Instantiate Bucket class<br />Bucket bucket = new Bucket(this, "Bucket");</pre> |
| <pre>// Instantiate Bucket with props<br />const bucket = new s3.Bucket(this, 'Bucket', {<br />  bucketName: 'my-bucket',<br />   versioned: true,<br />});</pre> | <pre># Instantiate Bucket with props<br />bucket = s3.Bucket(self, 'Bucket', <br />  bucket_name='my-bucket',<br />  versioned=True)</pre> | <pre>// Instantiate Bucket with props<br />Bucket bucket = new Bucket(this, "Bucket", BucketProps.builder()<br />  .bucketName("my-bucket")<br />  .versioned(true)<br />  .build());</pre> |

## Methods<a name="multiple_languages_methods"></a>

Methods and argument names in TypeScript or Java are `camelCased`, whereas in Python they are `snake_cased`\. Props objects at the end of an argument list in TypeScript are translated into keyword\-only arguments in Python, and their names become `snake_case`\. Java used [Builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) to generate props object.

The following table shows how you can translate TypeScript methods to Python methods or Java methods\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>// Call method<br />bucket.addCorsRule({<br />  allowedOrigins: ['*'],<br />  allowedMethods: [],<br />});</pre> | <pre># Call method<br />bucket.add_cors_rule(<br />  allowed_origins=['*'],<br />  allowed_methods=[]<br />)</pre> | <pre>// Call method<br />bucket.addCorsRule(CorsRule.builder()<br />  .allowedOrigins(List.of("*"))<br />  .allowedMethods(List.of())<br />  .build());</pre> |

## Enum Constants<a name="multiple_languages_enums"></a>

Enum constants are scoped to a class, and have uppercase names with underscores in all languages \(sometimes referred to as `SCREAMING_SNAKE_CASE`\)\. TypeScript enum constants, Python enum constants and Java enum constants are identical\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>s3.BucketEncryption.KMS_MANAGED</pre> | <pre>s3.BucketEncryption.KMS_MANAGED</pre> | <pre>BucketEncryption.KMS_MANAGED</pre> |

## Defining Constructs<a name="multiple_languages_constructs"></a>

In TypeScript, a construct's props are defined with a shape\-based interface, whereas Python takes keyword \(or keyword\-only, see [PEP3102](https://www.python.org/dev/peps/pep-3102/)\) arguments\. Java's props object should be a POJO and provided [Builder pattern](https://en.wikipedia.org/wiki/Builder_pattern) to define props object\. (Using [Lombok](https://projectlombok.org/) will be most convenient for you.)

The following table shows how TypeScript construct definitions translate to Python construct definitions or Java construct definitions\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>interface MyConstructProps {<br />  prop1: number;<br />  prop2?: number;<br />}<br /><br />class MyConstruct extends Construct {<br />  constructor(scope: Construct, id: string, props: MyConstructProps) {<br />     super(scope, id);<br />     const prop2 = props.prop2 !== undefined ? props.prop2 : 10;<br /><br />    // Construct contents here<br /><br />  }<br />}</pre> | <pre>class MyConstruct(Construct):<br /><br />  def __init__(scope, id, *, prop1, prop2=10):<br />    super().__init__(scope, id)<br /><br />    # Construct contents here</pre>  | <pre>@lombok.Getter<br />@lombok.Builder<br />public class MyConstructProps {<br />  private int prop1;<br />  @lombok.Builder.Default<br />  private int prop2 = 10;<br />}<br /><br />public class MyConstruct extends Construct {<br />  public MyConstruct(Construct scope, String id, MyConstructProps props) {<br />    super(scope, id);<br /><br />    // Construct contents here<br />  }<br />}</pre> |

## Structs \(Interfaces\)<a name="multiple_languages_structs"></a>

Structs are TypeScript interfaces that represent a set of values\. You can recognize them because their name doesn't start with an `I`, and all of their fields are **read\-only**\.

In TypeScript, structs are passed as object literals\. In Python, if the struct is the last argument to a method, its fields are lifted into the method call itself\. If the argument list contains nested structs, wrap them in a class named after the struct\. In Java, structs are passed as object directly\.

The following table shows how to call a method with two levels of structs\.


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>bucket.addLifecycleRule({<br />  transitions: [<br />    {<br />      storageClass: StorageClass.GLACIER,<br />      transitionAfter: Duration.days(10)<br />    }<br />  ]<br />});<br />          </pre> | <pre>bucket.add_lifecycle_rule(<br />  transitions=[<br />    Transition(<br />      storage_class=StorageClass.GLACIER,<br />      transition_after=Duration.days(10)<br />    )<br />  ]<br />)</pre> | <pre>bucket.addLifecycleRule(LifecycleRule.builder()<br />.transitions(List.of(Transition.builder()<br />  .storageClass(StorageClass.GLACIER)<br />  .transitionAfter(Duration.days(10))<br />  .build()))<br />.build());</pre> |

## Object Interfaces<a name="multiple_languages_object"></a>

The AWS CDK uses TypeScript and Java object interfaces to indicate that a class implements an expected set of methods and properties\. You can recognize an object interface because its name starts with `I`\.

Typically, Python users don't explicitly indicate that a class implements an interface\. However, for the AWS CDK you can do this by decorating your class with `@jsii.implements(interface)`\. 


| TypeScript | Python | Java |
| --- |--- | --- |
| <pre>import {IAspect, IConstruct } from '@aws-cdk/core';<br /><br />class MyAspect implements IAspect {<br />  public visit(node: IConstruct) {<br />    console.log('Visited', node.node.path);<br />  }<br />}</pre> | <pre>from aws_cdk.core import IAspect, IConstruct<br /><br />@jsii.implements(IAspect)<br />class MyAspect():<br />  def visit(self, node: IConstruct) -> None:<br />    print("Visited", node.node.path)</pre> | <pre>import software.amazon.awscdk.core.IAspect;<br />import software.amazon.awscdk.core.IConstruct;<br /><br />public class MyAspect implements IAspect {<br />  @Override<br />  public void visit(IConstruct node) {<br />    System.out.println("Visited " + node.getNode().getPath());<br />  }<br />}</pre> |
