Cron Jobs
=========

Cron jobs allow you to run scheduled tasks at specified times or intervals. The
``crons`` section of ``.symfony.cloud.yaml`` describes these tasks and the
schedule when they are triggered. Each item in the list has a unique name
identifying a separate cron job:

.. code-block:: yaml

    timezone: Europe/Paris

    crons:
        update_repo:
            # every day at 1h15 AM
            spec: 15 1 * * *
            cmd: |
                croncape sh -c "if [ ! -d var/data/code/.git ]; then \
                git clone https://github.com/... var/data/code; else \
                cd var/data/code && git fetch && \
                git reset --hard origin/master; fi"
        update_doc:
            # every day at 3h45 AM
            spec: 45 3 * * *
            cmd: croncape bin/console app:doc-update

Time Specification
------------------

The ``spec`` property follows the `IEEE standard crontab
<https://pubs.opengroup.org/onlinepubs/9699919799/>`_.

.. _cron-timezone:

By default, the time is interpreted in *UTC* timezone. Change the timezone used
for cron tasks by setting the global ``timezone`` property. Its value is one
of the `tz database region codes
<https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>`_.

Command to run
--------------

The ``cmd`` property is the command that should be executed. The command is
executed in the ``/app`` directory by the same user as the Web one.

To get feedback when something goes wrong, prefix the command with
``croncape``. ``croncape`` will send an email to the address defined by the
``MAILTO`` environment variable. Don't forget to set it first via the following
command:

.. code-block:: terminal

    $ symfony var:set MAILTO=sysadmin@example.com

.. tip::

    The minimum interval between cron runs is 5 minutes, even if specified as
    less. Additionally, a variable delay is added to each cron job in each
    project in order to prevent host overloading when every project tries to run
    their nightly tasks at the same time. Your crons will not run exactly at the
    time that you specify, but will be delayed by 0-300 seconds.

.. caution::

    Cron runs are executed using the dash shell, not the bash shell used by
    regular SSH sessions. In most cases that makes no differences but may
    impact some more involved cron scripts.

.. note::

    If an application defines both a ``web`` instance and a ``worker`` instance,
    cron tasks will be run only on the ``web`` instance.

.. tip::

    If you want to run a command in a cron hook for specific environments, check
    the ``SYMFONY_BRANCH`` environment variable:

    .. code-block:: yaml

        crons:
            snapshot:
                spec: 0 5 * * *
                cmd: |
                    # only run for the master branch, aka production
                    if [ "$SYMFONY_BRANCH" = "master" ]; then
                        croncape symfony ... --no-wait
                    fi

.. note::

    To ensure better reliability, by default ``croncape`` sends its emails
    using ``project-id@cron.noreply.s5y.io`` as the sender address
    (``project-id+branch@cron.noreply.s5y.io`` for non-master environments) and
    the provided :doc:`SymfonyCloud SMTP </services/emails>` service; even if
    you define your own ``MAILER_*`` environment variables.

    If you wish to use a custom SMTP and/or use a custom sender address you need
    to follow these steps:

    #. Define the sender address by defining the ``MAILFROM`` environment
       variable;
    #. Define the environment variables required to use your own email
       service, refers to the :doc:`email </services/emails>`
       documentation to check their names. Please note that only SMTP
       connections are supported;
    #. Disable the provided SMTP service using
       ``symfony env:setting:set smtp off``

Running a Cron Manually
-----------------------

You can force a cron to run manually by running the ``cron`` command (it takes
the cron name as an argument as specified in ``.symfony.cloud.yaml``):

.. code-block:: terminal

    $ symfony cron update_doc
