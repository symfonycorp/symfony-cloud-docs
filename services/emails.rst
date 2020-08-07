Emails
======

By default, only the **production** environment (**master**) can send emails. Outgoing emails need to be explicitly enabled for all other environments:

.. code-block:: terminal

    $ symfony env:setting:set smtp on --env=some-env

.. tip::

    Check if outgoing emails are enabled for the current environment by using
    the ``env:settings`` command:

    .. code-block:: terminal

        $ symfony env:settings

Environment Variables
---------------------

The configuration is exposed via the following environment variables:

* ``MAILER_ENABLED``: 1 if outgoing emails are enabled, 0 otherwise
* ``MAILER_DSN``/``MAILER_URL``: The Symfony-compatible mailer URL
* ``MAILER_HOST``: The SMTP server host
* ``MAILER_PORT``: The SMTP server port
* ``MAILER_TRANSPORT``: The SMTP transport mode (``smtp``)
* ``MAILER_AUTH_MODE``: The SMTP auth mode (``plain``)
* ``MAILER_USER``: The SMTP server user
* ``MAILER_PASSWORD``: The SMTP server password

Usage
-----

On Symfony projects using Flex, install Symfony Mailer:

.. code-block:: terminal

    $ composer require mailer

No further configuration is needed as the default configuration knows how to
use the value of ``MAILER_DSN`` automatically.

.. tip::

   ``MAILER_URL`` is also provided for backward-compatiblity with
   SwiftMailerBundle recipes


For other applications, use ``MAILER_DSN`` with Symfony Mailer or use the
``MAILER_*`` environment variables to configure your framework of choice.

.. caution::

    Avoid using the PHP built-in ``mail()`` function to send emails. Even if the
    PHP runtime is configured to send email via the assigned SendGrid
    sub-account, ``mail()`` is known to be difficult to secure depending on the
    supplied parameters.

Behind the Scenes
-----------------

Emails from SymfonyCloud are sent via a SendGrid-based SMTP proxy. Each
SymfonyCloud project is provisioned as a SendGrid sub-account. These SendGrid
sub-accounts are capped at 12k emails per month.

We do not guarantee the deliverability of emails, and we do not support
white-labeling them. Our SMTP proxy is intended as a zero-configuration, best
effort service. If needed, you can instead use your own SMTP server or email
delivery service provider. In that case, please bear in mind that TCP port 25 is
blocked for security reasons; use TCP port 465 or 587 instead.

.. note::

    You may follow the `SPF setup guidelines on SendGrid
    <https://sendgrid.com/docs/Glossary/spf.html>`_ to improve email
    deliverability with our SMTP proxy.
