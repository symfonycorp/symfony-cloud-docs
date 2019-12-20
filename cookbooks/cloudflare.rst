Cloudflare
==========

.. note::

    Setting up Cloudflare is optional and not a SymfonyCloud requirement.

CNAME Flattening
----------------

One of the main features that a DNS provider needs to have in order to work well
with SymfonyCloud is what is usually known as "CNAME Flattening". This solves
the problem of being able to point your APEX (the "root domain": example.com) to
a domain name using a CNAME record rather than an IP address (A record). `This
post from Cloudflare
<https://blog.cloudflare.com/introducing-CNAME-flattening-rfc-compliant-cnames-at-a-domains-root/>`_
explains this process in details.

In order to correctly point the DNS records to a SymfonyCloud project, you need
the master environment CNAME; get it via ``symfony domain:cname``.

HTTP Proxy
----------

In addition to their DNS service, Cloudflare also provides an HTTP proxy with
CDN, caching, TLS termination, and DDoS protection for free.

Full SSL
~~~~~~~~

When enabling the HTTP proxy feature, we **highly** recommend using
the "Full SSL" option from Cloudflare and not the "Flexible SSL".

.. code-block:: text

    # Cloudflare's Flexible SSL option
               HTTPS                       HTTP
    User <---------------> Cloudflare <-------------> SymfonyCloud

    # Cloudflare's Full SSL option
               HTTPS                       HTTPS
    User <---------------> Cloudflare <-------------> SymfonyCloud

It means that the traffic between Cloudflare's servers and your SymfonyCloud
project is encrypted. Without this configuration, the communication between
Cloudflare and SymfonyCloud is subject to `Man-in-the-middle
<https://en.wikipedia.org/wiki/Man-in-the-middle_attack>`_ attacks. Not using
"Full SSL" can also cause infinite redirect loops as HTTPS traffic will be seen
as HTTP only to your project.

Full (strict) SSL
~~~~~~~~~~~~~~~~~

For enhanced security, we recommend you to enable "Full (strict) SSL" option on
your Cloudflare admin. This option ensures your website has a valid certificate
installed, preventing `Man-in-the-middle
<https://en.wikipedia.org/wiki/Man-in-the-middle_attack>`_ attacks between
Cloudflare and SymfonyCloud.

First, `create and download a Cloudflare Origin Certificate
<https://support.cloudflare.com/hc/en-us/articles/115000479507-Managing-Cloudflare-Origin-CA-certificates>`_.
Then upload it to SymfonyCloud:

.. code-block:: terminal

    $ symfony domain:update --cert=certificate-file.pem --key=private-key.key example.com

Finally, you can turn on the "Full (strict) SSL" option.

Authenticated Origin Pulls
~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Cloudflare sits between end-users and a project on SymfonyCloud, but
SymfonyCloud stills allows direct requests to the project (by-passing
Cloudflare's features and protection). This is why we recommend you to enable
*Authenticated Origin Pulls*.

With this features enabled, every requests reaching the application is
authenticated using `TLS Client-authentication
<https://en.wikipedia.org/wiki/Transport_Layer_Security#Client-authenticated_TLS_handshake>`_,
where the client certificate is signed by a known authority which allows to
restrict the requests only to Cloudflare edge servers.

To enabled this setup, follow these steps:

1. Turn on "Authenticated Origin Pulls" in the "Crypto" section of the
   Cloudflare backend;
2. Make sure the project has the "HTTP Proxy" feature enabled (orange cloud) in
   the "DNS" section of the Cloudflare backend;
2. Download the Cloudflare Authenticated Origin Pulls CA certificate
   `origin-pull-ca.pem
   <https://support.cloudflare.com/hc/en-us/article_attachments/360044928032/origin-pull-ca.pem>`_
   and add it to the ``.symfony`` directory of the project to protect;
3. Adapt the ``.symfony/routes.yaml`` file as follows:

   .. code-block:: yaml

       "https://{all}/":
           type: upstream
           upstream: "app:http"
           tls:
               client_authentication: "require"
               client_certificate_authorities:
                   - !include
                       type: string
                       path: origin-pull-ca.pem
4. ``git add .symfony/origin-pull-ca.pem .symfony/routes.yaml && git commit``;
5. ``symfony deploy``
