InfluxDB
========

`InfluxDB`_ is a time series database optimized for high-write-volume use cases
such as logs, sensor data, and real-time analytics. It exposes an HTTP API for
client interaction.


To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    influx:
        # supported versions: 1.2, 1.3, 1.7
        type: influxdb:1.7
        disk: 1024

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        timedb: "influx:influxdb"

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``TIMEDB`` is the upper-cased version of the key defined in the relationship
above):

* ``TIMEDB_SCHEME``
* ``TIMEDB_HOST``
* ``TIMEDB_PORT``
* ``TIMEDB_IP``

.. _`InfluxDB`: https://en.wikipedia.org/wiki/InfluxDB
