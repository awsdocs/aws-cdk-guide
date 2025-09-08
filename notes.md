Plugins
=======
The |cdk| toolkit provides extension points that enable users to augment the features provided by
the toolkit. There is currently only one extension point, allowing users to define custom AWS
credential providers, but other extension points may be added in the future as the needs arise.
Loading Plugins
---------------
Plugins can be loaded by providing the Node module name (or path) to the CDK toolkit:
1. Using the ``--plugin`` command line option (which can be specified multiple times):
   .. code-block:: console
      $ cdk list --plugin=module
      $ cdk deploy --plugin=module_1 --plugin=module_2
2. Adding entries to the ``~/.cdk.json`` or ``cdk.json`` file:
   .. code-block:: js
      {
        // ...
        "plugin": [
            "module_1",
            "module_2"
        ],
        // ...
      }
Authoring Plugins
-----------------
Plugins must be authored in TypeScript or Javascript, and are defined by implementing a Node module
that implements the following protocol, and using :js:class:`~aws-cdk.PluginHost` methods:
.. tabs::
    .. code-tab:: typescript
        import cdk = require('aws-cdk');
        export = {
            version: '1', // Version of the plugin infrastructure (currently always '1')
            init(host: cdk.PluginHost): void {
                // Your plugin initialization hook.
                // Use methods of ``host`` to register custom code with the CDK toolkit
            }
        };
    .. code-tab:: javascript
        export = {
            version: '1', // Version of the plugin infrastructure (currently always '1')
            init(host) {
                // Your plugin initialization hook.
                // Use methods of ``host`` to register custom code with the CDK toolkit
            }
        };
Credential Provider Plugins
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Custom credential providers are classes implementing the
:js:class:`~aws-cdk.CredentialProviderSource` interface, and registered to the toolkit using
the :js:func:`~aws-cdk.PluginHost.registerCredentialProviderSource` method.
.. tabs::
   .. code-tab:: typescript
      import cdk = require('aws-cdk');
      import aws = require('aws-sdk');
      class CustomCredentialProviderSource implements cdk.CredentialProviderSource {
        public async isAvailable(): Promise<boolean> {
          // Return ``false`` if the plugin could determine it cannot be used (for example,
          // if it depends on files that are not present on this host).
          return true;
        }
        public async canProvideCredentials(accountId: string): Promise<boolean> {
          // Return ``false`` if the plugin is unable to provide credentials for the
          // requested account (for example if it's not managed by the credentials
          // system that this plugin adds support for).
          return true;
        }
        public async getProvider(accountId: string, mode: cdk.Mode): Promise<aws.Credentials> {
          let credentials: aws.Credentials;
          // Somehow obtain credentials in ``credentials``, and return those.
          return credentials;
        }
      }
      export = {
        version = '1',
        init(host: cdk.PluginHost): void {
          // Register the credential provider to the PluginHost.
          host.registerCredentialProviderSource(new CustomCredentialProviderSource());
        }
      };
   .. code-tab:: javascript
      class CustomCredentialProviderSource {
        async isAvailable() {
          // Return ``false`` if the plugin could determine it cannot be used (for example,
          // if it depends on files that are not present on this host).
          return true;
        }
        async canProvideCredentials(accountId) {
          // Return ``false`` if the plugin is unable to provide credentials for the
          // requested account (for example if it's not managed by the credentials
          // system that this plugin adds support for).
          return true;
        }
        async getProvider(accountId, mode) {
          let credentials;
          // Somehow obtain credentials in ``credentials``, and return those.
          return credentials;
        }
      }
      export = {
        version = '1',
        init(host) {
          // Register the credential provider to the PluginHost.
          host.registerCredentialProviderSource(new CustomCredentialProviderSource());
        }
      };
Note that the credentials obtained from the providers for a given account and mode will be cached,
and as a consequence it is strongly advised that the credentials objects returned are self-refreshing,
as descibed in the `AWS SDK for Javascript documentation <https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Credentials.html>`_.
Reference
---------
.. js:module:: aws-cdk
CredentialProviderSource *(interface)*
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. js:class:: CredentialProviderSource
   .. js:attribute:: name
      A friendly name for the provider, which will be used in error messages, for example.
      :type: ``string``
   .. js:method:: isAvailable()
      Whether the credential provider is even online. Guaranteed to be called before any of the other functions are called.
      :returns: ``Promise<boolean>``
   .. js:method:: canProvideCredentials(accountId)
      Whether the credential provider can provide credentials for the given account.
      :param string accountId: the account ID for which credentials are needed.
      :returns: ``Promise<boolean>``
   .. js:method:: getProvider(accountId, mode)
      Construct a credential provider for the given account and the given access mode.
      Guaranteed to be called only if canProvideCredentails() returned true at some point.
      :param string accountId: the account ID for which credentials are needed.
      :param aws-cdk.Mode mode: the kind of operations the credentials are intended to perform.
      :returns: ``Promise<aws.Credentials>``
Mode *(enum)*
^^^^^^^^^^^^^
.. js:class:: Mode
   .. js:data:: ForReading
      Credentials are inteded to be used for read-only scenarios.
   .. js:data:: ForWriting
      Credentials are intended to be used for read-write scenarios.
Plugin *(interface)*
^^^^^^^^^^^^^^^^^^^^
.. js:class:: Plugin()
   .. js:attribute:: version
      The version of the plug-in interface used by the plug-in. This will be used by
      the plug-in host to handle version changes. Currently, the only valid value for
      this attribute is ``'1'``.
      :type: ``string``
   .. js:method:: init(host)
      When defined, this function is invoked right after the plug-in has been loaded,
      so that the plug-in is able to initialize itself. It may call methods of the
      :js:class:`~aws-cdk.PluginHost` instance it receives to register new
      :js:class:`~aws-cdk.CredentialProviderSource` instances.
      :param aws-cdk.PluginHost host: The |cdk| plugin host
      :returns: ``void``
PluginHost
^^^^^^^^^^
.. js:class:: PluginHost()
   .. js:data:: instance
      :type: :js:class:`~aws-cdk.PluginHost`
   .. js:method:: load(moduleSpec)
      Loads a plug-in into this PluginHost.
      :param string moduleSpec: the specification (path or name) of the plug-in module to be loaded.
      :throws Error: if the provided ``moduleSpec`` cannot be loaded or is not a valid :js:class:`~aws-cdk.Plugin`.
      :returns: ``void``
   .. js:method:: registerCredentialProviderSource(source)
      Allows plug-ins to register new :js:class:`~aws-cdk.CredentialProviderSources`.
      :param aws-cdk.CredentialProviderSources source: a new CredentialProviderSource to register.
      :returns: ``void``