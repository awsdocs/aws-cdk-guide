# Working with the AWS CDK in Python<a name="work-with-cdk-python"></a>

Python is a fully\-supported client language for the AWS CDK and is considered stable\. Working with the AWS CDK in Python uses familiar tools, including the standard Python implementation \(dubbed CPython\), `virtualenv`, and `pip`, the Python package installer\. The modules comprising the AWS Construct Library are distributed via [pypi\.org](https://pypi.org/search/?q=aws-cdk)\. The Python version of the AWS CDK even uses Python\-style identifiers \(for example, `snake_case` method names\)\.

You can use any editor or IDE; many AWS CDK developers use [Visual Studio Code](https://code.visualstudio.com/) \(or its open\-source equivalent [VSCodium](https://github.com/VSCodium/vscodium)\), which has good support for Python via an [official extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python), though the simple IDLE editor included with Python will suffice to get started\. The Python modules for the AWS CDK do have type hints, so you may prefer a linting tool or an IDE that supports type validation\.

## Prerequisites<a name="python-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [AWS CDK Prerequisites](work-with.md#work-with-prerequisites)\.

Python AWS CDK applications require Python 3\.6 or later\. If you don't already have it installed, [download a compatible version](https://www.python.org/downloads/) for your platform at [python\.org](https://www.python.org/)\. If you run Linux, your system may have come with a compatible version, or you may install it using your distro's package manager \(`yum`, `apt`, etc\.\)\. Mac users may be interested in [Homebrew](https://brew.sh/), a Linux\-style package manager for Mac OS X\.

The Python package installer, `pip`, and virtual environment manager, `virtualenv`, are also required\. Windows installations of compatible Python versions include these tools\. On Linux, `pip` and `virtualenv` may be provided as separate packages in your package manager\. Alternatively, you may install them with the following commands:

```
python -m ensurepip --upgrade
python -m pip install --upgrade pip
python -m pip install --upgrade virtualenv
```

If you encounter a permission error, run the above commands using `sudo` \(to install the modules system\-wide\) or add the `--user` flag to each command so that the modules are installed in your user directory\.

**Note**  
It is common for Linux distros to use the executable name `python3` for Python 3\.x, and have `python` refer to a Python 2\.x installation, and similarly for `pip`/`pip3`\. You can adjust the command used to run your application by editing `cdk.json` in the project's main directory\.

Make sure the `pip` executable \(on Windows, `pip.exe`\) is in a directory included on the system `PATH`\. You should be able to type `pip --version` and see its version, not an error message\.

## Creating a project<a name="python-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\.

```
mkdir my-project
cd my-project
cdk init app --language python
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. 

After initializing the project, activate the project's virtual environment\. This allows the project's dependencies to be installed locally in the project folder, instead of globally\.

```
source .env/bin/activate
```

Then install the app's standard dependencies:

```
pip install -r requirements.txt
```

**Important**  
Activate the project's virtual environment whenever you start working on it\. If you don't, you won't have access to the modules installed there, and modules you install will go in Python's global module directory \(or will result in a permission error\)\.

## Managing AWS construct library modules<a name="python-managemodules"></a>

Use the Python package installer, `pip`, to install and update AWS Construct Library modules for use by your apps, as well as other packages you need\. `pip` also installs the dependencies for those modules automatically\.

AWS Construct Library modules are named like `aws-cdk.SERVICE-NAME`\. For example, the command below installs the modules for Amazon S3 and AWS Lambda\.

```
pip install aws-cdk.aws-s3 aws-cdk.aws-lambda
```

After installing a module, update your project's `requirements.txt` file, which maintains your project's dependencies\. You may do this manually, by opening `requirements.txt` in your editor, or by issuing:

```
pip freeze > requirements.txt
```

`pip freeze` captures the current versions of all modules installed in your virtual environment\. You can edit `requirements.txt` to allow upgrades; simply replace the `==` preceding a version number with `~=` to allow upgrades to a higher compatible version, or remove the version requirement entirely to specify the latest available version of the module\.

You may want to remove modules that are only installed because they are dependencies of other modules; these will be installed automatically anyway, and by not listing them explicitly you will ensure that you get a version compatible with the version of the module you actually want, and no unnecessary dependencies\.

With `requirements.txt` edited appropriately to allow upgrades, issue this command to upgrade the installed modules:

```
pip install --upgrade -r requirements.txt
```

**Note**  
All AWS Construct Library modules used in your project must be the same version\.

## AWS CDK idioms in Python<a name="python-cdk-idioms"></a>

### Props<a name="python-props"></a>

Natively, all AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), a *name*, and *props*, a bundle of key/value pairs that the construct uses to configure the resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

In Python, props are expressed as keyword arguments\. If an argument contains nested data structures, these are expressed using a class which takes its own keyword arguments at instantiation\. The same pattern is applied to other method calls that take a single structured argument\.

For example, in a Amazon S3 bucket's `add_lifecycle_rule` method, the `transitions` property is a list of `Transition` instances\.

```
bucket.add_lifecycle_rule(
  transitions=[
    Transition(
      storage_class=StorageClass.GLACIER,
      transition_after=Duration.days(10)
    )
  ]
)
```

When extending a class or overriding a method, you may want to accept additional arguments for your own purposes that are not understood by the parent class\. In this case you should accept the arguments you don't care about using the `**kwargs` idiom, and use keyword\-only arguments to accept the arguments you're interested in\. When calling the parent's constructor or the overridden method, pass only the arguments it is expecting \(often just `**kwargs`\)\. Passing arguments that the parent class or method doesn't expect results in an error\.

Future releases of the AWS CDK may coincidentally add a new property with a name you used for your own property\. This won't cause any technical issues for users of your construct or method \(since your property isn't passed "up the chain," the parent class or overridden method will simply use a default value\) but it may cause confusion\. You can avoid this potential problem by naming your properties so they clearly belong to your construct \(e\.g\. `bob_encryption` rather than just `encryption`, assuming you're Bob\)\. If there are many new properties, bundle them into an appropriately\-named class \(`BobBucketPoperties`?\) and pass it as a single keyword argument\.

### Missing values<a name="python-missing-values"></a>

When working with `**kwargs`, use the dictionary's `get()` method to provide a default value if a property is not provided\. Avoid using `kwargs[...]`, as this raises `KeyError` for missing values\.

```
encrypted = kwargs.get("encrypted")    		# None if no property "encrypted" exists
encrypted = kwargs.get("encrypted", False)	# specify default of False if property is missing
```

Some AWS CDK methods \(such as `tryGetContext()` to get a runtime context value\) return `None` to indicate a missing value, which you will need to check for and handle\.

### Using interfaces<a name="python-interfaces"></a>

Python doesn't have an interface feature as some other languages do\. \(If you're not familiar with the concept, Wikipedia has [a good introduction](https://en.wikipedia.org/wiki/Interface_(computing)#In_object-oriented_languages)\.\) TypeScript, the language in which the AWS CDK is implemented does, however, and constructs and other AWS CDK objects often require an instance that adheres to a particular interface, rather than inheriting from a particular class\. So the AWS CDK provides its own interface feature as part of the [JSII](https://github.com/aws/jsii) layer\.

To indicate that a class implements a particular interface, you can use the `@jsii.implements` decorator:

```
from aws_cdk.core import IAspect, IConstruct
import jsii

@jsii.implements(IAspect)
class MyAspect():
    def visit(self, node: IConstruct) -> None:
        print("Visited", node.node.path)
```

### Type pitfalls<a name="python-type-pitfalls"></a>

Python natively uses dynamic typing, where variables may refer to a value of any type\. Parameters and return values may be annotated with types, but these are "hints" and are not enforced\. This means that in Python, it is easy to pass the incorrect type of value to a AWS CDK construct\. Instead of getting a type error during build, as you would from a statically\-typed language, you may instead get a runtime error when the JSII layer \(which translates between Python and the AWS CDK's TypeScript core\) is unable to deal with the unexpected type\.

In our experience, the type errors Python programmers make tend to fall into these categories\. Be especially alert to these pitfalls\.
+ Passing a single value where a construct expects a container \(Python list or dictionary\) or vice versa\.
+ Passing a value of a type associated with a Level 1 \(`CfnXxxxxx`\) construct to a higher\-level construct, or vice versa\.

The AWS CDK Python modules do include type annotations\. If you are not using an IDE that supports these, such as [PyCharm](https://www.jetbrains.com/pycharm/), you might want to call the [MyPy](http://mypy-lang.org/) type validator as a step in your build process\. There are also runtime type checkers that can improve error messages for type\-related errors\.

## Synthesizing and deploying<a name="python-running"></a>

The [stacks](stacks.md) defined in your AWS CDK app can be deployed individually or together using the commands below\. Generally, you should be in your project's main directory when you issue them\.
+ `cdk synth`: Synthesizes a AWS CloudFormation template from one or more of the stacks in your AWS CDK app\.
+ `cdk deploy`: Deploys the resources defined by one or more of the stacks in your AWS CDK app to AWS\.

You can specify the names of multiple stacks to be synthesized or deployed in a single command\. If your app defines only one stack, you do not need to specify it\. 

```
cdk synth                 # app defines single stack
cdk deploy Happy Grumpy   @ app defines two or more stacks; two are deployed
```

You may also use the wildcards \* \(any number of characters\) and ? \(any single character\) to identify stacks by pattern\. When using wildcards, enclose the pattern in quotes; otherwise, the shell may try to expand \("glob"\) it to the names of files in the current directory before they are passed to the AWS CDK Toolkit\.

```
cdk synth "Stack?"    # Stack1, StackA, etc.
cdk deploy "*Stack"   @ PipeStack, LambdaStack, etc.
```

**Tip**  
You don't need to explicitly synthesize stacks before deploying them; `cdk deploy` performs this step for you to make sure your latest code gets deployed\.

For full documentation of the `cdk` command, see [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.