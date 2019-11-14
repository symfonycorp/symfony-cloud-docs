Private Dependencies
====================

Some projects use private dependencies requiring authentication in order for
packages to be downloaded. While committing these credentials in the Git
repository is technically possible, this is considered as a bad practice.

You have two options:

#. `Composer HTTP Authentication`_
#. `SSH Deploy Key`_

Composer HTTP Authentication
----------------------------

Composer allows you to define configuration for its `builtin authentication
mechanism <https://getcomposer.org/doc/articles/http-basic-authentication.md>`_
as an environment variable named ``COMPOSER_AUTH``. The content of the variable
should be a JSON formatted object containing some specific key objects (see
`Composer Auth specifications
<https://getcomposer.org/doc/03-cli.md#composer-auth>`_).

Set the Composer authentication by adding a project level variable called
``COMPOSER_AUTH`` as JSON and available only during build time:

.. code-block:: terminal

    $ symfony var:set --sensitive --hide-at-runtime \
    COMPOSER_AUTH='{"github-oauth": {"github.com": "oauthtoken"}, "http-basic": {"repo.example.org": {"username": "your-username", "password": "your-password"}}}'

Once set, Composer will be able to authenticate and download dependencies from
your authenticated repositories.

SSH Deploy Key
--------------

Typically, private dependencies are hosted in private Git repositories which
require SSH authentication. Fortunately, most private Composer tools (including
Satis, Toran Proxy, and `Private Packagist <https://packagist.com/>`_) mirror
tagged releases of dependencies and serve them directly rather than hitting the
Git repository. Therefore, as long as your dependencies specify tagged releases,
there should be no need to authenticate against a remote Git repository and
there should be no authentication issues.

But shall you need it, this use case is also covered: SymfonyCloud generates one
private SSH key per project and provides you with the public key allowing you to
authorize it for Git repositories or specific machine accounts. This key is
called a "deploy" key as its usage is reserved to deployment. GitHub provides
`documentation
<https://developer.github.com/v3/guides/managing-deploy-keys/#machine-users>`_
about this feature. You can safely restrict its access to read-only operations.

You can obtain this public key using the CLI:

.. code-block:: terminal

    $ symfony project:deploy-key
