.symfony/services.yaml
======================

.. toctree::
    :maxdepth: 1
    :titlesonly:

    emails
    http
    mysql
    postgresql
    redis
    memcached
    elasticsearch
    rabbitmq
    mongodb
    influxdb
    solr
    varnish
    kafka

Configure Services
------------------

SymfonyCloud allows you to completely define and configure the topology and
**services** you want to use on your project.

Unlike other PaaS services, SymfonyCloud is **batteries included** which means
that you don't need to subscribe to an external service to get a cache or a
search engine. And that those services are managed. When you back up your
project, all of the services are backed-up.

Services are configured through the ``.symfony/services.yaml`` file you will
need to commit to your Git repository. This section describes specifics you
might want to know about for each service.

Here is an example of a ``.symfony/services.yaml`` file:

.. code-block:: yaml

    database1:
        type: mysql:10.1
        disk: 2048

    database2:
        type: postgresql:9.6
        disk: 1024

In order for a service to be available to an application in your project
(SymfonyCloud supports not only multiple backends but also multiple applications
in each project) you will need to refer to it in the ``.symfony.cloud.yaml`` file
under the ``relationships`` key.

Name
~~~~

The `name` you want to give to your service. You are free to name each service
as you wish (*lowercase alphanumeric only*).

.. caution::

    Because we support multiple services of the same type (you can have 3
    different MySQL instances), changing the name of the service in
    ``services.yaml`` will be interpreted as destroying the existing service and
    creating a new one. This will make **all the data in that service disappear
    forever**. Remember to always snapshot your environment in which you have
    important data before modifying this file.

Type
~~~~

The ``type`` of your service in the following format: ``type:version``.

Disk
~~~~

The ``disk`` attribute is the size of the persistent disk (in MB) allocated to
the service. The default minimum disk size is 16MB but each service can
redefine this minimum if necessary.

For example, the current default storage amount per project is 5GB (meaning
5120MB) which you can distribute between your application (as defined in
``.symfony.cloud.yaml``) and each of its services. For memory-resident-only
services such as ``memcache`` or ``redis``, the disk key is not required and will
generate an error if present.

.. note::

    Currently we do not support downsizing the persistent disk of a service.

Size
~~~~

This attribute has the same effect as for :ref:`application containers <config-size>`.

.. tip::

    All services have their system timezone set to ``UTC`` by default.

Relationships
-------------

Services are exposed to the application using environment variables.

See the :ref:`relationships` section of the ``symfony.cloud.yaml``
documentation for more information.

.. _connecting-service:

Connecting to a service
-----------------------

Once a service is running and exposed as a relationship, its appropriate
credentials (host name, username if appropriate, etc.) will be exposed through
environment variables. The exact names are documented on the appropriate
service's page.

Be aware that the environment variable names are fixed but their values may
change on any deployment or restart. Never hard-code connection credentials for
a service into your application. You should use the environment variables every
time your script or application starts.

To connect to a remote service from your local computer, the easiest way is to
use the Symfony CLI to open an SSH tunnel:

.. code-block:: terminal

    $ symfony tunnel:open
    # Get connection information and credentials:
    $ symfony tunnel:info

Use the ``tunnels`` command to list all open tunnels:

.. code-block:: terminal

    $ symfony tunnels

Available services
------------------

* :doc:`mysql`
* :doc:`postgresql`
* :doc:`redis`
* :doc:`memcached`
* :doc:`elasticsearch`
* :doc:`rabbitmq`
* :doc:`mongodb`
* :doc:`influxdb`
* :doc:`solr`
* :doc:`varnish`
* :doc:`kafka`
