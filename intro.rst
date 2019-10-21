What is SymfonyCloud?
=====================

Welcome to **SymfonyCloud**, the *Symfony PaaS*. SymfonyCloud is a
Platform-as-a-Service built and optimized for Symfony developers.

SymfonyCloud is tightly integrated with the Symfony ecosystem and its
conventions to make developers more productive.

You can think of SymfonyCloud as being the hosting framework part of Symfony. It
abstracts your project infrastructure and manages it for you: never install nor
configure services like a web server, a MySQL database, or a Redis cache again.

SymfonyCloud is built on one main idea: your server infrastructure is part of
your application, so it should be version controlled along with your
application.

Every branch you push to your Git repository can come with bug fixes, new
features, **and** infrastructure changes. Everything can then be tested as an
independent deployment, including the application code and all services with a
copy of their data (database entries, search index, user files, ...).

It really is "what would my site look like if I merge these changes to
production?"

The Basics
----------

On SymfonyCloud, a **project** is linked to a Git repository. A project is
composed of one or more **applications**. An application is a directory in your
Git repository with a specific SymfonyCloud configuration and dedicated HTTP
endpoints (via the ``.symfony.cloud.yaml`` file).

Projects are deployed in **environments**. An environment is a standalone copy
of your live applications which can be used for testing, Q&A, implementing new
features, fixing bugs, ...

Every project you deploy on SymfonyCloud is built as a *virtual cluster*,
containing a series of containers. The master branch of your Git repository is
always deployed as a production cluster. Any other branch can be deployed as a
development cluster.

There are three types of containers within your cluster, all configured by
files stored along side your code:

* The *Router*, configured in ``.symfony/routes.yaml``, is a single nginx
  process responsible for mapping incoming requests to an Application
  container, and to optionally provide HTTP caching.

* One or more *Applications*, configured via ``.symfony.cloud.yaml`` files,
  holding the code of your project.

* Some optional *Services*, configured in ``.symfony/services.yaml``, like
  MySQL/MariaDB, Elasticsearch, Redis, or RabbitMQ; they come as optimized
  pre-built images.

The Workflow
------------

Every time you deploy a branch to SymfonyCloud, the code is *built* and then
*deployed* on a new cluster.

The **build** process looks through the configuration files in your repository
and assembles the necessary containers.

The **deploy** process makes those containers live, replacing the previous
versions, with no service downtime.

.. _application-build:

Building the Application
~~~~~~~~~~~~~~~~~~~~~~~~

During the build step, any dependencies specified in ``.symfony.cloud.yaml`` are
installed on application containers.

You can also customize the build step but providing a ``build`` hook composed of
one or more shell commands that help creating your production code base. That
could be compiling TypeScript files, running some scripts, rearranging files on
disk, or whatever else you want. Note that at this point all you have access to
is the filesystem; there are no services or other databases available. Your live
website is unaffected.

The default build step for Symfony removes development front controllers and
warms up the cache.

Once all of that is completed, the filesystem is frozen and a read-only
container image is created. That filesystem is the final build artifact.

.. _application-deploy:

Deploying the Application
~~~~~~~~~~~~~~~~~~~~~~~~~

Before starting the deployment of your application, we pause all incoming
requests and hold them so that there is no downtime.

Then, we stop the current containers and start the new ones. We then open
networking connections between the various containers, as specified in the
configuration files. The connection information for each service is available
as environment variables.

As for the build step, you can define a deploy hook to prepare your application.
Your application has complete access to all services, but the filesystem where
your code lives is now read-only.

The default deploy step for Symfony replaces the current cache with the newly
warmed up one and, for the web container, runs Doctrine migrations if any.

Finally, we open the floodgates and let incoming requests through your newly
deployed application. You are done!

Getting Help
------------

If you're facing any issue with SymfonyCloud, you can submit a support ticket
from a terminal via ``symfony open:support`` or visit the `support webpage
<https://symfony.com/cloud/support>`_.

What's Next?
------------

The next step to work with SymfonyCloud is to install the `Symfony CLI Tool
<getting-started>`_.
