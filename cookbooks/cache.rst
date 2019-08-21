HTTP Cache
==========

HTTP Cache is usually managed at an additional layer on top of the web server
(Varnish, :doc:`Cloudflare </cookbooks/cloudflare>`, Cloudfront...).
SymfonyCloud also allows you to enable HTTP caching right at the web server
level.

When the cache is enabled, a fresh response is first looked for in the cache;
if not available, the request is routed to the application.

Cache can be configured in the ``.symfony/routes.yaml`` file like below:

.. code-block:: yaml

    https://{default}/:
        type: upstream
        upstream: app:http
        cache:
            enabled: true
            headers: ['Accept', 'Accept-Language', 'X-Language-Locale']
            cookies: ['*']
            default_ttl: 10

.. caution::

    By default, responses issued with Symfony are not cacheable (``no-cache,
    private``). To make use of SymfonyCloud cache, set a shared max-age on
    responses::

        // src/Controller/DefaultController.php
        use Symfony\Component\HttpFoundation\Response;
        // ...

        public function index()
        {
            $response = $this->render('index.html.twig');

            // sets the shared max age - which also marks the response as public
            $response->setSharedMaxAge(3600);

            return $response;
        }

.. note::

    For more information about HTTP cache with Symfony, you can read its
    `official documentation <https://symfony.com/doc/current/http_cache.html>`_.

Cache per Route
---------------

If you need fine-grained caching, set caching rules separately for each route:

.. code-block:: yaml

    https://{default}/:
        type: upstream
        upstream: php:http
        cache:
            enabled: true

    https://{default}/foo:
        type: upstream
        upstream: php:http
        cache:
            enabled: false

    https://{default}/foo/bar:
        type: upstream
        upstream: php:http
        cache:
            enabled: true

With this configuration, the following routes are cached:

-  ``http://{default}/``
-  ``http://{default}/foo/bar``
-  ``http://{default}/foo/bar/baz``

And the following routes are **not** cached:

-  ``http://{default}/foo``
-  ``http://{default}/foo/baz``

.. note::

    To use WebSocket, ``cache`` must be **disabled** because WebSocket is
    incompatible with buffering, which is a requirement of our caching
    technology.

Cache Behavior
--------------

Cache is only applied to ``GET`` and ``HEAD`` requests.
Responses with the ``Cache-Control`` header set to ``Private``, ``No-Cache``,
or ``No-Store`` are not cached. Responses with the ``Set-Cookie`` header are
also not cached.

The web server does not honor the ``Pragma`` request header. Conditional
requests using ``If-Modified-Since`` and ``If-None-Match`` are both supported.

Cache Revalidation
------------------

When the cache is expired, the web server will route the request to the
application adding an ``If-Modified-Since`` header.

Cache Key
---------

When determining if a response is cached, in addition to the request URI,
SymfonyCloud builds a **cache key**. The cache key is built using the values of
the request Headers and Cookies and used to differentiate several requests to
the same URI.

The configuration lets you change which headers and cookies are used to build
the cache key. Default values for these parameters are:

.. code-block:: yaml

    cache:
        enabled: true
        cookies: ['*']
        headers: ['Accept', 'Accept-Language']

The ``Vary`` header in the response is also respected: multiple copies are
stored in the cache according to the value of this header.
For example, by adding ``Vary: X-Forwarded-Proto`` to the response header,
responses contents are cached but differentiated based on the request protocol
(i.e. HTTP or HTTPS).

Cache Configuration Reference
-----------------------------

.. code-block:: yaml

    https://{default}/:
        type: upstream
        upstream: php:http
        cache:
            enabled: true
            headers: ['Accept']
            cookies: ['/^__utm/', '_ga', '_gat', '_gid']
            default_ttl: 10

    https://{default}/admin:
        type: upstream
        upstream: php:http
        cache:
            enabled: false

``enabled``
~~~~~~~~~~~

* ``true`` : enable the cache for this route (default)
* ``false`` : disable the cache for this route

``headers``
~~~~~~~~~~~

The ``headers`` parameter defines which values the cache key depends on.

For example, the following cache configuration caches a different response for
each value of the ``Accept`` HTTP request header:

.. code-block:: yaml

    cache:
        enabled: true
        headers: ['Accept']

The default value is ``['Accept', 'Accept-Language']``

.. note::

    The following request headers cannot be used as cache key:
    ``Accept-Encoding``, ``Connection``, ``Proxy-Authorization``, ``TE``,
    ``Upgrade``.

``cookies``
~~~~~~~~~~~

The ``cookies`` parameter defines which values the cache key must depend on.

For example, with the following cache configuration; the cache key will depend
on the value of the ``foo`` cookie in the request; other cookies will be
ignored:

.. code-block:: yaml

    cache:
        enabled: true
        cookies: ['foo']

A special case is the ``['*']`` value: makes any request with a cookie bypass
the cache. This is the default value.

``[]`` ignores all cookies.

.. note::

    Regular expressions can be used for cookie names. Entries that begin and
    end with a ``/`` are interpreted as PCRE regexes and evaluated against the
    cookie name.

``default_ttl``
~~~~~~~~~~~~~~~

Defines the default cache duration, in seconds. The cache duration is decided
based on the ``Cache-Control`` response header value. If no ``Cache-Control``
header is in the response, then the value of the ``default_ttl`` parameter is
used. The default value is ``0``.

.. _cache-ssi:

SSI
---

SSI allows cache strategies to differ between the whole page and some
more-dynamic parts of an HTML page thanks to directives similar to HTML tags:

.. code-block:: html

    <!DOCTYPE html>
    <html>
        <body>
            <!-- ... some content -->

            <!-- Embed the content of another page here -->
            <!--#include virtual="http://..." -->

            <!-- ... more content -->
        </body>
    </html>

Enable SSI in ``.symfony/routes.yaml``:

.. code-block:: yaml

    https://{default}/:
        type: upstream
        upstream: app:http
        ssi:
            enabled: true


And the SSI and fragments support in the Symfony configuration:

.. code-block:: yaml

    # config/packages/framework.yaml
    framework:
        # ...
        ssi: true
        fragments: true

Now, suppose you have a page that is relatively static, except for the current
date included at the top of the content. With SSI, you can cache the date
independently of the rest of the page::

    // src/Controller/DefaultController.php

    use Symfony\Component\HttpFoundation\Response;
    // ...

    public function index()
    {
        $response = $this->render('index.html.twig');

        // sets the shared max age - which also marks the response as public
        $response->setSharedMaxAge(3600);

        return $response;
    }

    public function date()
    {
        $response = $this->render('date.html.twig');

        return $response;
    }

.. code-block:: twig

    {# templates/index.html.twig #}

    {# you can use a controller reference #}
    {{ render_ssi(controller('App\\Controller\\DefaultController::date')) }}

    {# ... or a URL #}
    {{ render_ssi(url('date')) }}

With SSI, the full page cache is valid for 600 seconds, but the date component
is not cached.

To dig further, you can read about SSI on `Wikipedia
<https://en.wikipedia.org/wiki/Server_Side_Includes>`_.
