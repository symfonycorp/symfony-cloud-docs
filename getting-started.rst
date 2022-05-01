Getting Started
===============

.. _cli-install:

Installing the Symfony CLI Tool
-------------------------------

To manage your Symfony projects with Platform.sh, you need Git, SSH, and the ``symfony`` CLI tool. Go to the `Download`_ page for instructions on how to install the Symfony CLI tool on your local machine.

To get started with the Symfony CLI tool, run ``symfony`` to get some common commands or ``symfony help`` to list all available commands.

.. tip::

    Even if you can use the Platform.sh CLI tool, we highly recommend you to use the Symfony CLI tool as it provides a tighter integration with Symfony via specific commands. When using a command from the `official Platform.sh documentation`_, replace ``platform`` with ``symfony``.

Deploying a Project on Platform.sh
----------------------------------

There are three steps needed to deploy a project on Platform.sh:

* Configure the project by describing its infrastructure;

* Create the project on Platform.sh;

* Deploy the project.

Configuring Platform.sh for a Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to play with Platform.sh with a simple sample project, create a Symfony Demo project:

.. code-block:: terminal

    $ symfony new --demo --cloud /path/to/demo
    $ cd /path/to/demo

Platform.sh manages the entire infrastructure of your project, from code to services (databases, queues, search, ...), from email sending to crons and workers. This infrastructure is described through configuration files, stored along side your code. The ``--cloud`` flag automatically generates the Platform.sh configuration files.

If you want to deploy an existing project, generate a sensible default Platform.sh configuration from within the project's directory:

.. code-block:: terminal

    $ symfony project:init

The command generates a default set of configuration files: ``.platform.app.yaml``, ``.platform/services.yaml``, ``.platform/routes.yaml``, and ``php.ini``.

Don't forget to commit the new files in your repository:

.. code-block:: terminal

    $ git add .platform.app.yaml .platform/services.yaml .platform/routes.yaml php.ini
    $ git commit -m "Add Platform.sh configuration"

If you have a closer look at ``.platform.app.yaml`` for a Symfony project, you will notice the calls to ``symfony-build`` and ``symfony-deploy`` scripts during the build and deploy hooks respectively. These scripts register some environment variables depending on the services you require (names match the one expected by Symfony recipes).  They also build the application cache and run the database migrations if any. They should cover most use cases for Symfony applications.

Creating a Project in the Cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then, create a new Platform.sh project (you will need to create a Platform.sh account):

.. code-block:: terminal

    $ symfony project:create --title=demo --plan=development

Deploying a Project
~~~~~~~~~~~~~~~~~~~

Finally, deploy the project to the cloud:

.. code-block:: terminal

    $ symfony deploy

.. note::

    If you have private dependencies, you might need to `authorize Platform.sh`_ to let Platform.sh access them during project build.

Check that everything went fine by opening the deployed URL:

.. code-block:: terminal

    $ symfony cloud:url --primary

.. _working-with-environments:

Working on a Project
--------------------

Now that the project is deployed, let's describe a typical scenario where you want to fix a bug or add a new feature.

First, you need to know that the ``main`` branch always represents the **production** environment. Any other branch is for developing new features, fixing bugs, or updating the infrastructure.

Let's create a new environment (a Git branch) to make some changes, without impacting production:

.. code-block:: terminal

    $ git checkout main
    $ symfony env:branch feat-a

This command creates a new local ``feat-a`` branch based on the ``main`` branch and activate a related environment on Platform.sh. If you have some services enabled, the new environment inherits the data of the parent environment (the production one here).

Let's make some simple visual changes. If you have created a Symfony demo application, edit the ``templates/default/homepage.html.twig`` template and make the following change:

.. code-block:: diff

    # templates/default/homepage.html.twig
    {% block body %}
        <div class="page-header">
    -        <h1>{{ 'title.homepage'|trans|raw }}</h1>
    +        <h1>Welcome to the Platform.sh Demo</h1>
        </div>

        <div class="row">

.. tip::

    If you want to check that the change is correct on your local machine, run ``symfony server:start -d`` and ``symfony open:local`` to test it in your local browser.

Commit the change:

.. code-block:: terminal

    $ git commit -a -m "Update text"
    # in a real-life scenario, you would also push the change to the upstream Git repository

And deploy the change to the ``feat-a`` environment:

.. code-block:: terminal

    $ symfony deploy

Browse the new version and notice that the domain name is different now (each environment has its own domain name):

.. code-block:: terminal

    $ symfony cloud:url --primary

Iterate by changing the code, committing, and deploying. When satisfied with the changes, merge it to main, deploy, and remove the feature branch:

.. code-block:: terminal

    $ git checkout main
    $ git merge feat-a
    $ symfony env:delete feat-a
    $ git branch -d feat-a
    $ symfony deploy

.. note::

    Note that deploying production was fast as it reused the image built for the ``feat-a`` environment.

.. tip::

    For a long running branch, you can keep the code up-to-date with ``main`` via ``git merge main`` or ``git rebase main``. And you can also keep the data in sync with the production environment via ``symfony env:sync``.

.. _`official Platform.sh documentation`: https://docs.platform.sh/
.. _`Download`: https://symfony.com/download
.. _`authorize Platform.sh`: https://docs.platform.sh/guides/general/composer-auth.html#private-repository-hosting
