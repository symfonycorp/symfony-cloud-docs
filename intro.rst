Platform.sh, the Official Symfony PaaS
======================================

If you are looking for the best way to host your Symfony projects, test `Platform.sh`_, the **Official Symfony PaaS**.

Symfony provides a tight integration with Platform.sh, based on conventions to reduce configuration and make developers more productive.

You can think of Platform.sh as being the hosting framework part of Symfony. It abstracts your project infrastructure and manages it for you: never install nor configure services like a Web server, a MySQL database, or a Redis cache again.

Platform.sh is built on one main idea: your server infrastructure is part of your application, so it should be version controlled along with your application.

Every branch you push to your Git repository can come with bug fixes, new features, **and** infrastructure changes. Everything can then be tested as an independent deployment, including the application code and all services with a copy of their data (database entries, search index, user files, ...).

It really is "what would my site look like if I merge these changes to production?".

The following sections introduce the main concepts and how Symfony projects are deployed on Platform.sh.

The Basics
----------

On Platform.sh, a **project** is linked to a Git repository. A project is composed of one or more **applications**. An application is a directory in your Git repository with a specific Platform.sh configuration and dedicated HTTP endpoints (via the ``.platform.app.yaml`` file).

Projects are deployed in **environments**. An environment is a standalone copy of your live applications which can be used for testing, Q&A, implementing new features, fixing bugs, ...

Every project you deploy on Platform.sh is built as a *virtual cluster*, containing a series of containers. The main branch of your Git repository is always deployed as a production cluster. Any other branch can be deployed as a production, staging, or development cluster.

There are three types of containers within your cluster, all configured by files stored alongside your code:

* The *Router*, configured in ``.platform/routes.yaml``, is a single nginx process responsible for mapping incoming requests to an Application container, and to optionally provide HTTP caching.

* One or more *Applications*, configured via ``.platform.app.yaml`` files, holding the code of your project.

* Some optional *Services*, configured in ``.platform/services.yaml``, like MySQL/MariaDB, Elasticsearch, Redis, or RabbitMQ; they come as optimized pre-built images.

The Workflow
------------

Every time you deploy a branch to Platform.sh, the code is *built* and then *deployed* on a new cluster.

The **build** process looks through the configuration files in your repository and assembles the necessary containers.

The **deploy** process makes those containers live, replacing the previous versions, with no service downtime.

.. _application-build:

Building the Application
~~~~~~~~~~~~~~~~~~~~~~~~

During the build step, any dependencies specified in ``.platform.app.yaml`` are installed on application containers.

You can also customize the build step by providing a ``build`` hook composed of one or more shell commands that help creating your production code base. That could be compiling TypeScript files, running some scripts, rearranging files on disk, or whatever else you want. Note that at this point all you have access to is the filesystem; there are no services or other databases available. Your live website is unaffected.

The default build step for Symfony removes development front controllers, warms up the cache, compiles your assets, and more.

Once all of that is completed, the filesystem is frozen and a read-only container image is created. That filesystem is the final build artifact.

.. _application-deploy:

Deploying the Application
~~~~~~~~~~~~~~~~~~~~~~~~~

Before starting the deployment of your application, we pause all incoming requests and hold them so that there is no downtime.

Then, we stop the current containers and start the new ones. We then open networking connections between the various containers, as specified in the configuration files. The connection information for each service is available as :doc:`environment variables <env>`.

As for the build step, you can define a deploy hook to prepare your application. Your application has complete access to all services, but the filesystem where your code lives is now read-only.

The default deploy step for Symfony replaces the current cache with the newly warmed up one and, for the web container, runs Doctrine migrations if any, and more.

Finally, we open the floodgates and let incoming requests through your newly deployed application. You are done!

Getting Help
------------

If you are facing any issue with Platform.sh, submit a `support ticket`_.

What's Next?
------------

The documentation on symfony.com *only* describes the integration between Platform.sh and Symfony. For detailed documentation about Platform.sh, please refer to its `official documentation`_. To benefit from the Symfony integration, use the `Symfony CLI`_ instead of the official Platform.sh one. The Symfony CLI wraps the Platform.sh one with added features related to Symfony so that you can replace ``platform`` with ``symfony`` when running commands from the documentation.

To get a feeling of what it looks like working with Symfony on Platform.sh, jump to the `Getting Started <getting-started>`_ guide.

.. _`Platform.sh`: https://platform.sh/marketplace/symfony/?utm_source=symfony-cloud-sign-up&utm_medium=backlink&utm_campaign=Symfony-Cloud-sign-up&utm_content=project-in-the-cloud
.. _`support ticket`: https://console.platform.sh/-/users/~/tickets/open
.. _`official documentation`: https://docs.platform.sh/
.. _`Symfony CLI`: https://symfony.com/download
