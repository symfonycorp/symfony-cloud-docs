Security
========

Protective Blocking
-------------------

SymfonyCloud protective blocking feature protects web sites with known security
vulnerabilities by restricting their HTTP access. We maintain a signature
database of known security vulnerabilities in popular Open-Source software that
have a high impact issues and have a low complexity attack scheme.

When pushing code to Git, SymfonyCloud rejects the push if a critical
vulnerability is detected.

When new vulnerabilities are added to the database, all projects are scanned
and application's HTTP access is blocked on security risk detection.

For production websites, the block is partial and allows the website to stay
online. Depending on the nature of the vulnerability, parts of a request, such
as the query string, cookies, or any additional headers, may be removed from
HTTP requests. Some HTTP requests may be blocked entirely.

For development websites, the block is full, the error message gives you
detailed information about the vulnerability.

Unblocking is automated upon resolution of the vulnerability.

.. tip::

    Even if not recommended, you can disable the protective block feature by
    changing the ``.symfony.cloud.yaml`` configuration as follows:

    .. code-block:: yaml

        preflight:
           enabled: false
