Troubleshooting
===============

Permission denied (publickey)
-----------------------------

SymfonyCloud leverages the SSH protocol to access running containers and Git
over SSH for project deployments.

They are four possible reasons to why you might encounter this error:

#. Your public key has not been uploaded to SymfonyCloud. Check that your SSH
   key is properly uploaded using ``symfony account:ssh:keys``. If your key is
   not present (or if you are not sure) use ``symfony account:ssh:key:add`` to
   upload it.

   .. caution::
       Please note that SymfonyCloud and SymfonyConnect SSH keys are currently
       differentiated: even if you previously uploaded your key to SymfonyConnect,
       you have to reupload it using the command mentionned above.

#. SSH keys propagation can take some time and you've been too fast. Try again
   to use ``symfony ssh`` or ``symfony deploy`` after a couple of
   minutes and it should work.
#. You've been recently added to a new project (**only impacts**
   ``symfony ssh``): after adding new users to a project, a new deployment
   or a redeployment is required before the new users can access the project.
   If no deployment occured since you've been granted access to the project,
   use ``symfony redeploy``, wait for the end of the deployment and try
   again to access the project (this must be done for every environment).
#. You are using Windows.

   * When using ``symfony ssh``, the SymfonyCloud client will make its
     best to use PuTTY registered SSH keys (connecting to Pageant) and OpenSSH
     keys saved in ``$HOME/.ssh`` (mimicking Unix). You can use
     ``symfony ssh -vvv`` to debug which keys are loaded by the SymfonyCloud
     client.

   * When using ``symfony deploy``, some work is handed of to Git and the
     details then depends on your setup. We recommend you to use the following
     snippet to debug your issue:

     .. code-block:: env

         set GIT_SSH_COMMAND=ssh -vvv
         set GIT_TRACE=2
         git ls-remote test@git.eu.s5y.io:test.git

Host key verification failed
----------------------------

**This issue usually happens only on Windows**

As for SSH authentication, on Windows, SymfonyCloud uses both PuTTY and OpenSSH
directories for SSH host key verification, but ``symfony deploy`` is based on
Git which is not using the same process. If you encounter this error message
you can use ``git ls-remote test@git.eu.s5y.io:test.git`` to trigger a new
connection and force Git to register the key.

Cannot decode encrypted private keys
------------------------------------

See `Unsupported key type`_.

Unsupported key type
--------------------

SymfonyCloud leverages the SSH protocol to access running containers and Git
over SSH for project deployments and might have to read your SSH keys to do so.

This error can happen in a very specific situation if the two following
conditions are met:

- Your key is stored in the OpenSSH format; as opposed to the PEM format (in
  this case, the first line will look like
  ``-----BEGIN OPENSSH PRIVATE KEY-----``)
- Your key uses the DSA or ECDSA algorithm (``ssh-keygen -l -f <file>`` will
  tell you which algorithm is in use)

This use case is not yet supported by the upstream library we use. In the
meantime you have two solutions:

#. Run an SSH agent and load your key using ``ssh-add <file>``
#. Convert your private key back to PEM format: ``ssh-keygen -p -m pem -f <file>``

   .. caution::

      This command upgrade your key inline. Please do a backup before!

.. note::

   DSA keys are considered insecure and ECDSA security assessment depends on
   your computer ability to generate good random numbers. So alternatively to
   the two solutions mentioned before, you can also migrate to the Ed25519
   algorithm but keep in mind this might require quite some work from you as
   you will end up with a new public key to deploy everywhere.

Environment variable not found (during the build step)
------------------------------------------------------

SymfonyCloud leverages Symfony `Runtime Environment Variables
</doc/current/configuration/external_parameters.html#config-env-vars>`_
defining services connection information via environment variables (read the
:ref:`Connecting to a service <connecting-service>` from the services
documentation for more information).

Additionally, to make containers reusable from one environment to another and to
make (re-)deployments faster, the build step has no services started. Therefore the
environment variables to connect to services are not available during the build
step.

It might happen that an application build breaks because of an environment
variable that does not exist:

.. We use this on purpose to **not** include the line numbers
.. literalinclude:: missing-env-variable-logs.txt
   :language: text

This usually happens with applications migrated from Symfony pre 3.2 to 3.2 and
upper. When migrating, one needs not to forget to define defaults in the
``.env`` file or in the service container configuration:

.. configuration-block::

    .. code-block:: env

        # .env
        DATABASE_URL="mysql://db_user:db_password@127.0.0.1:3306/db_name"

    .. code-block:: yaml

        # config/services.yaml
        parameters:
            env(DATABASE_URL): "mysql://db_user:db_password@127.0.0.1:3306/db_name"

    .. code-block:: xml

        <!-- config/services.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services https://symfony.com/schema/dic/services/services-1.0.xsd">

            <parameters>
                <parameter key="env(DATABASE_URL)">mysql://db_user:db_password@127.0.0.1:3306/db_name</parameter>
            </parameters>
         </container>

    .. code-block:: php

        // config/services.php
        $container->setParameter('env(DATABASE_URL)', 'mysql://db_user:db_password@127.0.0.1:3306/db_name');

.. caution ::

   Symfony applications created before November 2018 had a slightly different
   system, involving a ``.env.dist`` file. For information about upgrading, see:
   `Nov 2018 Changes to .env & How to Update
   </doc/current/configuration/dot-env-changes.html>`_.

SQLSTATE[HY000] [2002] Connection refused (during the build step)
-----------------------------------------------------------------

This error message is usually due to a known Doctrine bug/limitation.

Some parts of Doctrine configuration (such as Entities metadata build) need to
know the exact database version to be built. This is usually done during Symfony
cache warmup for more efficiency but this requires the database to be
available, which is not is the case on SymfonyCloud.

.. tip::

   One can reproduce this issue locally by shutting down their database and try
   clearing Symfony cache.

The best long-term solution to this problem is to make sure no connection to
the database is done during Symfony cache warmup. However, this solution might
require to fix some third-party code or not be possible at all.

In this case, the best workaround to date is to explicitly tell Doctrine which
database platform and version the application is going to use. This can be done
either by updating the default ``DATABASE_URL`` in the ``.env`` file by
appending ``?serverVersion=`` and the targeted version or by updating the
Doctrine semantic configuration under the keys ``driver`` and
``server_version``:

.. configuration-block::

    .. code-block:: env

        # .env
        ###> doctrine/doctrine-bundle ###
        DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name?serverVersion=10.2.0
        ###< doctrine/doctrine-bundle ###

    .. code-block:: yaml

        # config/packages/doctrine.yaml
        doctrine:
            dbal:
                driver: pdo_mysql
                server_version: '10.2.0'

.. caution::

   Don't forget to keep this value in sync with the version configured in
   ``.symfony/services.yaml``.
