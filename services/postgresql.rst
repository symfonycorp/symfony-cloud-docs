PostgreSQL
==========

`PostgreSQL`_ is a high-performance, standards-compliant relational SQL database.

To use it in your application, add it to ``.symfony/services.yaml``:

.. code-block:: yaml

    mydatabase:
        # supported versions: 9.6, 10, 11, 12, 13
        # 9.3 is also available but not maintained upstream
        type: postgresql:13
        disk: 1024

.. caution::

   Upgrading to PostgreSQL 12 or 13 using the ``postgis`` extension is currently not
   supported. Attempting to upgrade with this extension enabled will result in
   a failed deployment that will require support intervention to fix.

   See the :ref:`postgres_upgrading` section below for more details.

And wire it in ``.symfony.cloud.yaml`` (don't forget to enable the
``pdo_pgsql`` PHP extension):

.. code-block:: yaml

    relationships:
        database: "mydatabase:postgresql"

    runtime:
        extensions:
            - pdo_pgsql

Environment Variables
---------------------

The configuration is exposed via the following environment variables (where
``DATABASE`` is the upper-cased version of the key defined in the relationship
above):

* ``DATABASE_URL``: The database URL (in the PHP or Go format depending on your application)
* ``DATABASE_SERVER``: The database server
* ``DATABASE_DRIVER``: The database driver
* ``DATABASE_VERSION``: The database version
* ``DATABASE_HOST``: The database host
* ``DATABASE_PORT``: The database port
* ``DATABASE_NAME``: The database name
* ``DATABASE_DATABASE``: Alias for ``DATABASE_NAME``
* ``DATABASE_USERNAME``: The database username
* ``DATABASE_PASSWORD``: The database password

.. tip::

    The database version and a default charset is included in the database URL.
    One can override them using the ``DATABASE_VERSION`` and
    ``DATABASE_CHARSET`` environment variables respectively.

Connecting to the Database
--------------------------

From the Application Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Symfony projects using Doctrine or Laravel projects, no configuration is
needed.

For other applications, use the defined environment variables to configure your
framework of choice.

From the Command Line
~~~~~~~~~~~~~~~~~~~~~

To run command on your PostgreSQL server, use a local ``psql`` binary when the
tunnels are open:

.. code-block:: terminal

    $ symfony tunnel:open
    $ symfony run psql -c 'SELECT 1 + 2'

You can also execute commands directly on the server via the ``sql`` command:

.. code-block:: terminal

    $ symfony sql 'SHOW TABLES'

Importing Data
--------------

Export your database in an SQL file making sure that the exported file contains
objects without ownership and using ``IF EXISTS`` clauses. You can use the
following parameters:

.. code-block:: terminal

   $ pg_dump --no-owner --clean --if-exists > database.sql

Then import it via the following command:

.. code-block:: terminal

    $ symfony sql < database.sql

Exporting Data
--------------

Dump data stored in your SymfonyCloud database via the following command:

.. code-block:: terminal

    $ symfony db:dump > database.sql

Extensions
----------

SymfonyCloud supports a number of PostgreSQL extensions. To enable them, list
them under the ``configuration.extensions`` key:

.. code-block:: yaml

    # .symfony/services.yaml
    postgresql:
        type: "postgresql:12"
        disk: 1025
        configuration:
            extensions:
                - pg_trgm
                - hstore

Available extensions
~~~~~~~~~~~~~~~~~~~~

The following is the extensive list of supported extensions. Note that you
cannot currently add custom extensions not listed here.

* **address_standardizer** - Used to parse an address into constituent elements. Generally used to support geocoding address normalization step.
* **address_standardizer_data_us** - Address Standardizer US dataset example
* **adminpack** - administrative functions for PostgreSQL
* **autoinc** - functions for autoincrementing fields
* **bloom** - bloom access method - signature file based index (requires 9.6 or higher)
* **btree_gin** - support for indexing common datatypes in GIN
* **btree_gist** - support for indexing common datatypes in GiST
* **chkpass** - data type for auto-encrypted passwords
* **citext** - data type for case-insensitive character strings
* **cube** - data type for multidimensional cubes
* **dblink** - connect to other PostgreSQL databases from within a database
* **dict_int** - text search dictionary template for integers
* **dict_xsyn** - text search dictionary template for extended synonym processing
* **earthdistance** - calculate great-circle distances on the surface of the Earth
* **file_fdw** - foreign-data wrapper for flat file access
* **fuzzystrmatch** - determine similarities and distance between strings
* **hstore** - data type for storing sets of (key, value) pairs
* **insert_username** - functions for tracking who changed a table
* **intagg** - integer aggregator and enumerator (obsolete)
* **intarray** - functions, operators, and index support for 1-D arrays of integers
* **isn** - data types for international product numbering standards
* **lo** - Large Object maintenance
* **ltree** - data type for hierarchical tree-like structures
* **moddatetime** - functions for tracking last modification time
* **pageinspect** - inspect the contents of database pages at a low level
* **pg_buffercache** - examine the shared buffer cache
* **pg_freespacemap** - examine the free space map (FSM)
* **pg_prewarm** - prewarm relation data (requires 9.6 or higher)
* **pg_stat_statements** - track execution statistics of all SQL statements executed
* **pg_trgm** - text similarity measurement and index searching based on trigrams
* **pg_visibility** - examine the visibility map (VM) and page-level visibility info (requires 9.6 or higher)
* **pgcrypto** - cryptographic functions
* **pgrouting** - pgRouting Extension (requires 9.6 or higher)
* **pgrowlocks** - show row-level locking information
* **pgstattuple** - show tuple-level statistics
* **plpgsql** - PL/pgSQL procedural language
* **postgis** - PostGIS geometry, geography, and raster spatial types and functions
* **postgis_sfcgal** - PostGIS SFCGAL functions
* **postgis_tiger_geocoder** - PostGIS tiger geocoder and reverse geocoder
* **postgis_topology** - PostGIS topology spatial types and functions
* **postgres_fdw** - foreign-data wrapper for remote PostgreSQL servers
* **refint** - functions for implementing referential integrity (obsolete)
* **seg** - data type for representing line segments or floating-point intervals
* **sslinfo** - information about SSL certificates
* **tablefunc** - functions that manipulate whole tables, including crosstab
* **tcn** - Triggered change notifications
* **timetravel** - functions for implementing time travel
* **tsearch2** - compatibility package for pre-8.3 text search functions (obsolete, only available for 9.6 and 9.3)
* **tsm_system_rows** - TABLESAMPLE method which accepts number of rows as a limit (requires 9.6 or higher)
* **tsm_system_time** - TABLESAMPLE method which accepts time in milliseconds as a limit (requires 9.6 or higher)
* **unaccent** - text search dictionary that removes accents
* **uuid-ossp** - generate universally unique identifiers (UUIDs)
* **xml2** - XPath querying and XSLT

.. _postgresql-timezone:

Timezone
--------

The system timezone is set to UTC by default. Change it for the current session
by running ``SQL SET TIME ZONE <timezone>;``.

Troubleshooting
---------------

Could not find driver
~~~~~~~~~~~~~~~~~~~~~

If you see this error: ``Fatal error: Uncaught exception 'PDOException' with
message 'could not find driver'``, this means you are missing the ``pdo_pgsql``
PHP extension. You simply need to enable it in your ``.symfony.cloud.yaml``
(see above).

.. _postgres_upgrading:

Upgrading
---------

.. caution::

    Make sure you first test your migration on a separate environment.
    Be sure to take a snapshot of your master environment **before** you merge this change.

PostgreSQL 10 and later include an upgrade utility that can convert databases
from previous versions to version 10 or 11. If you upgrade your service from a
previous version of PostgreSQL to version 10 or above (by modifying the
``services.yaml`` file) the upgrader will run automatically.

.. caution::

   Upgrading to PostgreSQL 12 using the ``postgis`` extension is currently not
   supported. Attempting to upgrade with this extension enabled will result in
   a failed deployment that will require support intervention to fix.

   If you need to upgrade, you should follow the same steps recommended for
   performing downgrades.

The upgrader does not work to upgrade to PostgreSQL 9 versions, so upgrades
from PostgreSQL 9.3 to 9.6 are not supported. Upgrade straight to version 10 or
11 instead.

Downgrading is not supported. If you want, for whatever reason, to downgrade
you should dump to SQL, remove the service, recreate the service, and import
your dump.

.. _`PostgreSQL`: https://en.wikipedia.org/wiki/PostgreSQL
