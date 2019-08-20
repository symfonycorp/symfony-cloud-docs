HTTP
====

If your project has :doc:`multiple applications</cookbooks/multi-apps>`, you can
link them via ``relationships`` in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        database: "securitydb:postgresql"
        some-service: "some-server:http"

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``SOME_SERVICE`` is the upper-cased version of the key defined in the
relationship above):

* ``SOME_SERVICE_URL``: The full URL of the service
* ``SOME_SERVICE_IP``: The HTTP service IP
* ``SOME_SERVICE_PORT``: The HTTP service port
* ``SOME_SERVICE_SCHEME``: The HTTP service scheme
* ``SOME_SERVICE_HOST``: The HTTP service host

.. note::

    Try connecting with ``curl $SOME_SERVICE_URL/``.
