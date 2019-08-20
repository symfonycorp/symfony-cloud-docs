Kafka
=====

`Apache Kafka`_ is an Open-Source stream-processing software platform. It is a
framework for storing, reading and analyzing streaming data. It exposes an HTTP
API for client interaction. See the `Kafka documentation`_ for more information.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    mykafka:
        # supported versions: 2.1
        type: kafka:2.1
        disk: 1024

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        kafka: "mykafka:kafka"

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``KAFKA`` is the upper-cased version of the key defined in the relationship
above):

* ``KAFKA_URL``
* ``KAFKA_SCHEME``
* ``KAFKA_HOST``
* ``KAFKA_PORT``
* ``KAFKA_IP``

.. _`Apache Kafka`: https://kafka.apache.org/
.. _`Kafka documentation`: https://kafka.apache.org/documentation/
