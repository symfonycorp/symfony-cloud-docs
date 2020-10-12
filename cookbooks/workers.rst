Workers
=======

**Workers** (or consumers) are a great way to off-load processing in the
background to make a website as snap as possible. Implementing workers in
Symfony is made easy thanks to the `Messenger component
</doc/current/components/messenger.html>`_. This is why deploying workers is a
first-class use-case with SymfonyCloud.

Basics
------

To deploy a worker, add an entry under the ``workers`` section:

.. code-block:: yaml

    # .symfony.cloud.yaml
    workers:
        mails:
            commands:
                start: symfony console messenger:consume --time-limit=60 --memory-limit=128M

On SymfonyCloud, worker containers run the exact same code as the web container.
The container image is built only once, and then deployed multiple times in its
own container along the web one. The *build* hook and dependencies may not vary
but as these containers are independent they can be customized the same way
using common properties (default values are the one defined for the main
container):

- relationships
- size
- disk
- mount
- variables
- commands

The ``commands.start`` key is required and specifies the command to use to
launch the application worker. It may be any valid shell command. If the command
specified by the ``start`` key terminates it will be restarted automatically.

.. tip::

    Running the ``symfony-deploy`` command before starting your worker is not
    necessary anymore, SymfonyCloud takes care of running it automatically.

.. caution::

    Web and worker containers do not share :ref:`mounts <mounts>` targets.
    Sharing files between those containers using the filesystem is not
    possible. Every data sharing needs to be done using :doc:`services
    </services/intro>`.

.. tip::

   While worker and application containers share the same ``size`` by default,
   they usually benefit from being tweaked separately. Read the documentation
   about :doc:`resources allocations <resources_allocation>` for more
   details about using it.

Accessing a worker container
----------------------------

As for the web container, developers have access to container logs and console
using SSH. Using the Symfony CLI, one would use the ``--worker`` flag to target
a specific worker. This flag uses the name of the worker, as defined by the key
used in the ``workers`` section.

This flag is available for the following commands:

- ``symfony env:cp``
- ``symfony env:logs``
- ``symfony env:sql``
- ``symfony env:ssh``
- ``symfony db:dump``
- ``symfony tunnel:open``
- ``symfony tunnel:close``
- ``symfony tunnel:info``
