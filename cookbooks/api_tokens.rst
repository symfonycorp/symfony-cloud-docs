API Tokens
==========

Managing Tokens
---------------

The Symfony CLI Tool supports an API Token authentication option to allow
it to be used from CI services, automation tools, and directly from application
containers.

API tokens are managed on `symfony.com <https://symfony.com/account/security>`_.

The Symfony CLI Tool read the API token from the ``SYMFONY_TOKEN``
environment variable.

.. note::

    When running commands from any automated system, use the ``--no-wait`` flag
    to avoid blocking the process. When running a command in a SymfonyCloud cron
    task, **always** use the ``--no-wait`` flag.

Using the CLI Tool on Application Containers
--------------------------------------------

A common use case for API tokens is to allow the Symfony CLI Tool to be run
on application containers.

Set the ``SYMFONY_TOKEN`` environment variable first:

.. code-block:: terminal

    $ symfony var:set SYMFONY_TOKEN=API_KEY_VALUE --sensitive --env=master

You can now run a command from within the shell on the application container,
or via a cron hook (the project and the environment are auto-detected).

Some typical use cases: :doc:`create snapshots <snapshots>` and :doc:`renew SSL
certificates <https>`.
