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

Compression is disabled for dynamic contents to mitigate the
`BREACH <https://en.wikipedia.org/wiki/BREACH>`_ and
`CRIME <https://en.wikipedia.org/wiki/CRIME>`_ security exploits.

Applications can compress their own responses if they wish but we recommend
**against** compressing responses that include any-user specific information
(such as session cookies) to prevent against those security vulnerabilities.

If your application uses Symfony 5.3 or newer, you can compress dynamic contents
because Symfony provides a built-in automatic protection against BREACH and CRIME.

.. _`gzip`: https://en.wikipedia.org/wiki/Gzip
.. _`Brotli`: https://en.wikipedia.org/wiki/Brotli
