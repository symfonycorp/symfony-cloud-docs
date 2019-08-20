Users
=====

On SymfonyCloud each project can have several users, when a development team
works on a project, the project owner can decide to give access to other team
members, which can in turn add other members.

.. caution::

    After a new user is added to a project, it needs to be rebuilt.
    Rebuilds are triggered when you deploy a new commit to the environment in
    question. To be able to rebuild without new code changes you can issue the
    command ``git commit --allow-empty -m'rebuild' && symfony deploy`` to
    create an empty commit and "force" rebuilding the environment.

    When the environment rebuild is complete, allow a minute for the routes to
    update fully and for the new user to be able to use SSH access.

Manage user with the CLI
------------------------

You can use the SymfonyCloud command line client to fully manage your users and
integrate this with any other automated system.

Available commands:

* ``user:add``
  Add a user to the project

* ``user:delete``
  Delete a user

* ``user:list (users)``
  List project users

For example the following command would add ``alice@example.com`` to the
current project.

.. code-block:: terminal

    $ symfony user:add alice@example.com

    Summary:
    Email address/Username: alice@example.com
    Role: admin

    WARNING You are going to be charged for adding a user to the project.

    Do you confirm? [Y/n] y

Once this has been done the user will receive an email providing them
instructions to setup the project locally.

Ownership Transfer
------------------

If you want to transfer the ownership of a project to a different user, invite
the user and submit a support ticket to ask for the ownership transfer.

.. note::

    The **owner** account is locked and you can't remove their access before the
    transfer is completed.
