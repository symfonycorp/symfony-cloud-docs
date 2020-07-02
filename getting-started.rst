Getting Started
===============

Requirements
------------

Before getting started, check that you have Git installed and SSH properly
configured as SymfonyCloud relies on both quite heavily. This also means that
your project must use Git as its version control system.

.. _cli-install:

Installing the CLI Tool
-----------------------

To manage your Symfony projects with SymfonyCloud, install the ``symfony``
CLI tool:

.. code-block:: terminal

    $ curl -sS https://get.symfony.com/cli/installer | bash

.. note::

    On Windows, download and run the `Windows installer
    <https://get.symfony.com/cli/setup.exe>`_ instead.

Then, authenticate with your `SymfonyConnect credentials
<https://connect.symfony.com/>`_:

.. code-block:: terminal

    $ symfony login

You are now all set! To get started with the Symfony CLI tool, run ``symfony``
to get some common commands or ``symfony help`` to list all available commands.

If you want more detailed information about a command, use ``symfony help
COMMAND_NAME``, or ``symfony COMMAND_NAME --help``. The second one is useful
when you have already added some arguments and/or flags like in ``symfony
something --flag arg --help``.

Deploying a Project on SymfonyCloud
-----------------------------------

There are three steps needed to deploy a project on SymfonyCloud:

* Configure the project by describing its infrastructure (including its
  services);

* Create the project on SymfonyCloud;

* Deploy the project.

If you want to deploy a sample project on SymfonyCloud, create a Symfony Demo
project with the following command:

.. code-block:: terminal

    $ symfony new --demo /path/to/demo
    $ cd /path/to/demo

Configuring the Project
~~~~~~~~~~~~~~~~~~~~~~~

SymfonyCloud manages the entire infrastructure of your project, from code to
services (databases, queues, search, ...), from email sending to crons and
workers. This infrastructure is described through configuration files, stored
along side your code.

From within the project's directory, generate a sensible default SymfonyCloud
configuration:

.. code-block:: terminal

    $ symfony project:init

The command detects the language and the framework used by the project and it
generates a default configuration in the following files:
``.symfony.cloud.yaml``, ``.symfony/services.yaml``, ``.symfony/routes.yaml``,
and ``php.ini``.

Don't forget to commit the new files in your repository:

.. code-block:: terminal

    $ git add .symfony.cloud.yaml .symfony/services.yaml .symfony/routes.yaml php.ini
    $ git commit -m "Add SymfonyCloud configuration"

If you have a closer look at ``.symfony.cloud.yaml`` for a Symfony project, you
will notice the calls to ``symfony-build`` and ``symfony-deploy`` scripts during
the build and deploy hooks respectively. These scripts register some environment
variables depending on the services you require (names match the one expected by
Symfony recipes). They also build the application cache and run the database
migrations if any. They should cover most use cases for Symfony applications.

.. note::

    By default, the ``symfony new`` command does not automatically configure
    the project for SymfonyCloud; the ``--cloud`` flags allows you to create a
    project and automatically configure it for SymfonyCloud at once.

Creating the Project in the Cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then, create a new SymfonyCloud project:

.. code-block:: terminal

    $ symfony project:create --title=demo --plan=development

.. caution::

    .. include:: _includes/trial-warning.rst

Deploying the Project
~~~~~~~~~~~~~~~~~~~~~

.. tip::

    If you didn't do it yet, now is the perfect moment to upload your SSH key
    to SymfonyCloud. You can do it using the ``symfony account:ssh:key:add``,
    use ``symfony account:ssh:keys`` to see the list of keys you already
    uploaded.

Finally, deploy the project to the cloud:

.. code-block:: terminal

    $ symfony deploy

.. note::

    If you have private dependencies, you might need to authorize SymfonyCloud
    to get them. Use ``symfony project:deploy-key`` displays the SSH key
    SymfonyCloud uses.

If you have created a Symfony demo application, you need to set some environment
variables to make it work properly:

.. code-block:: terminal

    $ symfony var:set --file=.env

Check that everything went fine by opening the deployed URL:

.. code-block:: terminal

    $ symfony open:remote

.. note::

    Note that SSL has been configured automatically for your application thanks
    to `Let's Encrypt <https://letsencrypt.org/>`_.

.. _working-with-environments:

Working on a Project
--------------------

Now that the project is deployed, let's describe a typical scenario where you
want to fix a bug or add a new feature.

First, you need to know that the ``master`` branch always represents the
**production** environment. Any other branch is for developing new features,
fixing bugs, or updating the infrastructure.

Let's create a new environment (a Git branch) to make some changes, without
impacting production:

.. code-block:: terminal

    $ git checkout master
    $ symfony env:create feat-a

This command creates a new local ``feat-a`` branch based on the ``master``
branch and activate a related environment on SymfonyCloud. If you have some
services enabled, the new environment inherits the data of the parent
environment (the production one here).

Let's make some simple visual changes. If you have created a Symfony demo
application, edit the ``templates/default/homepage.html.twig`` template and make
the following change:

.. code-block:: diff

    # templates/default/homepage.html.twig
    {% block body %}
        <div class="page-header">
    -        <h1>{{ 'title.homepage'|trans|raw }}</h1>
    +        <h1>Welcome to the SymfonyCloud Demo</h1>
        </div>

        <div class="row">

.. tip::

    If you want to check that the change is correct on your local machine, run
    ``symfony server:start`` and ``symfony open:local`` to test it in your
    local browser.

Commit the change:

.. code-block:: terminal

    $ git commit -a -m "Update text"
    $ # in a real-life scenario, you would push the change to the upstream Git repository

And deploy the change to the ``feat-a`` environment:

.. code-block:: terminal

    # the 'env:create' command checkouts into the new branch automatically,
    # but if you changed it for some reason, checkout the 'feat-a' branch again
    # $ git checkout feat-a

    $ symfony deploy

Browse the new version and notice that the domain name is different now (each
environment has its own domain name):

.. code-block:: terminal

    $ symfony open:remote

Iterate by changing the code, committing, and deploying. When satisfied with the
changes, merge it to master, deploy, and remove the feature branch:

.. code-block:: terminal

    $ git checkout master
    $ git merge feat-a
    $ git branch -d feat-a
    $ symfony env:delete feat-a
    $ symfony deploy

.. note::

    Note that deploying production was fast as it reused the image built on the
    ``feat-a`` environment.

.. tip::

    For a long running branch, you can keep the code up-to-date with ``master``
    via ``git merge master`` or ``git rebase master``. And you can also keep the
    data in sync with the production environment via ``symfony env:sync``.

.. _delete-project:

Deleting a project
------------------

Use ``symfony project:delete`` to delete a SymfonyCloud project (optionally
specifying the project ID using the ``--project`` flag); use
``symfony projects`` to list all active projects under your account.
