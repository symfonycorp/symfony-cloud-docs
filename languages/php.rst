Running PHP Applications
========================

When running PHP applications, edit the main config file to set the ``type``
property to ``php`` and select the PHP version to use:

.. code-block:: yaml

    # .symfony.cloud.yaml

    # supported versions: 7.3, 7.4, 8.0
    # security-fix only versions: 7.2
    # deprecated versions: 5.4, 5.5, 5.6, 7.0, 7.1
    type: "php:7.4"

PHP code is executed using `PHP-FPM`_ and, as of PHP 7.1, SymfonyCloud uses the
Zend Thread Safe (ZTS) version of PHP.

PHP Extensions
--------------

PHP extensions can be enabled/disabled under the ``runtime.extensions`` and
``runtime.disabled_extensions`` sections of the main config file:

.. code-block:: yaml

    # .symfony.cloud.yaml
    runtime:
        extensions:
            - http
            - redis
            - ssh2
        disabled_extensions:
            - sqlite3

To install specific versions or build an extension not available on PECL or on
SymfonyCloud, use the :ref:`php-ext-install <php-ext-install>` script instead.

Default PHP Extensions
~~~~~~~~~~~~~~~~~~~~~~

The following extensions are enabled by default:

* bcmath
* bz2 (not in 7.0)
* calendar
* ctype
* curl
* date
* dba
* dom
* ereg (5.6 and earlier)
* exif
* fileinfo
* filter
* ftp
* gd
* gettext
* hash
* iconv
* intl
* json
* libxml
* mbstring
* mcrypt (5.6 and earlier)
* mhash (5.6 and earlier)
* mysql (5.6 and earlier)
* mysqli
* mysqlnd
* openssl
* pcntl
* pcre
* pdo
* pdo_mysql
* pdo_sqlite
* phar
* posix
* session
* shmop (5.6 and earlier)
* simplexml
* soap
* sockets
* sqlite3
* sysvmsg
* sysvsem
* sysvshm
* tokenizer
* wddx (5.6 and earlier)
* xml
* xmlreader
* xmlwriter
* zendopcache (5.4 only) / opcache (5.5 and later)
* zip
* zlib

.. _php-extensions-list:

Available PHP Extensions
~~~~~~~~~~~~~~~~~~~~~~~~

This is the complete list of extensions supported by SymfonyCloud and the PHP
versions where they can be enabled:

===============  ===  ===  ===  ===  ===  ===  ===  ===  ===
Extension        5.4  5.5  5.6  7.0  7.1  7.2  7.3  7.4  8.0
===============  ===  ===  ===  ===  ===  ===  ===  ===  ===
amqp                            *    *    *    *    *
apc              *
apcu             *    *    *    *    *    *    *    *
apcu_bc                         *    *    *    *    *
applepay                        *    *         *    *
bcmath                          *    *    *    *    *    *
blackfire        *    *    *    *    *    *    *    *    *
bz2                             *    *    *    *    *    *
calendar                        *    *    *    *    *    *
ctype                           *    *    *    *    *    *
curl             *    *    *    *    *    *    *    *    *
dba                             *    *    *    *    *    *
dom                             *    *    *    *    *    *
enchant          *    *    *    *    *    *    *    *    *
event                                *    *    *    *
exif                            *    *    *    *    *    *
ffi                                                 *    *
fileinfo                        *    *    *    *    *    *
ftp                             *    *    *    *    *    *
gd               *    *    *    *    *    *    *    *    *
gearman          *    *    *
geoip            *    *    *    *    *    *    *    *
gettext                         *    *    *    *    *    *
gmp              *    *    *    *    *    *    *    *    *
http             *    *                        *    *
iconv                           *    *    *    *    *    *
igbinary                        *    *    *    *    *
imagick          *    *    *    *    *    *    *    *
imap             *    *    *    *    *    *    *    *    *
interbase        *    *    *    *    *    *    *    *
intl             *    *    *    *    *    *    *    *    *
ioncube                         *    *    *    *
json                       *    *    *    *    *    *
ldap             *    *    *    *    *    *    *    *    *
mailparse                       *    *    *    *    *
mbstring                        *    *    *    *    *    *
mcrypt           *    *    *    *    *
memcache         *    *    *
memcached        *    *    *    *    *    *    *    *
mongo            *    *    *
mongodb                         *    *    *    *    *
msgpack                    *    *    *    *    *    *
mssql            *    *    *
mysql            *    *    *
mysqli           *    *    *    *    *    *    *    *    *
mysqlnd          *    *    *    *    *    *    *    *    *
newrelic                   *    *    *    *    *    *    *
oauth                           *    *    *    *    *
odbc             *    *    *    *    *    *    *    *    *
opcache               *    *    *    *    *    *    *    *
pdo              *    *    *    *    *    *    *    *    *
pdo_dblib        *    *    *    *    *    *    *    *    *
pdo_firebird     *    *    *    *    *
pdo_mysql        *    *    *    *    *    *    *    *    *
pdo_odbc         *    *    *    *    *    *    *    *    *
pdo_pgsql        *    *    *    *    *    *    *    *    *
pdo_sqlite       *    *    *    *    *    *    *    *    *
pdo_sqlsrv                      *    *    *    *    *
pecl-http                  *
pgsql            *    *    *    *    *    *    *    *    *
phar                            *    *    *    *    *    *
pinba            *    *    *
posix                           *    *    *    *    *    *
propro                     *                   *    *
pspell           *    *    *    *    *    *    *    *    *
pthreads                             *    *
raphf                      *                   *    *
readline         *    *    *    *    *    *    *    *    *
recode           *    *    *    *    *    *    *
redis            *    *    *    *    *    *    *    *
shmop                           *    *    *    *    *    *
simplexml                       *    *    *    *    *    *
snmp             *    *    *    *    *    *    *    *    *
soap                            *    *    *    *    *    *
sockets                         *    *    *    *    *    *
sodium                                    *    *    *    *
sourceguardian                  *    *    *    *
spplus           *    *
sqlite3          *    *    *    *    *    *    *    *    *
sqlsrv                          *    *    *    *    *
ssh2             *    *    *    *    *    *    *    *
sysvmsg                         *    *    *    *    *    *
sysvsem                         *    *    *    *    *    *
sysvshm                         *    *    *    *    *    *
tideways                        *    *    *    *    *    *
tideways_xhprof                 *    *    *    *    *
tidy             *    *    *    *    *    *    *    *    *
tokenizer                       *    *    *    *    *    *
uuid                                 *    *    *    *
wddx                            *    *    *    *
xcache           *    *
xdebug           *    *    *    *    *    *    *    *    *
xhprof           *    *    *
xml                             *    *    *    *    *    *
xmlreader                       *    *    *    *    *    *
xmlrpc           *    *    *    *    *    *    *    *
xmlwriter                       *    *    *    *    *    *
xsl              *    *    *    *    *    *    *    *    *
yaml                                 *    *    *    *
zbarcode                        *    *    *    *
zendopcache      *
zip                             *    *    *    *    *    *
===============  ===  ===  ===  ===  ===  ===  ===  ===  ===

.. note::

    Get the up-to-date complete list of extensions running this command after
    you SSH into your environment: ``ls /etc/php/*/mods-available`` (for PHP 5
    run ``ls /etc/php5/mods-available``).

.. caution::

    .. include:: ../_includes/php-redis-warning.rst

Custom PHP Extensions
~~~~~~~~~~~~~~~~~~~~~

To install custom PHP extensions not available on PECL or on SymfonyCloud, use
the :ref:`php-ext-install <php-ext-install>` script.

If you already have a pre-compiled ``.so`` file, follow these steps instead:

1. Download the ``.so`` file for the extension as part of your build hook using
   ``curl`` or similar.

   .. note::

      Alternatively, if the file is not publicly downloadable, you can also
      build the extension in your :ref:`build section <build-hook>` (although
      this means rebuilding the same extension on every deployment) or add the
      file to your Git repository (although committing large binary blobs to
      Git is generally not recommended).

2. Edit your ``php.ini`` file in the application root (as a sibling of your
   ``.symfony.cloud.yaml`` file) that loads the extension using its absolute
   path:

   .. code-block:: ini

    ; php.ini
    extension=/app/myextension.so

Alternate Start Command
-----------------------

Although PHP runs using PHP-FPM by default, you can also start alternative
processes if desired, such as if you're running an Async PHP daemon, a
thread-based worker process, etc. To do so, define an alternative ``start``
command in the main config file:

.. code-block:: yaml

    # .symfony.cloud.yaml
    web:
        commands:
            start: 'php run.php'
        upstream:
            socket_family: tcp
            protocol: http

The above configuration will execute the ``run.php`` script in the application
root when the container starts, but will not launch PHP-FPM. It will also tell
the front-controller (Nginx) to connect to your application via a TCP socket,
which will be specified in the ``PORT`` environment variable. Note that the
start command **must run in the foreground**.

.. note::

    Note that PHP-FPM cannot run simultaneously along with another persistent
    process (such as ReactPHP or Amp). If you need both they will have to run in
    separate containers.

.. _php_size:

PHP Worker Sizing Hints
-----------------------

The number of workers of a PHP runtime is set automatically and can be checked
by running the following command:

.. code-block:: terminal

    $ symfony ssh -- find /etc/php -name php-fpm.conf -exec grep -e '^pm.max_children' {} \;
    pm.max_children = 2

SymfonyCloud uses the following formula to calculate the number of workers:

.. code-block:: text

                  / Container Memory - Reserved Request Memory     \
    workers = max|  ------------------------------------------- , 2 |
                  \           Average Request Memory               /

**Container Memory**
    The total  memory available for the container, which depends on
    :ref:`the size of the container <config-size>`.

**Average Request Memory**
    The memory that an average request is expected to require. Default value: 45 MB.

**Reserved Request Memory**
    The memory that should be reserved for things that are not specific to a
    request (memory for nginx, the op-code cache, some OS page cache, etc.)
    Default value: 70 MB.

You can tweak the default values if your application will typically consume
more memory or if your application container is swapping a lot. To do so, set
the following options in the main config file:

.. code-block:: yaml

    # .symfony.cloud.yaml
    runtime:
        sizing_hints:
            # values are considered megabytes
            request_memory: 10
            reserved_memory: 70

The ``request_memory`` value must be 1 MB or higher and the ``reserved_memory``
value must be 70 MB or higher.

Measuring PHP Worker Memory Usage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To see how much memory your PHP worker processes are using, look at the PHP
access log:

.. code-block:: terminal

    $ symfony log php.access

In the fifth column, you'll see the peak memory usage that occurred while each
request was handled. A good way to determine an optimal request memory is with
the following command:

.. code-block:: terminal

    $ tail -n5000 /var/log/php.access.log | awk '{print $6}' | sort -n | uniq -c

This will print out a table of how many requests used how much memory, in KB,
for the last 5,000 requests that reached PHP-FPM (increase that number if your
site has lot of traffic). As an example, consider the following output:

.. code-block:: terminal

    4800 2048
     948 4096
     785 6144
     584 8192
     889 10240
     492 12288
     196 14336
      68 16384
       2 18432
       1 22528
       6 131072

This indicates that the majority of requests (4800) used 2048 KB of memory. In
this case that's likely application caching at work. Most requests used up to
around 10 MB of memory, while a few used as much as 18 MB and a very few
(6 requests) peaked at 131 MB.

A conservative approach would suggest an average request memory of 16 MB should
be sufficient. A more aggressive stance would suggest 10 MB. The more
aggressive approach would potentially allow for more concurrent requests at the
risk of some requests needing to use swap memory, thus slowing them down.

.. note::

    If you are running PHP 5.x then don't bother adjusting the worker memory
    usage until you upgrade to PHP 7.x. PHP 7 is way more memory efficient and
    you will likely need less than half as much memory per process with PHP7.

.. _php-configuration:

PHP Configuration
-----------------

There are three ways to customize ``php.ini`` values for your application.

**Option 1.** Create a file called ``php.ini`` in the root directory of the
application. This is the recommended method:

.. code-block:: ini

    ; php.ini
    memory_limit = 256M

**Option 2.** Use the ``variables.php`` property of the main config file:

.. code-block:: yaml

    # .symfony.cloud.yaml
    variables:
        php:
            memory_limit: 256M

**Option 3.** Define the values as env vars for some specific environment. This
method is mostly useful on development environments to enable error outputting,
enable the Xdebug extension, etc.:

.. code-block:: terminal

    $ symfony var:set --env-level php:memory_limit=256M

Settings defined as environment variables have precedence over the ones defined
in ``.symfony.cloud.yaml``, which has precedence over the ones defined in
``php.ini``.

.. caution::

   There's no limit on what you can define in your PHP configuration, but many
   settings can break your application. This is a feature for advanced users.

Default ``php.ini`` values
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: ini

   ; php.ini
   max_execution_time=300
   max_input_time=60
   memory_limit=128M
   post_max_size=64M
   upload_max_filesize=64M
   max_file_uploads=20
   max_input_vars=3000
   opcache.memory_consumption=64
   opcache.validate_timestamps=On
   variables_order="EGPCS"
   display_errors=On
   zend.assertions=-1
   session.save_path=/tmp/sessions

Error Handling
~~~~~~~~~~~~~~

By default, ``display_errors`` is set to ``On`` to ease setting up your project.
We strongly recommend providing a custom error handler in your application or
setting this value to ``Off`` before you make your site live and this is done
automatically for you during initial configuration using the ``project:init``
command:

.. configuration-block::

    .. code-block:: ini

        ; php.ini
        display_errors=Off

    .. code-block:: yaml

        # .symfony.cloud.yaml
        variables:
            php:
                display_errors: Off

.. _php-timezone:

PHP Timezone
~~~~~~~~~~~~

You can change the timezone of the PHP runtime with the following configuration:

.. configuration-block::

    .. code-block:: ini

        ; php.ini
        date.timezone="Europe/Paris

    .. code-block:: yaml

        # .symfony.cloud.yaml
        variables:
            php:
                "date.timezone": "Europe/Paris"

Check out the :doc:`main Timezone article </cookbooks/timezone>` to learn more
about setting the timezone of other services.

Runtime configuration
~~~~~~~~~~~~~~~~~~~~~

In addition to `PHP Configuration`_, SymfonyCloud also lets you configure the
PHP-FPM runtime via the ``runtime`` section on the ``.symfony.cloud.yaml`` file.
The following options are configurable:

* ``request_terminate_timeout``:
  The timeout for serving a single request after which the PHP-FPM worker
  process will be killed. This option should be used when the
  ``max_execution_time`` ini option does not stop script execution for some
  reason.

  .. code-block:: yaml

      # .symfony.cloud.yaml
      runtime:
          request_terminate_timeout: 300

* ``sizing_hints``:
  Hints given to SymfonyCloud to compute the optimal number of workers serving
  HTTP requests. See `PHP Worker Sizing Hints`_ for more information.

  .. code-block:: yaml

      # .symfony.cloud.yaml
      runtime:
          sizing_hints:
              # values are considered megabytes
              request_memory: 10
              reserved_memory: 70

Debug PHP-FPM
-------------

Use the ``env:fpm:status`` command to inspect what's going on with PHP-FPM:

.. code-block:: terminal

    $ symfony env:fpm:status

    pool:                 web
    process manager:      ondemand
    start time:           10/Jan/2019:08:55:43 +0000
    start since:          365469
    accepted conn:        17329
    listen queue:         0
    max listen queue:     0
    listen queue len:     0
    idle processes:       0
    active processes:     1
    total processes:      1
    max active processes: 2
    max children reached: 102
    slow requests:        0

    ************************
    pid:                  20216
    state:                Running
    start time:           14/Jan/2019:14:26:14 +0000
    start since:          38
    requests:             3
    request duration:     155
    request method:       GET
    request URI:          /-/status?&full
    content length:       0
    user:                 -
    script:               -
    last request cpu:     0.00
    last request memory:  0

.. _`PHP-FPM`: https://php-fpm.org/
