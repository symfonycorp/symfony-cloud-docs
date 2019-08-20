Compression
===========

Static files
------------

SymfonyCloud supports response compression using either `gzip`_ or `Brotli`_ if
the following conditions are met:

* The request supports gzip or brotli;
* The file is served directly from disk, not passed through your application;
* The request is served with a cache expiration in the future.

If an incoming request meets the previous conditions and the file type is one
of: ``html``, ``javascript``, ``json``, ``pdf``, ``postscript``, ``svg``,
``css``, ``csv``, plain text, or ``XML``; the response is automatically
compressed with the appropriate algorithm.

Alternatively, if a file with a ``.gz`` or ``.br`` extension exists, this file
will be served instead of being compressed on the fly. This approach supports
any file type and offers some CPU optimization.

Dynamic content
---------------

Compression is disabled for dynamic content due to a `well known security issue
<https://en.wikipedia.org/wiki/BREACH_%28security_exploit%29>`_. Applications
can compress their own responses but they should not do it when responses
include any-user specific information (such as session cookies) to prevent
against the previously linked vulnerability. As a precautionary measure, we
recommend **against** compressing dynamic content.

.. _`gzip`: https://en.wikipedia.org/wiki/Gzip
.. _`Brotli`: https://en.wikipedia.org/wiki/Brotli
