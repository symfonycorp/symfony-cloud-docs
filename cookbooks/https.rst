HTTPS
=====

On SymfonyCloud all environments support both HTTP and HTTPS. We recommend
using HTTPS requests exclusively. Doing so provides better security, access to
certain features that web browsers only permit over HTTPS, and access to HTTP/2
connections which can greatly improve performance.

Specify only HTTPS routes in the ``routes.yaml`` file:

.. code-block:: yaml

    "https://{default}/":
        type: upstream
        upstream: "app:http"

    "https://www.{default}/":
        type: redirect
        to: "https://{default}/"

With such a configuration, any requests for an HTTP URL will
automatically be redirected to HTTPS.

.. note::

    Specifying HTTP routes results in duplicate HTTPS routes being created
    automatically, allowing the site to be served from **both** HTTP and HTTPS
    without redirects.

Let's Encrypt
-------------

Production SSL certificates are automatically provided by `Let's Encrypt
<https://letsencrypt.org/>`_. The Let's Encrypt challenge is resolved using
HTTP-01 so certificates can only be provisioned once your DNS records are
switched to target SymfonyCloud. You may alternatively :ref:`provide your own
SSL certificates <custom-ssl>` at no charge from us.

Certificate Renewal
~~~~~~~~~~~~~~~~~~~

Let's Encrypt certificate renewals are attempted on a regular basis and their
deployment happens each time the environment is deployed. If the project does
not receive regular code commits, you will need to manually issue a
re-deployment to ensure the certificate remains valid. This can be done by
issuing the following command from your **local** environment:

.. code-block:: terminal

    $ symfony redeploy

You can also automate the redeploy via a cron task:

.. code-block:: yaml

    crons:
        renewcert:
            # Force a redeploy at 10 am (UTC) on the 1st and 15th of every month.
            # for the master branch, aka production
            spec: '0 10 1,15 * *'
            cmd: |
                if [ "$SYMFONY_BRANCH" = master ]; then
                    croncape symfony redeploy --no-wait
                fi

For the command to work, `set up an authentication token <api_tokens.html>`_.

.. note::

    The certificate will not renew unless it has less than one month remaining.

.. caution::

    As the redeploy does cause a momentary pause in service, we recommend
    running during non-peak hours for your site.

.. _custom-ssl:

User provided certificate
-------------------------

User provided certificates are certificates provisioned and managed outside of
SymfonyCloud. This configuration is handy when migrating to SymfonyCloud and a
certificate has been previously provisioned, when a project requires an EV
(Extended Validation) certificate or signed by a customer authority.

Uploading a certificate to SymfonyCloud can be done either when attaching a
domain or independently:

.. code-block:: terminal

    # Attach a domain and provide a custom certificate
    $ symfony domain:attach example.com \
      --key=path/to/private.key \
      --cert=path/to/certificate.pem \
      --chain=path/to/chain.pem

    # Update a domain by providing a custom certificate
    $ symfony domain:update example.com \
      --key=path/to/private.key \
      --cert=path/to/certificate.pem \
      --chain=path/to/chain.pem

    # Redeploy to apply changes
    $ symfony redeploy

TLS Configuration
-----------------

Refine how secure TLS connections are handled via the ``tls`` route property:

.. code-block:: yaml

    https://{default}/:
        type: upstream
        upstream: app:http
        tls:
            min_version: TLSv1.2
            strict_transport_security:
                enabled: true
                include_subdomains: true
                preload: true

* ``min_version``: Sets a minimum version of TLS (possible values are
  ``TLSv1.2`` and ``TLSv1.3``). Note that if multiple routes for the same domain have
  different ``min_version`` specified, the highest specified is used for the
  whole domain.

* ``strict_transport_security``: Sets the HTTP Strict Transport Security (HSTS)
  policy; there are three sub-properties:

  * ``enabled``: Set it to ``false`` to ignore other properties.

  * ``include_subdomains``: Set it to ``true`` to instruct browsers to apply
    HSTS restrictions to all subdomains as well.

  * ``preload``: Set it to ``true`` to allow
    Google and others to add your site to a lookup reference of sites that
    should only ever be connected to over HTTPS.

If enabled, the ``Strict-Transport-Security`` header will always be sent with a
lifetime of 1 year. The `Mozilla Developer Network
<https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security>`_
has more detailed information on HSTS.

.. note::

    If multiple routes for the same domain specify different HSTS settings, the
    entire domain will still use a shared configuration. Specifically, if any
    route on the domain has ``strict_transport_security.enabled`` set to
    ``false``, HSTS will be disabled for the whole domain. Otherwise, it will be
    enabled for the whole domain if at least one such route has ``enabled`` set
    to `true`. As this logic may be tricky to configure correctly, we strongly
    recommend picking a single configuration for the whole domain and adding it
    on only a single route.

Client authenticated TLS
~~~~~~~~~~~~~~~~~~~~~~~~

In some non-browser applications (such as mobile applications, IoT devices, or
other restricted-client-list use cases), it is beneficial to restrict access to
selected devices using TLS. This process is known as client-authenticated TLS,
and functions effectively as a more secure alternative to HTTP Basic Auth.

By default, any valid SSL certificate issued by one of the common certificate
issuing authorities will be accepted. Alternatively, you can restrict access to
SSL certs issued by just those certificate authorities you specify, including a
custom authority. To do so, set ``client_authentication`` to ``require`` and
then provide a list of the certificates of the certificate authorities you wish
to allow:

.. code-block:: yaml

    tls:
        client_authentication: require
        client_certificate_authorities:
            # file path is resolved relative to the ``.symfony`` directory
            - !include
                type: string
                path: file1.key
            # inline key
            - |
                -----BEGIN CERTIFICATE-----
                ### ... ###
                -----END CERTIFICATE-----
