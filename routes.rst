.symfony/routes.yaml
====================

SymfonyCloud allows you to define how incoming requests are routed to project's
applications.

*Routes* are defined in the ``.symfony/routes.yaml`` file and describe how an
incoming HTTP request must be processed by SymfonyCloud.

The ``.symfony/routes.yaml`` YAML file is composed of a list of routes defined
by `route patterns`_ and their associated configuration. Here is an example of a
basic ``.symfony/routes.yaml`` file:

.. code-block:: yaml

    "http://{default}/":
        type: redirect
        # Redirect from HTTP to HTTPS
        to: "https://{default}/"

    "https://{default}/":
        type: upstream
        # Route requests to the "app" application
        # ("app" is the name defined in .symfony.cloud.yaml)
        upstream: "app:http"

This configuration makes SymfonyCloud redirect all HTTP-only requests to HTTPS
and forward HTTPS requests to the "app" application. For now, let's say that
``{{default}}`` makes SymfonyCloud match the project's default domain for
incoming requests.

Route Patterns
--------------

The configuration can contain several route definitions. The identifier of each
route is used as a **pattern** that should match the beginning of the URL for
the route to be used.

If more than one route match an incoming request, the most specific definition
(the one with the longest size) is used:

.. code-block:: yaml

    "http://{default}/":
        type: upstream
        upstream: "app:http"

    "http://{default}/login":
        type: redirect
        to: "https://{default}/login"

    "https://{default}/":
        type: upstream
        upstream: "app:http"

This example makes the website available over both HTTP and HTTPS except for
HTTP requests to ``/login`` that are redirected to HTTPS.

Two special **placeholders** are available to use in route patterns:

* ``{default}`` will match the default domain of a project.
* ``{all}`` will match any domain (think about it like ``.*`` for Regexes)

Placeholders can be used in any part of the host name, for example
``https://{default}/`` or ``https://www.{default}/``.

Any number of domains can be configured on a project but only one of them
is set as the *default* domain using ``symfony domain:default example.com``. If
a project has no domains configured, SymfonyCloud provides you with a default
development one.

.. note::

    When using the ``{all}`` placeholder, creating routes to redirect HTTP
    requests to HTTPS is optional: if you create a single HTTPS route,
    SymfonyCloud automatically creates a HTTP one redirecting to the HTTPS
    version.

Multiple Applications
~~~~~~~~~~~~~~~~~~~~~

SymfonyCloud supports running multiple applications per project; in this
configuration the ``.symfony/routes.yaml`` file defines how to route requests to
different applications:

.. code-block:: yaml

    "http://{default}/":
        type: redirect
        to: "https://{default}/"

    "https://{default}/":
        type: upstream
        # Route requests to the "app" application
        # "app" is the name defined in .symfony.cloud.yaml
        upstream: "app:http"

    "https://{default}/blog":
        type: upstream
        # Route requests to the "blog" application
        # "blog" is the name defined in another .symfony.cloud.yaml
        upstream: "blog:http"

Wildcard Routes
~~~~~~~~~~~~~~~

SymfonyCloud supports wildcard routes so that multiple subdomains can be defined
as any other routes; routed to an application or redirected to another one.

To create a wildcard route, prefix the route with a star (``*``). For example
``*.example.com`` matches any HTTP requests to ``www.example.com``,
``blog.example.com``, or ``us.example.com`` once the wildcard domain
(``*.example.com``) is added to the project.

.. caution::

    Please note that when using wildcard routes, SymfonyCloud can not
    automatically provision SSL certificates on your behalf. As a consequence,
    to protect traffic with SSL, generate or buy a wildcard SSL certificate and
    configure the project domains to use it.

Absolute Routes
~~~~~~~~~~~~~~~

A route can also be defined as an absolute URL. For example:

.. code-block:: yaml

    "https://{default}/":
        type: upstream
        upstream: "app:http"

    "https://blog.example.com/":
        type: upstream
        upstream: "blog:http"

If a request matches several routes, the ones defined with absolute URLs take
precedence over the generic (ie. using placeholders) ones.

Development Environments
~~~~~~~~~~~~~~~~~~~~~~~~

For each development environment, a unique subdomain is generated using
information such as the project and environment IDs. This subdomain matches
``{default}`` and ``{all}`` in your routes.

For routes using absolute hostnames, SymfonyCloud uses those as subdomains of
the domain generated specifically for this environment.

Get the generated URLs with ``symfony urls``.

Route Configuration
-------------------

Each route has the following properties available for configuration:

* ``type`` defines how requests are be processed. Possible values are:

  * ``upstream``: requests are routed to an application.

  * ``redirect``: requests are redirected to another destination.

* ``upstream`` (required when ``type`` equals to ``upstream``) specifies the
  name of the application; as defined in ``.symfony.cloud.yaml`` and followed by
  ``:http``; used to serve requests.

* ``to`` (required when ``type`` equals to ``redirect``) defines to which URL
  incoming requests should be redirected to.

* ``cache`` controls :doc:`caching behavior of the route </cookbooks/cache>`.

* ``ssi`` enables or disables Server Side Includes support; see
  :ref:`SSI <cache-ssi>` for more information.

* ``redirects`` controls :doc:`partial redirect rules </cookbooks/redirects>`
  associated with the route.