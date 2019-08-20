Multiple Applications
=====================

A SymfonyCloud project can contain several applications (i.e. a RESTful web
service and a front-end, or a main web site and a blog). Each application runs
in its own container and is configured by one ``.symfony.cloud.yaml`` file at
its root directory:

.. code-block:: terminal

    $ tree
    oss-websites
    ├── .git
    ├── .symfony
    │   ├── routes.yaml
    │   └── services.yaml
    ├── awards.example.com
    │   └── .symfony.cloud.yaml
    ├── pimple.example.com
    │   └── .symfony.cloud.yaml
    ├── sismo.example.com
    │   └── .symfony.cloud.yaml
    ├── swiftmailer.example.com
    │   └── .symfony.cloud.yaml
    └── twig.example.com
        └── .symfony.cloud.yaml

.. note::

    A SymfonyCloud project is limited to only a single Git repository: all the
    applications are contained into one repository.

Routes
------

:doc:`Routes </routes>` are defined at the project level in the
``.symfony/routes.yaml`` file and determine how to route requests to different
applications:

.. code-block:: yaml

    # .symfony/routes.yaml
    "https://twig.example.com/":
        type: upstream
        upstream: "twig:http"

    "https://awards.example.com/":
        type: upstream
        upstream: "awards:http"

    # ...

The ``upstream`` property is used to specify which application receives requests
by using the :ref:`name <application-name>` of the application as defined in
its ``.symfony.cloud.yaml`` and followed by ``:http``.

Services
--------

:doc:`Services </services/intro>` are defined at the project level in the
``.symfony/services.yaml`` file and can be shared by several applications. The
:ref:`relationships <relationships>` section of the ``symfony.cloud.yaml`` files
specifies which service are exposed to which application:

.. code-block:: yaml

    # .symfony/services.yaml
    awardsdb:
        type: postgresql:9.6
        disk: 1024

.. code-block:: yaml

    # awards.example.com/.symfony.cloud.yaml

    # ...
    relationships:
        database: "awardsdb:postgresql"
    # ...

Micro-services
--------------

In a micro-services approach, a project can be composed of a "main" application
using :doc:`HTTP relationships </services/http>` to other more specialized
applications using them as regular services:

.. code-block:: yaml

    # awards.example.com/.symfony.cloud.yaml
    # ...
    relationships:
        database: "awardsdb:postgresql"
        server: "some-app:http"

In this configuration, ``some-app`` refers to another :ref:`application name
<application-name>` .