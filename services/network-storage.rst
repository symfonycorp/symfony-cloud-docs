Network Storage
===============

The network storage service provides a file store and enables a new kind of
``mount`` that can be shared between different application containers.

To use it in your project, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    files:
        type: network-storage:1.0
        disk: 256

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    mounts:
        '/var': { source: local, source_path: var }
        '/public/uploads': { source: service, service: files, source_path: uploads }

The ``public/uploads`` directory can now be read from and written to as usual
and several containers can access this directory and share the **same** files.
The ``var`` directory however stays "local" and will not be shared between
different containers.

.. note::

   You do **not** need to add a relationship to point to the ``files`` service.
   This is handled automatically by SymfonyCloud.

.. note::

    There is a small performance impact for using a network mount over a local
    mount, but in most cases it should not be noticeable. Please note that
    high-volume sequential file creation may experience a more significant
    performance impact.

How can I migrate a Local Storage to a Network Storage?
-------------------------------------------------------

There is no automated way of transferring data from one storage type to
another. The process consists in moving files around.

Suppose you have this mount configuration and want to move to a network storage
mount:

.. code-block:: yaml

    mounts:
        '/var': { source: local, source_path: var }
        '/public/uploads': { source: local, source_path: uploads }

The following steps will do so with a minimum of service interruption.

1. Add a new ``network-storage`` service named ``files``, that has at least
   enough space for your existing files with some buffer.

   .. caution::

       You may need to increase your project's allocated storage to do this.
       This operation is not reversible and will incur charges.

2. Add a new mount to the network storage service on a non-public directory:

   .. code-block:: diff

       mounts:
           '/var': { source: local, source_path: var }
           '/public/uploads': { source: local, source_path: uploads }
    +      '/new-uploads': { source: service, service: files, source_path: uploads }

3. Deploy the changes;
4. Connect to your container using ``symfony ssh``;
5. Use ``rsync`` to copy all files from the local mount to the network mount:

   .. code-block:: terminal

      $ rsync -avz web/uploads/* new-uploads/

6. Reverse the mounts, commit and deploy:

   .. code-block:: diff

       mounts:
           '/var': { source: local, source_path: var }
    -      '/public/uploads': { source: local, source_path: uploads }
    +      '/old-uploads': { source: local, source_path: uploads }
    -      '/new-uploads': { source: service, service: files, source_path: uploads }
    +      '/public/uploads': { source: service, service: files, source_path: uploads }

7. Run ``rsync`` again to make sure all files uploaded during the transition are moved.

   .. code-block:: terminal

      $ rsync -avz old-uploads/* web/uploads/

8. Once you're confident that all files are accounted for, delete the entire
   contents of ``old-uploads``.

9. Once done, you can remove the ``old-uploads`` mount and push again to
   finish the process. You are also free to reduce the ``disk`` size in the
   ``.symfony.cloud.yaml`` file if desired.
