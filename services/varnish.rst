Varnish
=======

`Varnish`_ is a popular HTTP proxy server, often used for caching. It is usually
not needed on SymfonyCloud, as each project's router already provides an HTTP
cache and most more advanced use cases will use a CDN instead, both of
which make Varnish redundant.

However, it is possible to configure a Varnish instance as part of an
application if Varnish-specific functionality is needed.

Supported versions
------------------

* 5.2
* 6.0

How it works
------------

All incoming requests still go through the environment's router first. When
using Varnish, a Varnish service sits between the router and the application
server or servers.

Configuration
-------------

Add a Varnish service
~~~~~~~~~~~~~~~~~~~~~

Add the following to your ``.symfony/services.yaml`` file:

.. code-block:: yaml

    varnish:
        type: varnish:6.0
        relationships:
            application: 'app:http'
        configuration:
            vcl: !include
                type: string
                path: config.vcl

In the ``relationships`` block, define a relationship (``application``) to the
application container (``app``) using the ``http`` endpoint. That allows
Varnish to talk to the application container.

The configuration block is required, and must reference a VCL file (here
``config.vcl``). The file name is relative to the ``.symfony`` directory.

Create a VCL template file
~~~~~~~~~~~~~~~~~~~~~~~~~~

The VCL file you provide has three specific requirements over and above the VCL
syntax itself.

1. You **MUST NOT** define a ``vcl_init()`` function. SymfonyCloud will
   auto-generate that function based on the relationships you define. In
   particular, it will define a "backend" for each relationship defined in
   ``services.yaml``, named the same as the relationship.
#. You **MUST NOT** include the preamble at the beginning of the file,
   specifying the VCL version. That will be auto-generated as well. You **CAN**
   add imports, but not ``std`` and ``directors``.
#. You **MUST** specify the backend to use in ``vcl_recv()``. If you have a
   single app container/relationship/backend, it's just a single line. If you
   want to split requests to different relationships/backends based on some rule
   then the logic for doing so should be incorporated into the ``vcl_recv()``
   function.


The absolute bare minimum VCL file is:

.. code-block:: yaml

    sub vcl_recv {
        set req.backend_hint = application.backend();
    }

Where `application` is the name of the relationship defined in
``services.yaml`` (of the relationship was named differently, use that name
instead).

If you have :doc:`multiple applications</cookbooks/multi-apps>` behind the same
Varnish instance then you will need to include logic to determine to which
application a request is forwarded:

.. code-block:: yaml

    varnish:
        type: varnish:6.0
        relationships:
            blog: 'blog:http'
            main: 'app:http'
        configuration:
            vcl: !include
                type: string
                path: config.vcl

.. code-block:: vcl

    # config.vcl
    sub vcl_recv {
        if (req.url ~ "^/blog/") {
            set req.backend_hint = blog.backend();
        } else {
            set req.backend_hint = main.backend();
        }
    }

This configuration will direct all requests to a URL beginning with a `/blog/`
path to the application on the relationship ``blog``, and all other requests to
the application on the relationship ``main``.

Besides that, the VCL file, including the ``vcl_recv()`` function, can be
arbitrarily complex to suit the needs of the project. That includes additional
``include`` directives if appropriate. See the `Varnish
documentation <https://varnish-cache.org/docs/index.html>`_ for more details
on the functionality offered by Varnish.

.. note::

    A misconfigured VCL file can result in incorrect, often mysterious and
    confusing behavior. SymfonyCloud does not provide support for VCL
    configuration options beyond the basic connection logic documented here.

Route incoming requests to Varnish
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To enable Varnish now, edit the ``.symfony/routes.yaml`` file to point to the
Varnish service you just created. You also need to disable the router cache as
it is now entirely redundant with Varnish:

.. code-block:: yaml

    "https://{default}/":
        type: upstream
        upstream: "varnish:http"
        cache:
            enabled: false

That will map all incoming requests to the Varnish service rather than the
application. Varnish will then, based on the VCL file, forward requests to the
application as appropriate.

Stats endpoint
--------------

The Varnish service also offers an ``http+stats`` endpoint, which provides
access to some Varnish analysis and debugging tools. To access it, from a
**dedicated** (see note below) app container, add the following to
``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        varnishstats: "varnish:http+stats"

You can then access the ``varnishstats`` relationship over HTTP at the
following paths to get diagnostic information:

* ``/``: returns the error if generating the VCL failed with an error
* ``/config``: returns the generated VCL
* ``/stats``: returns the output of ``varnishstat``
* ``/logs``: returns a streaming response of ``varnishlog``

.. caution::

    Because SymfonyCloud does not support circular relationships between
    services or applications at this time, you cannot add a relationship that
    points to the Varnish service in the ``.symfony.cloud.yaml`` of an
    application that Varnish is forwarding to. If you do so one of the
    relationships will be skipped and the connection will not work. **The
    relationship needs to be defined by a separate application container.** Read
    our :doc:`Multiple applications </cookbooks/multi-apps>` cookbook for more
    information on how create an additional application container.

.. _`Varnish`: https://en.wikipedia.org/wiki/Varnish_(software)
