MySQL/MariaDB
=============

`MariaDB`_ is a MySQL-compatible relational database system. Its XtraDB storage
engine is equivalent to MySQL with InnoDB.

To use it in your application, add it to your ``.symfony/services.yaml``:

.. code-block:: yaml

    # .symfony/services.yaml
    mydatabase:
        # supported versions: 10.0, 10.1, 10.2, 10.3, 10.4, 10.5
        # 5.5 is also available but not maintained upstream
        type: mysql:10.4
        disk: 1024

And wire it in ``.symfony.cloud.yaml`` (don't forget to enable the
``pdo_mysql`` PHP extension):

.. code-block:: yaml

    # .symfony.cloud.yaml
    relationships:
        database: "mydatabase:mysql"

    runtime:
        extensions:
            - pdo_mysql

Oracle's `MySQL`_ is also available using ``oracle-mysql``:

.. code-block:: yaml

    # .symfony/services.yaml
    mydatabase:
        # supported versions: 5.7, 8.0
        type: oracle-mysql:8.0
        disk: 1024

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

.. note::

    The password is usually empty when using only the default endpoint on
    MySQL/MariaDB. It will be filled in when you define custom endpoints. This
    does not create a security risk as port access is tightly restricted to
    your application container.

Connecting to the Database
--------------------------

From the Application Container
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Symfony projects using Doctrine or Laravel projects, no configuration is
needed.

For other applications, use the defined environment variables to configure your
framework of choice.

.. note::

    1. There is a single MySQL user, so you can not use "DEFINER" Access
    Control mechanism for Stored Programs and Views.

    2. MySQL Errors such as "PDO Exception 'MySQL server has gone away'" are
    usually simply the result of exhausting your existing diskspace. Be sure you
    have sufficient space allocated to the service in
    ``.symfony/services.yaml``.

From the Command Line
~~~~~~~~~~~~~~~~~~~~~

To run commands on your MariaDB server, run the following command:

.. code-block:: terminal

    $ symfony sql 'SHOW TABLES'

Importing Data
--------------

Export your database in an SQL file or in a compressed file and import it via
the following command:

.. code-block:: terminal

    $ symfony sql < database.sql

Exporting Data
--------------

Dump data stored in your SymfonyCloud database via the following command:

.. code-block:: terminal

    $ symfony db:dump > database.sql

.. _mysql-timezone:

Timezone
--------

The system timezone is set to UTC by default. Change it for the current
connection with ``SQL SET time_zone = <timezone>;``.

Multiple databases
------------------

If you are using version 10.0 or later of this service it is possible to define
multiple databases as well as multiple users with different permissions. To do
so requires defining multiple endpoints. Under the ``configuration`` key of your
service, there are two additional keys:

* ``schemas``: This is a YAML array listing the databases that should be
  created. If not specified, a single database named ``main`` will be created.

* ``endpoints``: This is a nested YAML array defining different credentials.
  Each endpoint may have access to one or more schemas (databases), and may have
  different levels of permission on each. The valid permission levels are:

  * ``ro``: Only SELECT queries are allowed

  * ``rw``: SELECT queries as well INSERT/UPDATE/DELETE queries are allowed

  * ``admin``: All queries are allowed, including DDL queries (CREATE TABLE,
    DROP TABLE, ...)

Consider the following illustrative example:

.. code-block:: yaml

    # .symfony/services.yaml
    mysqldb:
        type: mysql:10.2
        disk: 2048
        configuration:
            schemas:
                - main
                - legacy
            endpoints:
                admin:
                    default_schema: main
                    privileges:
                        main: admin
                        legacy: admin
                reporter:
                    privileges:
                        main: ro
                importer:
                    default_schema: legacy
                    privileges:
                        legacy: rw

This creates a MySQL/MariaDB service named ``mysqldb`` which has two databases,
``main`` and ``legacy``. There will be three endpoints created. The first, named
``admin``, will have full access to both databases. The second, ``reporter``,
will have SELECT query access to the ``main`` DB but no access to ``legacy`` at
all. The ``importer`` user will have SELECT/INSERT/UPDATE/DELETE access (but not
DDL access) to the ``legacy`` database but no access to ``main``.

If a given endpoint has access to multiple databases you should also specify
which will be listed by default in the relationships array. If one isn't
specified the `path` property of the relationship will be ``null``. While that
may be acceptable for an application that knows the name of the database to
connect to, it would mean that automated tools such as the Symfony CLI will
not be able to access the database on that relationship. For that reason the
``default_schema`` property is always recommended.

Once those endpoints are defined, you need to expose them to your application as
a relationship. Continuing with our example, this would be a possible
corresponding block under ``relationships``:

.. code-block:: yaml

    # .symfony.cloud.yaml
    relationships:
        database: "mysqldb:admin"
        reports: "mysqldb:reporter"
        imports: "mysqldb:importer"

This block defines three relationships, ``database``, ``reports``, and
``imports``. They'll be available as environment variables with the
``DATABASE_``, ``REPORTS_``, and ``IMPORTS_`` prefixes. Use those to connect to
the appropriate database with the specified restrictions.

If no ``configuration`` block is specified at all, it is equivalent to the
following default:

.. code-block:: yaml

    # .symfony/services.yaml
    configuration:
        schemas:
            - main
        endpoints:
            mysql:
                default_schema: main
                privileges:
                    main: admin

.. caution::

    If either schemas or endpoints are defined, then no default will be applied
    and you must specify the full configuration.

Advanced Configuration
----------------------

For version 10.2 and later, some MariaDB configuration properties from the
``my.cnf`` file can be customized:

.. code-block:: yaml

    # .symfony/services.yaml
    mysqldb:
        type: mysql:10.2
        disk: 2048
        configuration:
            properties:
                # size in Mb (defaults to 16)
                max_allowed_packet: 64

Note that increasing the size of the maximum packet will also automatically
decrease the ``max_connections`` value. The number of connections allowed will
depend on the packet size and the memory available to the service. In most cases
leaving this value at the default is recommended.

Troubleshooting
---------------

Worker timeout
~~~~~~~~~~~~~~

Another possible cause of "MySQL server has gone away" errors is a server
timeout. MySQL has a built-in timeout for idle connections, which defaults to
10 minutes. Most typical web connections end long before that is ever
approached, but it's possible that a long-running worker may idle and not need
the database for longer than the timeout. In that case the same "server has
gone away" message may appear.

If that's the case, the best way to handle it is to wrap your connection logic
in code that detects a "server has gone away" exception and tries to
re-establish the connection.

Alternatively, if your worker is idle for too long it can self-terminate.
SymfonyCloud will automatically restart the worker process, and the new process
can establish its own new database connection.

.. _`MariaDB`: https://en.wikipedia.org/wiki/MariaDB
.. _`MySQL`: https://en.wikipedia.org/wiki/MySQL
