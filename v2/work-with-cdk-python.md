# Working with the AWS CDK in Python<a name="work-with-cdk-python"></a>

Python is a fully\-supported client language for the AWS Cloud Development Kit \(AWS CDK\) and is considered stable\. Working with the AWS CDK in Python uses familiar tools, including the standard Python implementation \(CPython\), virtual environments with `virtualenv`, and the Python package installer `pip`\. The modules comprising the AWS Construct Library are distributed via [pypi\.org](https://pypi.org/search/?q=aws-cdk)\. The Python version of the AWS CDK even uses Python\-style identifiers \(for example, `snake_case` method names\)\.

You can use any editor or IDE\. Many AWS CDK developers use [Visual Studio Code](https://code.visualstudio.com/) \(or its open\-source equivalent [VSCodium](https://vscodium.com/)\), which has good support for Python via an [official extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python)\. The IDLE editor included with Python will suffice to get started\. The Python modules for the AWS CDK do have type hints, which are useful for a linting tool or an IDE that supports type validation\.

**Topics**
+ [Get started with Python](#python-prerequisites)
+ [Creating a project](#python-newproject)
+ [Managing AWS Construct Library modules](#python-managemodules)
+ [Managing dependencies in Python](#work-with-cdk-python-dependencies)
+ [AWS CDK idioms in Python](#python-cdk-idioms)
+ [Synthesizing and deploying](#python-running)

## Get started with Python<a name="python-prerequisites"></a>

To work with the AWS CDK, you must have an AWS account and credentials and have installed Node\.js and the AWS CDK Toolkit\. See [Getting started with the AWS CDK](getting_started.md)\.

Python AWS CDK applications require Python 3\.6 or later\. If you don't already have it installed, [download a compatible version](https://www.python.org/downloads/) for your operating system at [python\.org](https://www.python.org/)\. If you run Linux, your system may have come with a compatible version, or you may install it using your distro's package manager \(`yum`, `apt`, etc\.\)\. Mac users may be interested in [Homebrew](https://brew.sh/), a Linux\-style package manager for macOS\.

**Note**  
Third\-party language deprecation: language version is only supported until its EOL \(End Of Life\) shared by the vendor or community and is subject to change with prior notice\.

The Python package installer, `pip`, and virtual environment manager, `virtualenv`, are also required\. Windows installations of compatible Python versions include these tools\. On Linux, `pip` and `virtualenv` may be provided as separate packages in your package manager\. Alternatively, you may install them with the following commands:

```
python -m ensurepip --upgrade
python -m pip install --upgrade pip
python -m pip install --upgrade virtualenv
```

If you encounter a permission error, run the above commands with the `--user` flag so that the modules are installed in your user directory, or use `sudo` to obtain the permissions to install the modules system\-wide\.

**Note**  
It is common for Linux distros to use the executable name `python3` for Python 3\.x, and have `python` refer to a Python 2\.x installation\. Some distros have an optional package you can install that makes the `python` command refer to Python 3\. Failing that, you can adjust the command used to run your application by editing `cdk.json` in the project's main directory\.

**Note**  
On Windows, you may want to invoke Python \(and pip\) using the py executable, the [>Python launcher for Windows](https://docs.python.org/3/using/windows.html#launcher)\. Among other things, the launcher allows you to easily specify which installed version of Python you want to use\.  
If typing python at the command line results in a message about installing Python from the Windows Store, even after installing a Windows version of Python, open Windows' Manage App Execution Aliases settings panel and turn off the two App Installer entries for Python\.

## Creating a project<a name="python-newproject"></a>

You create a new AWS CDK project by invoking `cdk init` in an empty directory\. Use the `--language` option and specify `python`:

```
mkdir my-project
cd my-project
cdk init app --language python
```

`cdk init` uses the name of the project folder to name various elements of the project, including classes, subfolders, and files\. Hyphens in the folder name are converted to underscores\. However, the name should otherwise follow the form of a Python identifier; for example, it should not start with a number or contain spaces\. 

To work with the new project, activate its virtual environment\. This allows the project's dependencies to be installed locally in the project folder, instead of globally\.

```
source .venv/bin/activate
```

**Note**  
You may recognize this as the Mac/Linux command to activate a virtual environment\. The Python templates include a batch file, `source.bat`, that allows the same command to be used on Windows\. The traditional Windows command, `.\venv\Scripts\activate`, works, too\.  
If you initialized your AWS CDK project using CDK Toolkit v1\.70\.0 or earlier, your virtual environment is in the `.env` directory instead of `.venv`\.

**Important**  
Activate the project's virtual environment whenever you start working on it\. Otherwise, you won't have access to the modules installed there, and modules you install will go in the Python global module directory \(or will result in a permission error\)\.

After activating your virtual environment for the first time, install the app's standard dependencies:

```
python -m pip install -r requirements.txt
```

## Managing AWS Construct Library modules<a name="python-managemodules"></a>

Use the Python package installer, pip, to install and update AWS Construct Library modules for use by your apps, as well as other packages you need\. pip also installs the dependencies for those modules automatically\. If your system does not recognize pip as a standalone command, invoke pip as a Python module, like this:

```
python -m pip PIP-COMMAND
```

Most AWS CDK constructs are in `aws-cdk-lib`\. Experimental modules are in separate modules named like `aws-cdk.SERVICE-NAME.alpha`\. The service name includes an *aws* prefix\. If you're unsure of a module's name, [search for it at PyPI](https://pypi.org/search/?q=aws-cdk)\. For example, the command below installs the AWS CodeStar library\.

```
python -m pip install aws-cdk.aws-codestar-alpha
```

Some services' constructs are in more than one namespace\. For example, besides `aws-cdk.aws-route53`, there are three additional Amazon Route 53 namespaces, named `aws-route53-targets`, `aws-route53-patterns`, and `aws-route53resolver`\.

**Note**  
The [Python edition of the CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/python/index.html) also shows the package names\.

The names used for importing AWS Construct Library modules into your Python code look like the following\.

```
import aws_cdk.aws_s3 as s3
import aws_cdk.aws_lambda as lambda_
```

We recommend the following practices when importing AWS CDK classes and AWS Construct Library modules in your applications\. Following these guidelines will help make your code consistent with other AWS CDK applications as well as easier to understand\.
+ Generally, import individual classes from top\-level `aws_cdk`\.

  ```
  from aws_cdk import App, Construct
  ```
+ If you need many classes from the `aws_cdk`, you may use a namespace alias of `cdk` instead of importing individual classes\. Avoid doing both\.

  ```
  import aws_cdk as cdk
  ```
+ Generally, import AWS Construct Libraries using short namespace aliases\.

  ```
  import aws_cdk.aws_s3 as s3
  ```

After installing a module, update your project's `requirements.txt` file, which lists your project's dependencies\. It is best to do this manually rather than using `pip freeze`\. `pip freeze` captures the current versions of all modules installed in your Python virtual environment, which can be useful when bundling up a project to be run elsewhere\.

Usually, though, your `requirements.txt` should list only top\-level dependencies \(modules that your app depends on directly\) and not the dependencies of those libraries\. This strategy makes updating your dependencies simpler\.

You can edit `requirements.txt` to allow upgrades; simply replace the `==` preceding a version number with `~=` to allow upgrades to a higher compatible version, or remove the version requirement entirely to specify the latest available version of the module\.

With `requirements.txt` edited appropriately to allow upgrades, issue this command to upgrade your project's installed modules at any time:

```
pip install --upgrade -r requirements.txt
```

## Managing dependencies in Python<a name="work-with-cdk-python-dependencies"></a>

In Python, you specify dependencies by putting them in `requirements.txt` for applications or `setup.py` for construct libraries\. Dependencies are then managed with the PIP tool\. PIP is invoked in one of the following ways:

```
pip command options
python -m pip command options
```

The python \-m pip invocation works on most systems; pip requires that PIP's executable be on the system path\. If pip doesn't work, try replacing it with python \-m pip\.

The cdk init \-\-language python command creates a virtual environment for your new project\. This lets each project have its own versions of dependencies, and also a basic `requirements.txt` file\. You must activate this virtual environment by running source \.venv/bin/activate each time you begin working with the project\. On Windows, run \.\\venv\\Scripts\\activate instead

### CDK applications<a name="work-with-cdk-python-dependencies-apps"></a>

The following is an example `requirements.txt` file\. Because PIP does not have a dependency\-locking feature, we recommend that you use the == operator to specify exact versions for all dependencies, as shown here\.

```
aws-cdk-lib==2.14.0
aws-cdk.aws-appsync-alpha==2.10.0a0
```

Installing a module with pip install does not automatically add it to `requirements.txt`\. You must do that yourself\. If you want to upgrade to a later version of a dependency, edit its version number in `requirements.txt`\.

To install or update your project's dependencies after creating or editing `requirements.txt`, run the following:

```
python -m pip install -r requirements.txt
```

**Tip**  
The pip freeze command outputs the versions of all installed dependencies in a format that can be written to a text file\. This can be used as a requirements file with `pip install -r`\. This file is convenient for pinning all dependencies \(including transitive ones\) to the exact versions that you tested with\. To avoid problems when upgrading packages later, use a separate file for this, such as `freeze.txt` \(not `requirements.txt`\)\. Then, regenerate it when you upgrade your project's dependencies\.

### Third\-party construct libraries<a name="work-with-cdk-python-dependencies-libraries"></a>

In libraries, dependencies are specified in `setup.py`, so that transitive dependencies are automatically downloaded when the package is consumed by an application\. Otherwise, every application that wants to use your package needs to copy your dependencies into their `requirements.txt`\. An example `setup.py` is shown here\.

```
from setuptools import setup

setup(
  name='my-package',
  version='0.0.1',
  install_requires=[
    'aws-cdk-lib==2.14.0',
  ], 
  ...
)
```

To work on the package for development, create or activate a virtual environment, then run the following command\.

```
python -m pip install -e .
```

Although PIP automatically installs transitive dependencies, there can only be one installed copy of any one package\. The version that is specified highest in the dependency tree is selected; applications always have the last word in what version of packages get installed\. 

## AWS CDK idioms in Python<a name="python-cdk-idioms"></a>

### Language conflicts<a name="python-keywords"></a>

In Python, `lambda` is a language keyword, so you cannot use it as a name for the AWS Lambda construct library module or Lambda functions\. The Python convention for such conflicts is to use a trailing underscore, as in `lambda_`, in the variable name\.

By convention, the second argument to AWS CDK constructs is named `id`\. When writing your own stacks and constructs, calling a parameter `id` "shadows" the Python built\-in function `id()`, which returns an object's unique identifier\. This function isn't used very often, but if you should happen to need it in your construct, rename the argument, for example `construct_id`\.

### Arguments and properties<a name="python-props"></a>

All AWS Construct Library classes are instantiated using three arguments: the *scope* in which the construct is being defined \(its parent in the construct tree\), an *id*, and *props*, a bundle of key/value pairs that the construct uses to configure the resources it creates\. Other classes and methods also use the "bundle of attributes" pattern for arguments\.

*scope* and *id* should always be passed as positional arguments, not keyword arguments, because their names change if the construct accepts a property named *scope* or *id*\.

In Python, props are expressed as keyword arguments\. If an argument contains nested data structures, these are expressed using a class which takes its own keyword arguments at instantiation\. The same pattern is applied to other method calls that take a structured argument\.

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

```
class MyConstruct(Construct):
    def __init__(self, id, *, MyProperty=42, **kwargs):
        super().__init__(self, id, **kwargs)
        # ...
```

A future release of the AWS CDK could coincidentally add a new property with a name you used for your own property\. This won't cause any technical issues for users of your construct or method \(since your property isn't passed "up the chain," the parent class or overridden method will simply use a default value\) but it may cause confusion\. You can avoid this potential problem by naming your properties so they clearly belong to your construct\. If there are many new properties, bundle them into an appropriately\-named class and pass it as a single keyword argument\.

### Missing values<a name="python-missing-values"></a>

The AWS CDK uses `None` to represent missing or undefined values\. When working with `**kwargs`, use the dictionary's `get()` method to provide a default value if a property is not provided\. Avoid using `kwargs[...]`, as this raises `KeyError` for missing values\.

```
encrypted = kwargs.get("encrypted")         # None if no property "encrypted" exists
encrypted = kwargs.get("encrypted", False)  # specify default of False if property is missing
```

Some AWS CDK methods \(such as `tryGetContext()` to get a runtime context value\) may return `None`, which you will need to check explicitly\.

### Using interfaces<a name="python-interfaces"></a>

Python doesn't have an interface feature as some other languages do, though it does have [abstract base classes](https://docs.python.org/3/library/abc.html), which are similar\. \(If you're not familiar with interfaces, Wikipedia has [a good introduction](https://en.wikipedia.org/wiki/Interface_(computing)#In_object-oriented_languages)\.\) TypeScript, the language in which the AWS CDK is implemented, does provide interfaces, and constructs and other AWS CDK objects often require an object that adheres to a particular interface, rather than inheriting from a particular class\. So the AWS CDK provides its own interface feature as part of the [JSII](https://github.com/aws/jsii) layer\.

To indicate that a class implements a particular interface, you can use the `@jsii.implements` decorator:

```
from aws_cdk import IAspect, IConstruct
import jsii

@jsii.implements(IAspect)
class MyAspect():
    def visit(self, node: IConstruct) -> None:
        print("Visited", node.node.path)
```

### Type pitfalls<a name="python-type-pitfalls"></a>

Python uses dynamic typing, where all variables may refer to a value of any type\. Parameters and return values may be annotated with types, but these are "hints" and are not enforced\. This means that in Python, it is easy to pass the incorrect type of value to a AWS CDK construct\. Instead of getting a type error during build, as you would from a statically\-typed language, you may instead get a runtime error when the JSII layer \(which translates between Python and the AWS CDK's TypeScript core\) is unable to deal with the unexpected type\.

In our experience, the type errors Python programmers make tend to fall into these categories\.
+ Passing a single value where a construct expects a container \(Python list or dictionary\) or vice versa\.
+ Passing a value of a type associated with a layer 1 \(`CfnXxxxxx`\) construct to a L2 or L3 construct, or vice versa\.

The AWS CDK Python modules do include type annotations, so you can use tools that support them to help with types\. If you are not using an IDE that supports these, such as [PyCharm](https://www.jetbrains.com/pycharm/), you might want to call the [MyPy](http://mypy-lang.org/) type validator as a step in your build process\. There are also runtime type checkers that can improve error messages for type\-related errors\.

## Synthesizing and deploying<a name="python-running"></a>

The [stacks](stacks.md) defined in your AWS CDK app can be synthesized and deployed individually or together using the commands below\. Generally, you should be in your project's main directory when you issue them\.
+ `cdk synth`: Synthesizes a AWS CloudFormation template from one or more of the stacks in your AWS CDK app\.
+ `cdk deploy`: Deploys the resources defined by one or more of the stacks in your AWS CDK app to AWS\.

You can specify the names of multiple stacks to be synthesized or deployed in a single command\. If your app defines only one stack, you do not need to specify it\. 

```
cdk synth                 # app defines single stack
cdk deploy Happy Grumpy   # app defines two or more stacks; two are deployed
```

You may also use the wildcards \* \(any number of characters\) and ? \(any single character\) to identify stacks by pattern\. When using wildcards, enclose the pattern in quotes\. Otherwise, the shell may try to expand it to the names of files in the current directory before they are passed to the AWS CDK Toolkit\.

```
cdk synth "Stack?"    # Stack1, StackA, etc.
cdk deploy "*Stack"   # PipeStack, LambdaStack, etc.
```

**Tip**  
You don't need to explicitly synthesize stacks before deploying them; `cdk deploy` performs this step for you to make sure your latest code gets deployed\.

For full documentation of the `cdk` command, see [AWS CDK Toolkit \(`cdk` command\)](cli.md)\.