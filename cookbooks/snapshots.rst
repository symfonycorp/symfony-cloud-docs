Snapshots
=========

A snapshot is a complete backup of an environment, including all persistent
data from running services and files stored in mounted volumes.

Snapshots and Restore
---------------------

Snapshots can be created with the following command:

.. code-block:: terminal

    $ symfony env:snapshot:create

.. caution::

    Triggering a snapshot causes a momentary pause in site availability so that
    all requests can complete, allowing the snapshot to be taken against a known
    consistent state. The total interruption is usually only 1-2 seconds and any
    requests during that time are held temporarily, not dropped.

Use snapshot IDs via ``symfony env:snapshots`` to restore them:

.. code-block:: terminal

    $ symfony env:snapshots
    $ symfony env:snapshot:restore 92c9a4b2aa75422efb3d

.. tip::

    It is strongly recommended to make a snapshot of the live environment
    before merging an environment to the live environment, or when increasing
    the storage space of a service.

Automated Snapshots
-------------------

No snapshots are triggered automatically. You can use the
``env:snapshot:create`` command to automate snapshots, including from the
application container itself via a cron task:

.. code-block:: yaml

    crons:
        snapshot:
            # everyday at 5AM
            spec: 0 5 * * *
            cmd: |
                # only snapshot the master environment, aka production
                if [ "$SYMFONY_BRANCH" = master ]; then
                    croncape symfony env:snapshot:create --no-wait
                fi

.. caution::

   For the command to work, you first need to
   :doc:`set up an API authentication token </cookbooks/api_tokens>`.

.. note::

   Please do not schedule more than one snapshot a day to minimize data usage.

Retention
---------

Snapshots are guaranteed to be kept for at least 7 days.
