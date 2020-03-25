Solr
====

`Apache Solr`_ is a scalable and fault-tolerant search index.

Solr search with generic schemas provided, and a custom schema is also
supported.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    mysearch:
        # supported versions: 3.6, 4.10, 6.3, 6.6, 7.6, 7.7, 8.0, 8.4
        type: solr:8.4
        disk: 1024

And wire it in ``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        solr: "mysearch:solr"


Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``SOLR`` is the upper-cased version of the key defined in the relationship
above):

* ``SOLR_HOST``: The Solr host
* ``SOLR_PORT``: The Solr port
* ``SOLR_NAME``: The Solr name
* ``SOLR_DATABASE``: An alias for ``SOLR_NAME``

Configuration
-------------

Solr 4
~~~~~~

For Solr 4, SymfonyCloud supports only a single core per server called
``collection1``.

If you want to provide your own Solr configuration, add a ``core_config`` key in
the ``.symfony/services.yaml`` file:

.. code-block:: yaml

    mysearch:
        type: solr:4.10
        disk: 1024
        configuration:
            core_config: !archive "<directory>"

The ``directory`` parameter points to a directory in the Git repository, in or
below the ``.symfony/`` folder. This directory needs to contain everything that
Solr needs to start a core. At the minimum, ``solrconfig.xml`` and ``schema.xml``.

Solr 6 and later
~~~~~~~~~~~~~~~~

For Solr 6 and later, SymfonyCloud supports multiple cores via different
endpoints. Cores and endpoints are defined separately, with endpoints
referencing cores. Each core may have its own configuration or share a
configuration:

.. code-block:: yaml

    solrsearch:
        type: solr:6.6
        disk: 1024
        configuration:
            cores:
                mainindex:
                    conf_dir: !archive "core1-conf"
                extraindex:
                    conf_dir: !archive "core2-conf"
            endpoints:
                main:
                    core: mainindex
                extra:
                    core: extraindex

Each endpoint is available in the relationships definition in
``.symfony.cloud.yaml``:

.. code-block:: yaml

    relationships:
        solr1: 'solrsearch:main'
        solr2: 'solrsearch:extra'

Configsets
..........

For even more customizability, it's also possible to define Solr configsets. For
example, the following snippet would define one configset, which would be used
by all cores. Specific details can then be overridden by individual cores using
``core_properties``, which is equivalent to the Solr ``core.properties`` file.

.. code-block:: yaml

    solrsearch:
        type: solr:8.4
        disk: 1024
        configuration:
            configsets:
                mainconfig: !archive "configsets/solr6"
            cores:
                english_index:
                    core_properties: |
                        configSet=mainconfig
                        schema=english/schema.xml
                arabic_index:
                    core_properties: |
                        configSet=mainconfig
                        schema=arabic/schema.xml
            endpoints:
                english:
                    core: english_index
                arabic:
                    core: arabic_index

.. caution::

    Note that not all ``core.properties`` features make sense to specify in the
    ``core_properties`` property. Some keys, such as ``name`` and ``dataDir``,
    are not supported, and may result in a solrconfig that fails to work as
    intended, or at all.

Default configuration
.....................

If no configuration is specified, the default configuration is equivalent to:

.. code-block:: yaml

    solrsearch:
        type: solr:8.4
        configuration:
            cores:
                collection1:
                    conf_dir: {}  # This will pick up the default Drupal 8 configuration
            endpoints:
                solr:
                    core: collection1

Limitations
...........

The recommended maximum size for configuration directories (zipped) is 2MB.
These need to be monitored to ensure they don't grow beyond that. If the zipped
configuration directories grow beyond this, performance will decline and deploys
will become longer. The directory archives will be compressed and string
encoded. You could use this bash pipeline inside the directory to get an idea
of the archive size:

.. code-block:: terminal

    $ echo $(($(tar czf - . | base64 | wc -c )/(1024*1024))) Megabytes

The ``!archive "<directory">`` is a collection of configuration data, like a
data dictionary, e.g. small collections of key/ value sets. The best way to
keep the size small is to restrict the directory context to plain
configurations. Including binary data like plugin .jars will inflate the
archive size, and is not recommended.

.. _`Apache Solr`: https://en.wikipedia.org/wiki/Apache_Solr
