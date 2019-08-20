RabbitMQ
========

`RabbitMQ`_ is an Open-Source message broker software (sometimes called
message-oriented middleware) that implements the Advanced Message Queuing
Protocol (AMQP).

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    myrabbitmq:
        # supported versions: 3.5, 3.6, 3.7
        type: rabbitmq:3.7
        disk: 1500

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        rabbitmq: "myrabbitmq:rabbitmq"

The configuration is exposed via the following environment variables (where
``RABBITMQ`` is the upper-cased version of the key defined in the relationship
above):

* ``RABBITMQ_URL``: The RabbitMQ standardized URL
* ``RABBITMQ_SERVER``: The RabbitMQ server
* ``RABBITMQ_HOST``: The RabbitMQ host
* ``RABBITMQ_PORT``: The RabbitMQ port
* ``RABBITMQ_SCHEME``: The RabbitMQ scheme
* ``RABBITMQ_USER``: The RabbitMQ username
* ``RABBITMQ_USERNAME``: The RabbitMQ username
* ``RABBITMQ_PASSWORD``: The RabbitMQ passsword

Connecting to RabbitMQ
----------------------

From the Application Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the defined environment variables to configure your framework of choice.

.. note::

    The application container doesn't include any useful utilities to connect to
    RabbitMQ with. However, you can add a client as a dependency in your
    ``.symfony.cloud.yaml`` file, like `amqp-utils
    <https://github.com/dougbarth/amqp-utils/>`_:

    .. code-block:: yaml

        dependencies:
            ruby:
                amqp-utils: "0.5.1"

    Then, use any ``amqp-*`` commands when connected via SSH.

Management UI
-------------

We ship RabbitMQ with the `Management UI
<https://www.rabbitmq.com/management.html>`_ enabled by default.

The configuration to access it from your containers is exposed via the same
set of environment variables but prefixed with ``RABBITMQ_MANAGEMENT`` (where
``RABBITMQ`` is the upper-cased version of the key defined in the
relationship).

To access this UI from your local machine, use the Symfony CLI to open an SSH
tunnel:

.. code-block:: terminal

   $ symfony tunnel:open
   SSH tunnel opened on port 30000 to relationship rabbitmq
   SSH tunnel opened on port 30001 to relationship rabbitmq-management

.. tip::

    Use ``symfony tunnel:info`` to discover username and password to access the UI.

Troubleshooting
---------------

Messages not being received
~~~~~~~~~~~~~~~~~~~~~~~~~~~

RabbitMQ probably doesn't have enough disk space allocated to it. On
SymfonyCloud, the service writes all of its data to disk, and thus needs a disk
value that is greater than its RAM allocation. For most Master branches, the
service needs at least 1300 MB of disk space to work properly.

.. _`RabbitMQ`: https://en.wikipedia.org/wiki/RabbitMQ
