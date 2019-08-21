Environment Variables
=====================

SymfonyCloud exposes information to your application via environment variables,
including database credentials, the SMTP address, and more.

SymfonyCloud also lets you set environment variables that are independent of
the project's code base, but available at build and/or runtime for your code to
leverage.

Levels and Scopes
-----------------

Environment variables can be defined at four different levels, defining four
scopes:

* Global environment variables (set by SymfonyCloud);
* Project environment variables;
* Branch environment variables;
* Application environment variables;
* Shell Environment Variables.

SymfonyCloud provided values override values set from the shell, which override
the ones set for an environment, which override the ones set at project-level,
which override the ones from the application.

When listing all defined environment variables, visibility and information
about where the variable was defined and inheritance is given:

.. code-block:: terminal

    $ symfony vars

    ┌────────────────────────────┬─────────────────────────┬────────────┬────────────┬───────────┐
    │             ID             │          Value          │ Defined in │ Visibility │ Inherited │
    ├────────────────────────────┼─────────────────────────┼────────────┼────────────┼───────────┤
    │ env:PROJECT_VAR            │ bar                     │ Project    │ always     │ -         │
    │ env:APP_DEBUG              │ 1                       │ Env        │ deploy     │ No        │
    │ env:APP_ENV                │ prod                    │ Env        │ deploy     │ Yes       │
    │ env:APP_SECRET             │ 0e0c56ec8c6bdb8e3f29bc3 │ Env        │ deploy     │ Yes       │
    │ env:BLACKFIRE_SERVER_ID    │ abe7e442-23e0-4daf-bc13 │ Env        │ deploy     │ Yes       │
    │ env:BLACKFIRE_SERVER_TOKEN │ 765a496493cabc94fcf6cbf │ Env        │ deploy     │ Yes       │
    │ env:RECAPTCHA_SECRET_KEY   │ 6LcRx0oUAAAAAJD5fe8kGhO │ Env        │ deploy     │ Yes       │
    └────────────────────────────┴─────────────────────────┴────────────┴────────────┴───────────┘

Accessing Environment Variables
-------------------------------

You can access environment variables using standard tools::

    echo $_SERVER['PROJECT_VAR'];

.. code-block:: terminal

    $ symfony ssh
    $ echo $PROJECT_VAR

Project Environment Variables
-----------------------------

Project environment variables are available **both** at build time **and** at
runtime, and are the same for all environments in the project. Set one with the
following:

.. code-block:: terminal

    $ symfony var:set FOO=bar

    # set several variables
    $ symfony var:set FOO=bar BAR=foo

Project environment variables are a good place to store secret information that
is needed at **build time**, such as credentials for a private 3rd party code
repository.

You can suppress runtime visibility with the ``--hide-at-runtime`` flag.

.. caution::

    For project environment variables to take effect, you need to redeploy.

Branch Environment Variables
----------------------------

Branch environment variables are bound to a specific Git branch/environment. An
environment will also inherit variables from its parent environment, unless it
has a variable defined with the same name. Set a branch environment
variable via the following:

.. code-block:: terminal

    $ symfony var:set FOO=bar --env-level

That will set a variable on the **currently active environment** (the Git
branch you have checked out).

Setting or changing an environment variable causes that environment to be
redeployed. However, it will not redeploy any child environments.

Environment variables are a good place to store values that apply only on
SymfonyCloud and not on your local development environment. This includes API
credentials for 3rd party services, mode settings if your application has a
separate "dev" and "prod" runtime toggle, ...

.. tip::

    If your project requires some variables only for production (ie:
    credentials, production flags, etc) you can define them at the environment
    level and disable inheritance using ``symfony var:set --env-level --inheritable=false``

Removing Environment Variables
------------------------------

Remove environment variables via the following command:

.. code-block:: terminal

    # for a project level var
    $ symfony var:delete PROJECT_VAR

    # or for a branch level var
    $ symfony var:delete PROJECT_VAR --env-level

Application Environment Variables
---------------------------------

Application environment variables are defined for a specific application of
your project and can be defined in the ``.symfony.cloud.yaml`` file:

.. code-block:: yaml

    variables:
        env:
            APPLICATION_VAR: 'foo'

These values will be the same across all environments and present in the Git
repository, which makes them a poor fit for API keys and such. This capability
is mainly for :doc:`multi-applications projects </cookbooks/multi-apps>` where
the value should be different for each application.

Application-provided variables are available **at both** build time **and**
runtime.

SymfonyCloud Environment Variables
----------------------------------

SymfonyCloud exposes some environment variables about the application, all
prefixed with ``SYMFONY_*``:

* ``SYMFONY_APP_DIR``: The absolute path to the application directory.

* ``SYMFONY_APPLICATION_NAME``: The name of the application, as configured
  in ``.symfony.cloud.yaml``

* ``SYMFONY_PROJECT``: The ID of the project

* ``SYMFONY_TREE_ID``: The ID of the tree the application was built from.
  It's essentially the SHA hash of the tree in Git. If you need a unique ID for
  each build, this is the value you should use.

* ``SYMFONY_PROJECT_ENTROPY``: A random value created when the project is
  first created, which is then stable throughout the project’s life.

The following variables exist **only** at runtime. If used in a build hook they
will evaluate to an empty string like any other unset variable:

* ``SYMFONY_BRANCH``: The name of the Git branch.

* ``SYMFONY_DOCUMENT_ROOT``: The absolute path to the web document root, if
  applicable.

* ``SYMFONY_ENVIRONMENT``: The name of the environment generated by the name of
  the Git branch.

* ``SYMFONY_DEFAULT_ROUTE_URL``: The default endpoint serving your application.
  It can be used to avoid hard-coding domains that can't be used to reach
  non-production environments. Parts of the URL (scheme, domain, port and path)
  are also exposed each in their own variable whose name starts with
  ``SYMFONY_DEFAULT_ROUTE_`` followed by the name of the part.

SymfonyCloud Services Environment Variables
-------------------------------------------

SymfonyCloud also exposes information about services via environment variables
(check each service documentation to learn more about the specifics). List all
SymfonyCloud environment variables with the following:

.. code-block:: terminal

    $ symfony ssh -- symfony var:export

Shell Environment Variables
---------------------------

You can provide a ``.environment`` file as part of your application, in your
application root. That file will be sourced as a bash script when the container
starts and on all SSH logins. It can be used to set any environment variables
directly, such as the ``PATH`` variable:

.. code-block:: terminal

    $ export PATH=/app/vendor/bin:$PATH
