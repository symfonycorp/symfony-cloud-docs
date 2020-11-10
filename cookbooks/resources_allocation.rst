Resources Allocation
====================

SymfonyCloud handles resource allocation in a reserved approach, rather than
on-demand. This means that your project (applications plus services) are given a
certain amount of resources and can use as much, or as little, of them as
needed.

Plan Size
---------

Your production plan size determines how much CPU and RAM are available to your
entire production environment.

.. note::

   The development plan has no dedicated resources allocated: the main
   environment is treated like a
   :ref:`development environment <dev-env-sizing>` and container sizes are set
   to ``S``.

You can change your project plan at anytime using the
``symfony project:scale <plan>`` command.

Possible production plan sizes are:

* Standard
* Medium
* Large
* X-Large
* 2X-Large

Each plan comes with increasing CPU and memory compared to the previous one.
Please refer to the `SymfonyCloud pricing page`_ to get the official allowance
coming with each plan.

Container Size
--------------

By default, SymfonyCloud allocates CPU and memory resources to each container
automatically. Some services are optimized for high CPU load, some for high
memory load and applications are treated as balanced. By default, SymfonyCloud
tries to allocate the largest "fair" size possible to all containers, given the
available resources on the plan. That is not always optimal, however, and you
can customize that behavior on any application or service container.

.. caution::

    If the total resources requested by all apps and services is larger than
    what the plan size allows then a production deployment will fail with an
    error.

.. _dev-env-sizing:
.. note::

    This value is always ignored in development environments and set to ``S``.
    It will only take effect in the production environment. This default
    development container size can be changed (paid option) if you need to.
    Please contact `SymfonyCloud Support`_ for more information.

Recommendations
~~~~~~~~~~~~~~~

For the vast majority of applications, the default fair allocation will be the
best option. In particular it will allow the application to grow as the plan
size increases. If a container is set at a fixed size, however, it will not get
any bigger and the extra resources might go to waste.

The main use-case where setting size manually is helpful is when setting
particular containers to a fixed size to indicate that it really doesn't need
much CPU or RAM or if its workload is fixed and doesn't need to scale much.
**Static sites**, **background workers**, and other **low-need containers** can
be set that way to allow more resources to go to other applications and services
containers that really need them. For example, keep your main web container on
the auto size and set your Symfony Messenger consumer sending emails to ``XS``.
A :doc:`../services/redis` container - set to ``S`` - holding backend sessions
for an application where traffic is mostly due to stateless API requests is
another plausible scenario.

Outside of the PHP world, two additional use cases can benefit from setting
these values:

* Shifting memory allocation from the ``memory_ratio`` to ``base_memory`` (see
  the `custom resources allocation`_ section) for applications with a high
  baseline need but a low per-request need, such as Java or Go applications.
* For especially memory intensive per-process applications, setting a higher
  ``memory_ratio`` (see the `custom resources allocation`_ section) will give
  those applications more head room to grow. However, be aware that doing so may
  draw memory away from other applications or services that need it. Try to
  optimize your application before throwing more resources at it.

In both cases you will need to do your own profiling to determine optimal values
for your application.

Service Sizes
~~~~~~~~~~~~~

Service sizes are set via the ``size`` property in :doc:`../services/intro`.
Possible values are ``AUTO`` (the default if not set), ``S``, ``M``, ``L``,
``XL``, ``2XL`` and ``4XL``.

Application Sizes
~~~~~~~~~~~~~~~~~

Application (web and workers containers) sizes are set via the ``size``
properties in :doc:`../config`. Possible values are ``AUTO``
(the default if not set), ``XS``, ``S``, ``M``, ``L``, ``XL``, ``2XL`` and
``4XL``.

These sizes have a fixed allocation of CPU associated with them and a default
RAM one:

==== ==== ===========
Size CPU  Default RAM
==== ==== ===========
XS   0.25 128 MB
S    0.5  192 MB
M    1.0  320 MB
L    2.0  576 MB
XL   4.0  1088 MB
2XL  8.0  2112 MB
==== ==== ===========

.. note::

   The smaller ``XS`` container size is only available when using custom
   resources allocation as describe below in the `custom resources allocation`_
   section.

Custom Resources Allocation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::

   The default values are appropriate for most reasonably sized scripted
   applications (PHP, Python, Node.js, Ruby) and so should rarely be changed.
   Please read carefully this section to determine if you actually need to tweak
   those settings.

Application containers have memory allocated based on a default ratio that is
configurable. Two properties can be adjusted:

.. code-block:: yaml

   resources:
       base_memory: 64   # default and minimum
       memory_ratio: 256 # default, minimum: 128

``base_memory`` represents a fixed amount of memory (in megabytes) allocated to
the container. ``memory_ratio`` is an amount of memory allocated per CPU share.
For example: an ``S`` container gets a 0.5 CPU share, with the default
configuration it would get 64 + 0.5 x 256 = 192 MB of RAM.
If the same container gets its size increased to an ``XL``, it would now get
64 + 4 x 256 = 1088 MB of RAM.

Memory is expressed as two separate values to match different performance
profiles depending on your application needs.

For instance, a PHP web application needs a relatively low base memory to start
PHP-FPM, but each parallel request consumes some additional memory. It would
benefit from a low ``base_memory`` and a relatively high ``memory_ratio``. That
way, increasing the container size would increase the CPU share allowing more
concurrent requests and its memory allocation naturally scales to match.

Applications in languages that have a persistent runtime, such as Java or Go,
requires a high baseline memory to boot up but then each parallel request
consumes relatively little memory. They would benefit from a much higher
``base_memory`` value and a lower ``memory_ratio``, so that any container size
has sufficient memory to start the application.

Background workers can also generally benefit from a higher ``base_memory``
value and a lower ``memory_ratio`` if they only process one task at a time as
the increase of the CPU share will speed up processing time but will not cause
more tasks to be processed at once so the additional memory is not required.

.. _SymfonyCloud Support: https://symfony.com/cloud/support
.. _SymfonyCloud pricing page: https://symfony.com/cloud/pricing