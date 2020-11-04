Elasticsearch
=============

`Elasticsearch`_ is a distributed RESTful search engine built for the cloud.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    # .symfony/services.yaml
    mysearch:
        # supported versions: 6.5 and 7.2
        # 0.90, 1.4, 1.7, 2.4, 5.2 and 5.4 are also available but not
        # maintained upstream
        type: elasticsearch:7.2
        disk: 1024

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    # .symfony.cloud.yaml
    relationships:
        elasticsearch: "mysearch:elasticsearch"

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``ELASTICSEARCH`` is the upper-cased version of the key defined in the
relationship above):

* ``ELASTICSEARCH_URL``: The full URL of the Elasticsearch service
* ``ELASTICSEARCH_HOST``: The Elasticsearch host
* ``ELASTICSEARCH_PORT``: The Elasticsearch port
* ``ELASTICSEARCH_SCHEME``: The Elasticsearch protocol scheme (``http`` or ``https``)

.. caution::

    When you create an index on Elasticsearch, you should not specify
    ``number_of_shards`` and ``number_of_replicas`` settings in your
    Elasticsearch API call. These values will be set automatically based on
    available resources.

Plugins
-------

The Elasticsearch 2.4 and later services offer a number of plugins. To enable
them, list them under the ``configuration.plugins`` key in your
``.symfony/services.yaml`` file:

.. code-block:: yaml

    mysearch:
        type: "elasticsearch:7.2"
        disk: 1024
        configuration:
            plugins:
                - analysis-icu
                - lang-python

Available plugins
-----------------

This is the complete list of official Elasticsearch plugins that can be enabled:

===================== ========================================================================================= ===  === === === ===
Plugin                Description                                                                               2.4  5.2 5.4 6.5 7.2
===================== ========================================================================================= ===  === === === ===
analysis-icu          Support ICU Unicode text analysis                                                         *    *   *   *   *
analysis-nori         Integrates Lucene nori analysis module into Elasticsearch                                              *   *
analysis-kuromoji     Japanese language support                                                                 *    *   *   *   *
analysis-phonetic     Phonetic analysis                                                                         *    *   *   *   *
analysis-smartcn      Smart Chinese Analysis Plugins                                                            *    *   *   *   *
analysis-stempel      Stempel Polish Analysis Plugin                                                            *    *   *   *   *
analysis-ukrainian    Ukrainian language support                                                                     *   *   *   *
cloud-aws             AWS Cloud plugin, allows storing indices on AWS S3                                        *
delete-by-query       Support for deleting documents matching a given query                                     *
discovery-multicast   Ability to form a cluster using TCP/IP multicast messages                                 *
ingest-attachment     Extract file attachments in common formats (such as PPT, XLS, and PDF)                         *   *   *   *
ingest-user-agent     Extracts details from the user agent string a browser sends with its web requests              *   *   *
lang-javascript       Javascript language plugin, allows the use of Javascript in Elasticsearch scripts              *   *
lang-python           Python language plugin, allows the use of Python in Elasticsearch scripts                 *    *   *
mapper-annotated-text Adds support for text fields with markup used to inject annotation tokens into the index               *   *
mapper-attachments    Mapper attachments plugin for indexing common file types                                  *    *   *
mapper-murmur3        Murmur3 mapper plugin for computing hashes at index-time                                  *    *   *   *   *
mapper-size           Size mapper plugin, enables the `_size` meta field                                        *    *   *   *   *
repository-s3         Support for using S3 as a repository for Snapshot/Restore                                      *   *   *   *
===================== ========================================================================================= ===  === === === ===

If there is a publicly available plugin you need that is not listed here,
please contact our support team.

Upgrading
---------

The Elasticsearch data format sometimes changes between versions in
incompatible ways. Elasticsearch does not include a data upgrade mechanism as
it is expected that all indexes can be regenerated from stable data if needed.
To upgrade (or downgrade) Elasticsearch you will need to use a new service from
scratch.

There are two ways of doing that.

Destructive
~~~~~~~~~~~

In your ``.symfony/services.yaml`` file, change the version of your
Elasticsearch service **and** its name. Then update the name in the
``.symfony.cloud.yaml`` relationships block.

When you deploy this change to SymfonyCloud, the old service will be deleted
and a new one with the new name is created, with no data. You can then have
your application reindex data as appropriate.

This approach is simple but has the downside of temporarily having an empty
Elasticsearch instance, which your application may or may not handle
gracefully, and needing to rebuild your index afterward. Depending on the size
of your data that could take a while.

Transitional
~~~~~~~~~~~~

For a transitional approach you will temporarily have two Elasticsearch
services. Add a second Elasticsearch service with the new version and a new
name and give it a new relationship in ``.symfony.cloud.yaml``. You can
optionally run in that configuration for a while to allow your application to
populate indexes in the new service as well.

Once you're ready to cut over, remove the old Elasticsearch service and
relationship. You may optionally have the new Elasticsearch service use the old
relationship name if that's easier for your application to handle. Your
application is now using the new Elasticsearch service.

This approach has the benefit of never being without a working Elasticsearch
instance. On the downside, it requires two running Elasticsearch servers
temporarily, each of which will consume resources and need adequate disk space.
Depending on the size of your data that may be a lot of disk space.

.. _`Elasticsearch`: https://en.wikipedia.org/wiki/Elasticsearch
