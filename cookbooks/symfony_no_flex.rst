SymfonyCloud without Flex
=========================

The SymfonyCloud experience is optimized for Symfony applications using Flex,
but not every project has been migrated to Flex yet. Even if you are using
Symfony 2.x or 3.x, you can start using SymfonyCloud and actually leverage some
of its features to ease the migration to Symfony 4 and Flex.

.. tip::

   Although this guide helps you setting up your own project for SymfonyCloud,
   we suggest you first follow the :doc:`Getting Started </getting-started>`
   guide with a brand new Symfony 4 project and a free trial to get a better
   understanding of SymfonyCloud features and philosophy.

   .. caution::

      .. include:: ../_includes/trial-warning.rst

Initial Setup
-------------

.. note::

   You have to :ref:`install the CLI Tool <cli-install>` first.

From within your project's directory, generate a default SymfonyCloud
configuration:

.. code-block:: terminal

    $ symfony project:init

Thanks to our versatile templates, most of the hard work of adapting the
configuration to your Symfony project structure is already done
for you in the ``.symfony.cloud.yaml`` and
``app/config/parameters_symfonycloud.php`` files.

As for any SymfonyCloud project, you will have to edit the
``.symfony.cloud.yaml`` and ``.symfony/services.yaml`` files according to your
needs. See the :doc:`Services documentation </services/intro>` for a complete
description of currently supported services types, endpoints and associated
environment variables.

Symfony Version
---------------

The quality of your experience with SymfonyCloud vastly depends on your Symfony
version: before Symfony 3.2 you do not benefit from the
`Runtime Environment Variables`_ as the `DotEnv component`_ has been
introduced in Symfony 3.3. SymfonyCloud leverages environment variables to
expose service credentials and to allow fast creation of new environments with
the same codebase. Those two features are key for a great SymfonyCloud
experience.

We highly recommend to upgrade to 3.2+ before getting started. If you cannot do
so, your SymfonyCloud configuration will remain the same but your deployments
will take longer as your Symfony cache will need to be built each time.

.. tip::

   As the configuration change between the two cases is minimal, you can
   proceed with a first deployment "as-is" without changing your Symfony
   codebase and then iteratively tweak it
   (:ref:`using SymfonyCloud environments <working-with-environments>` as a
   staging sandbox) until your are ready to move to production.

Before Symfony 3.2
~~~~~~~~~~~~~~~~~~

In such configuration, Symfony cache can only be properly warmed up during the
environment deployment. As a result the :ref`symfony-build` helper will not
persist the cache, and :ref:`symfony-deploy` will transparently warm up the
cache during deployment.

Another consideration is regarding environment variables: most Symfony 2 or 3
applications are using parameters to handle configuration values such as
database credentials. As Symfony configuration cannot read them, you will have
to "map" environment variables to parameters values.

The ``symfony init`` command generated some boilerplate code to do so for you::

    // app/config/parameters_symfonycloud.php

    // Not running in a SymfonyCloud runtime
    if (!isset($_SERVER['SYMFONY_ENVIRONMENT'])) {
        return;
    }

    $container->setParameter('secret', $_SERVER['APP_SECRET'] ?? $_SERVER['SECRET']);
    $container->setParameter('kernel.environment', $_SERVER['APP_ENV'] ?? $_SERVER['SYMFONY_ENV']);
    $container->setParameter('kernel.debug', (bool) ($_SERVER['APP_DEBUG'] ?? $_SERVER['SYMFONY_DEBUG']));
    $container->setParameter('router.request_context.host', $_SERVER['SYMFONY_DEFAULT_ROUTE_HOST']);
    $container->setParameter('router.request_context.scheme', $_SERVER['SYMFONY_DEFAULT_ROUTE_SCHEME']);

    $envMapping = [
        'mailer_transport' => 'MAILER_TRANSPORT',
        'mailer_user' => 'MAILER_USER',
        'mailer_password' => 'MAILER_PASSWORD',
        'mailer_host' => 'MAILER_HOST',
        'mailer_port' => 'MAILER_PORT',
    ];

    foreach ($envMapping as $parameter => $env) {
        if (!isset($_SERVER[$env])) {
            continue;
        }

        $container->setParameter($parameter, $_SERVER[$env]);
    }

You will have to edit the ``$envMapping`` table (the key is the parameter name,
the value is the name of the environment variable to read) and make Symfony
consider this file as part of your configuration:

.. code-block:: yaml

   # app/config/config.yml
   imports:
       - { resource: parameters.yml }
   +   - { resource: parameters_symfonycloud.php }
       - { resource: security.yml }
       - { resource: services.yml }

With this setup, when warming its cache in a SymfonyCloud environment, Symfony
will read the parameters value from the environment variables.

.. note::

   During the build phase, Composer scripts are run, which by default will build
   Symfony cache, even if its not reused later on. This requires that your
   Symfony application has sensible default parameters, but also that the
   warm-up must be possible even when services (like a database) are not
   available.

Symfony 3.2 and up
~~~~~~~~~~~~~~~~~~

.. note::

   Even though only `Symfony 3.2`_ is strictly required in this
   case, we **highly** recommend you to upgrade to 3.4 right away:
   - `Symfony 3.3`_ brings the `DotEnv component`_ that will ease
   your local setups
   - `Symfony 3.4`_ is the latest version for the 3.x branch, an
   `LTS version <https://symfony.com/doc/current/contributing/community/releases.html#maintenance>`_
   and remains backward compatible with previous 3.x versions

When using Symfony 3.2, the Symfony cache can be warmed up and setup to read
some of its configuration values at runtime from environment variables. To
leverage this on SymfonyCloud, you will have to migrate some of your uses of
parameters in the configuration to environment variables uses.

For example, the following Doctrine configuration:

.. code-block:: yaml

   # app/config/config.yml
   doctrine:
        dbal:
            # ...
            host: "%database_host%"
            port: "%database_port%"
            name: "%database_name%"
            user: "%database_user%"
            password: "%database_password%"

should be migrated as follows:

.. code-block:: yaml

   # app/config/config.yml
   doctrine:
        dbal:
            # ...
            host: "%env(DATABASE_HOST)%"
            port: "%env(DATABASE_PORT)%"
            name: "%env(DATABASE_NAME)%"
            user: "%env(DATABASE_USER)%"
            password: "%env(DATABASE_PASSWORD)%"

Don't forget to define sensible defaults to allow Symfony Kernel boot during
build stage:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            env(DATABASE_HOST): 127.0.0.1
            env(DATABASE_PORT): 3306
            env(DATABASE_NAME): db_name
            env(DATABASE_USER): db_user
            env(DATABASE_PASSWORD): db_password

    .. code-block:: env

        # .env
        # Only possible if the DotEnv component is available and configured
        DATABASE_HOST: 127.0.0.1
        DATABASE_PORT: 3306
        DATABASE_NAME: db_name
        DATABASE_USER: db_user
        DATABASE_PASSWORD: db_password

.. caution::

   Some third-party bundles might resolve configuration values during the cache
   build, this results in inlined values in the container. This prevents the use
   of runtime environment variables. In such case, the bundle must be upgraded
   or patched, or you can decide to use a static configuration if it is enough
   for your needs.

The last step is to make SymfonyCloud save and reuse the built cache during the
build stage by uncommenting the ``mv`` command in the build hook:

.. code-block:: yaml

    # .symfony.cloud.yaml
    hooks:
        build: |
            set -x -e

            curl -s https://get.symfony.com/cloud/configurator | (>&2 bash)
            (>&2
              symfony-build
              # Keep the cache in a persistent directory to speed up deployments
              mkdir -p tmp/cache && mv app/cache/prod tmp/cache/
            )

.. tip::

   With this migration done, it is possible that your application does not
   require environment variable mapping at all. If this is the case you can
   safely remove ``app/config/parameters_symfonycloud.php``.

Deployment
----------

Don't forget to commit the new files in your repository:

.. code-block:: terminal

    $ git add .symfony.cloud.yaml .symfony/services.yaml .symfony/routes.yaml app/config/parameters_symfonycloud.php php.ini
    $ git commit -m "Add SymfonyCloud configuration"

You can now create a new SymfonyCloud project and deploy it:

.. code-block:: terminal

    $ symfony deploy

.. caution::

    .. include:: ../_includes/trial-warning.rst

Migrations
----------

You want to upgrade to Symfony 4? Switch to Flex? Or even move from Symfony 2.x
to 3.4 and start using runtime environment variables? By using SymfonyCloud you
can now :ref:`spin-up new environments <working-with-environments>` to quickly
test in a production-like environment your changes and then deploy with
confidence.

Additionally, the great advantage of using SymfonyCloud before migrating to Flex
is that you gain the ability to synchronize the code upgrade and the
infrastructure change: for example you can synchronize the switch from
``app/cache`` to ``var/cache`` or from ``web`` to ``public`` in the code and in
the ``.symfony.cloud.yaml`` at the same time. You can do progressive and small
iterations instead of planning a hard-core migration maintenance window.

Here are several small tweaks we implemented to assist you in this migration:

* SymfonyCloud will automatically detect if it should use ``app/console`` or
  ``bin/console``. You can do the same by using ``symfony console``.
* :ref:`symfony-deploy` will automatically build the cache if required. This
  allows you to keep the same configuration from Symfony 2 up to Symfony 4.
* SymfonyCloud will automatically detect if ``symfony/flex`` is installed and
  how cache should be warmed up.
* SymfonyCloud will synchronize ``SYMFONY_ENV`` and ``APP_ENV`` as well as
  ``SYMFONY_DEBUG`` and ``APP_DEBUG``.
* SymfonyCloud will check for Symfony's cache in ``var/cache`` if
  ``symfony/flex`` is installed and will fallback to ``app/cache`` otherwise.
  You can define ``SYMFONY_APP_CACHE_DIR`` to override where the cache is
  stored.
* If SymfonyCloud finds some pre-warmed up cache it will use it, regardless of
  the Symfony version you are using.

.. _Runtime Environment Variables: https://symfony.com/blog/new-in-symfony-3-2-runtime-environment-variables
.. _DotEnv component: https://symfony.com/blog/new-in-symfony-3-3-dotenv-component
.. _Symfony 3.2: https://symfony.com/releases/3.3
.. _Symfony 3.3: https://symfony.com/releases/3.3
.. _Symfony 3.4: https://symfony.com/releases/3.4
