robots.txt
==========

By default, SymfonyCloud includes an additional ``X-Robots-Tag`` header on all
non-production environments:

.. code-block:: text

    X-Robots-Tag: noindex, nofollow

That tells search engines to not index sites on non-production environments
entirely nor traverse links from those sites, even if they are publicly
visible. That keeps non-production sites out of search engine indexes that
would dilute the SEO of the production site. To disable that feature for a
non-production environment, use the Symfony CLI command below:

.. code-block:: terminal

    $ symfony setting:set robots off

On a production instance (the master branch, after a domain has been assigned)
the search-blocker is disabled and your application can serve a ``robots.txt``
file as normal.
