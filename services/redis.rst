Redis
=====

`Redis`_ is a high-performance in-memory object store, well-suited for application
level caching.

SymfonyCloud supports two different Redis configurations: One **persistent**
(useful for key-value application data) and one **ephemeral** (in-memory only,
useful for application caching).

.. note::

    Versions 3.0 and higher support up to 64 different databases per instance of
    the service, but Redis 2.8 is configured to support only a single database.

Ephemeral Redis
---------------

The ``redis`` service type is configured to serve as a LRU cache with the
eviction policy ``allkeys-lru`` - its storage is not persistent. It is not
suitable for use except as a disposable cache.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    rediscache:
        # supported versions: 3.2, 4.0, 5.0
        # 2.8 and 3.0 are also available but not maintained upstream
        type: redis:5.0

Data in an Ephemeral Redis instance is stored only in memory, and thus requires
no disk space. When the service hits its memory limit it will automatically
evict old cache items to make room for new ones.

Persistent Redis
----------------

The ``redis-persistent`` service type is configured for persistent storage. That
makes it a good choice for fast application-level key-value storage.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    redisdata:
        # supported versions: 3.2, 4.0, 5.0
        # 2.8 and 3.0 are also available but not maintained upstream
        type: redis-persistent:5.0
        disk: 1024

The ``disk`` key is required for redis-persistent to configure how much disk
space to reserve for Redis' persistent data.

Wire Redis in your application
------------------------------

To wire Redis in ``.symfony.cloud.yaml`` (don't forget to enable the ``redis``
PHP extension):

.. code-block:: yaml

    runtime:
        extensions:
            - redis

    relationships:
        redis: "myredis:redis"

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``REDIS`` is the upper-cased version of the key defined in the relationship
above):

* ``REDIS_URL``: The Redis URL
* ``REDIS_HOST``: The Redis host
* ``REDIS_PORT``: The Redis port
* ``REDIS_SCHEME``: The Redis scheme

.. _`Redis`: https://en.wikipedia.org/wiki/Redis
