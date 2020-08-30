Continuous Delivery
===================

Continuous Delivery is an approach that aims at releasing software faster and
more frequently. SymfonyCloud can help you implement this methodology in several
ways we are going to detail in this document.

.. note::

   Continuous Delivery is not to be confused with Continuous Integration or
   automatic testing (even though they can work together): SymfonyCloud will
   **not** run your tests.

SymfonyCloud provides environments as replicas of your main environment (see
`What is SymfonyCloud? <basics>`_), one will be able to spin-up new environments
based on their use-case and apply infrastructure changes to the SymfonyCloud
configuration. Because this configuration `is part of their code <config>`_, the
changes will be recorded in the code versioning and can be integrated within the
regular code workflow: commit, push, pull/merge requests, code review, etc. Once
the modification is accepted and integrated into the main branch and deployed,
SymfonyCloud uses the new code and the new configuration at once.

.. caution::

   The number of environments that can be created for a given project is limited
   depending on its quota (3 by default). The quota can increased using
   ``symfony project:edit environment X`` (where X must be replaced by the
   desired number - a multiple of 3) and will impact the project's billing based
   on `SymfonyCloud pricing <https://symfony.com/cloud/pricing>`_.

.. tip::

   As a bonus, when SymfonyCloud deploys an application, an image for the
   current tree (the combination of the code and the configuration) is first
   looked up. If a previous build for the same tree is found, this image is
   reused right away leading to faster deployments and more confidence as the
   exact image has been validated.

When implementing Continuous Delivery with SymfonyCloud, the changes to the
infrastructure configuration are expected to be mostly manual depending on your
needs. The automation will focus on creating environments and deploying the
changes. There are two possible approaches:

* Fully automated using our `integrations`_
* `Custom implementation`_ using the
  `Symfony CLI <https://symfony.com/download>`_ commands.

Integrations
------------

SymfonyCloud provides integrations with the main Git providers of the market.
Those integrations rely on webhooks those providers trigger on specific events
such as code push, branch creation or pull request opening: environments are
created and deleted on the fly when branches are created or deleted, code and
configuration changes are deployed automatically on pushes. Depending on the
configuration on SymfonyCloud side, only specific branches can be deployed or
pull/merge requests can be deployed or skipped.

.. tip::

   To ease reviewers works, for most Git providers, integrations push links to
   the deployed environments as a commit status or as a comment to the
   discussion.

Please refer to :doc:`our integrations list </integrations/index>` to discover
the complete list of the available integrations and their specific configuration.

Custom implementation
---------------------

For some projects, it might be more appropriate to use a custom implementation.
With a custom implementation one project can have a semi-automated or a
completely automated delivery pipeline based on the requirements. In this
context, the final logic will depend on the project requirements but you will
find bellow a list of best practices, tips and pointers to help you in your
implementation.

* `Requirements <cd-requirements>`_
* `Create and use a machine user for authentication <machine-user>`_
* `CLI flags and environment variables <cd-env-flags>`_
* `Creating and deleting environments <cd-env-create-delete>`_
* `URLs <cd-url>`_

.. _cd-requirements:

Requirements
^^^^^^^^^^^^

The system implementing a custom SymfonyCloud deployment automation will have
several requirements:

#. Have the `Symfony CLI <https://symfony.com/download>`_ installed and runnable
#. Have Git installed and runnable
#. Have `authentication`_ completed

Authentication
^^^^^^^^^^^^^^

If the automation is expected to be run interactively by human beings, they must
first - locally on their machine - setup Git and run ``symfony login`` as
well as ``symfony account:ssh:key:add``.

.. _machine-user:

If the implementation is to be run in a non-interactive environment (in a system
like Jenkins, Travis, GitLab CI or GitHub Actions for example), we recommend the
use of a "Machine user":  head to `SymfonyConnect <https://connect.symfony.com>`_
and register a new dedicated account.

As this context will not be interactive, interactive authentication will not be
possible, this is why you have to use an :doc:`API token
</cookbooks/api_tokens>`.

Additionally, as deployment are based on Git, you will
have to generate a new SSH key, and register it in SymfonyCloud:

.. code-block:: terminal

   $ ssh-keygen -N '' -f symfonycloud-ci.key
   $ SYMFONY_TOKEN=xxxx symfony account:ssh:key:add symfonycloud-ci.key

.. caution::

   Don't forget to register and expose the ``SYMFONY_TOKEN`` environment value
   as well as the generated SSH key in your automation system.

When running ``symfony deploy``, SSH host keys are checked to ensure not MITM
attack is happening. This security measure requires you to register the
following SSH host key as a know key in your automation system (usually in
``~/.ssh/known_hosts``):

.. literalinclude:: ../_includes/ssh-host-key-eu.txt
  :language: text

.. caution::

   Disabling SSH hosts key checking or using ``ssh-keyscan`` in your automation
   pipeline to register the SSH host key breach several security measures and
   should not be done.

.. tip::

   Even if this is unlikely, this key might change in the future. You can use
   ``ssh-keyscan git.eu.s5y.io`` to ensure its freshness.

CLI behavior
^^^^^^^^^^^^

When the CLI tool detects it is currently running in an automation tool,
interaction is disabled. If you run a command that requires some input, default
values are going to be used.

In order to determine this the CLI checks the following:

* stdin is a terminal or not
* ``DEBIAN_FRONTEND`` environment variable is set to ``noninteractive``
* Presence of several environment variable based on commonly used by CI systems
  (``BUILD_ID``, ``GITLAB_CI``, ``TRAVIS``, ``CIRCLECI``, etc)
* ``SHELL_INTERACTIVE`` environment variables is set to ``''`` (the empty
  string)

If your system is not automatically detected, you can use the
``--no-interaction`` flag.

.. _cd-env-flags:

CLI flags and environment variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CLI tool exposes flags and check some environment variables in order to
allow you to change its behavior. As a general rule, for automated context,
environment variables are preferable. Flags can be used to override environment
variables punctually:

.. code-block:: terminal

   $ export SYMFONY_ENVIRONMENT=pr-1
   $ symfony deploy
   # Deploying to pr-1
   $ symfony deploy --environment=staging
   # Deploying to staging

SYMFONY_PROJECT
"""""""""""""""

Setting this environment value replace the need to run ``symfony project:link``.

Its value should be set to the SymfonyCloud project's id that can be found with
``symfony project:info`` or in the emails sent by SymfonyCloud.

.. tip:: One can also use the ``--project`` flag.

SYMFONY_ENVIRONMENT
"""""""""""""""""""

By default, when SymfonyCloud needs to work on the remote environment, it tries
to find an existing environment whose name matches the current Git branch name.
Changing this logic can be done by setting the ``SYMFONY_ENVIRONMENT``
environment variable to a different value. You can use the ``symfony envs``
commands to list environments currently present on SymfonyCloud.

.. tip:: One can also use the ``--environment`` flag.

SYMFONY_NO_INTERACTION
""""""""""""""""""""""

Can be used to override the automatic detection made by the CLI.

.. tip:: One can also use the ``--no-interaction`` flag.

.. _cd-env-create-delete:

Creating and deleting environments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``symfony env:create <name> <parent>`` command creates a new SymfonyCloud
environment. It is not usually required to provide any argument when running it
locally, but when running inside a script it is best to specify them. The first
argument is the name desired for the environment to be created on SymfonyCloud.
The second argument is the name of the "parent" environment: the new environment
will first be created as a copy of the parent and as result inherits the
parent's code and data.

Deleting an environment can be made using the ``symfony project env:delete
<name>`` command.

.. _cd-url:

Environments URLs
^^^^^^^^^^^^^^^^^

Each SymfonyCloud environment gets a unique subdomain that can not be determined
before its creation. Once created this domain is stable.

This implies that the application code has to consider this: avoid any
hard-coded domain values. Even Symfony parameters should be avoided and the use
of the current request's host should be preferred. This requirement make working
with systems like OAuth, SSO or webhooks more difficult. To help you work with
this, SymfonyCloud exposes a ``SYMFONY_PROJECT_DEFAULT_ROUTE_URL`` environment
variable. More information in `the dedicated section
<symfonycloud-environment-variables>`_.

To determine the available URLs from your automated system, please use the
``symfony env:urls`` command. The ``--first`` flag is available shall you need
only the first reachable URL.
