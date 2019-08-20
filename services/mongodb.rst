MongoDB
=======

`MongoDB`_ is a cross-platform, document-oriented database.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    mydatabase:
        # supported versions: 3.0
        type: mongodb:3.0
        disk: 1024

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        mongodb: "mydatabase:mongodb"

.. note::

    The minimum disk size for Mongo DB is ``512`` (MB).

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``MONGODB`` is the upper-cased version of the key defined in the relationship
above):

* ``MONGODB_SERVER``
* ``MONGODB_HOST``
* ``MONGODB_PORT``
* ``MONGODB_SCHEME``
* ``MONGODB_NAME``
* ``MONGODB_DATABASE``
* ``MONGODB_USER``
* ``MONGODB_USERNAME``
* ``MONGODB_PASSWORD``

.. _`MongoDB`: https://en.wikipedia.org/wiki/MongoDB
