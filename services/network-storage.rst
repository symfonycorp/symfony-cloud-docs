Network Storage
===============

The network storage service provides a file store and enables a new kind of
``mount`` that can be shared between different application containers.
application containers.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    files:
        type: network-storage:1.0
        disk: 256

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    mounts:
        "/var": { source: local, source_path: var }
        "/public/uploads": { source: service, service: files, source_path: uploads }

.. note::

   You do **not** need to add a relationship to point to the ``files`` service.
   This is handled automatically by SymfonyCloud.

The application can now read from and write to the ``public/uploads`` directory
as if it was a local writeable mount.

.. note::

    There is a small performance hit for using a network mount over a local
    mount but in most cases it should not be noticeable. Please note that
    high-volume sequential file creation may experience a more significant
    performance hit.

How can I migrate a local storage to a network storage?
-------------------------------------------------------

There is no automated way of transferring data from one storage type to
another. Fundamentally, the process is consists in moving files around.

Suppose you have this mount configuration and want to move to a network storage
mount:

.. code-block:: yaml

    mounts:
        "/var": { source: local, source_path: var }
        "/public/uploads": { source: local, source_path: uploads }

The following approximate steps will do so with a minimum of service interruption.

1. Add a new ``network-storage`` service named `files`, that has at least
   enough space for your existing files with some buffer

   .. note::

       You may need to increase your project's allocated storage to do
       this.

2. Add a new mount to the network storage service on a non-public directory:

   .. code-block:: yaml

       mounts:
           "/var": { source: local, source_path: var }
           "/public/uploads": { source: local, source_path: uploads }
           "/new-uploads": { source: service, service: files, source_path: uploads }

3. Deploy the changes.
4. Connect to your container using ``symfony ssh``
5. Use ``rsync`` to copy all files from the local mount to the network mount:

   .. code-block:: terminal

      $ rsync -avz web/uploads/* new-uploads/

5. Reverse the mounts, commit and deploy:

   .. code-block:: yaml

       mounts:
           "/var": { source: local, source_path: var }
           "/old-uploads": { source: local, source_path: uploads }
           "/public/uploads": { source: service, service: files, source_path: uploads }

5) Run rsync one more time to make sure any files uploaded during the
   transition are not lost.

   .. code-block:: terminal

      $ rsync -avz old-uploads/* web/uploads/

6. Once you're confident all the files are accounted for, delete the entire
   contents of ``old-uploads``. If you do not, the files will remain on disk
   but inaccessible, just eating up disk space needlessly.

7. Once that is done you can remove the ``old-uploads`` mount and push again to
   finish the process. You are also free to reduce the ``disk`` size in the
   ``.symfony.cloud.yaml`` file if desired.
