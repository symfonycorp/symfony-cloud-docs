Blackfire
=========

`Blackfire.io <https://blackfire.io/>`_ is a PHP profiler and automated
performance testing tool that can be used in the development, Integration,
Staging, and Production environments.

It provides details information on your PHP code's resources consumption across
Wall-Time, CPU, I/O, Memory, Network Calls, HTTP requests and SQL queries.

In addition, it can profile your code automatically and notify you whenever your
code does not comply with best practices for PHP, Symfony, Drupal, eZPlatform,
Typo3 & Magento code performance management.

Blackfire is installed by default on SymfonyCloud and can be enabled at any time:

.. code-block:: yaml

   # .symfony.cloud.yaml
   runtime:
       extensions:
           - blackfire

Commit and deploy the changes to enable Blackfire:

.. code-block:: terminal

   $ git add .symfony.cloud.yaml
   $ git commit -m "Enable Blackfire."
   $ symfony deploy

Next, go to your Blackfire Dashboard, create a new
environment `under the Environments tab <https://blackfire.io/my/environments>`_
and specify your credentials:

.. code-block:: terminal

   $ symfony var:set BLACKFIRE_SERVER_ID=<insert your Server ID> BLACKFIRE_SERVER_TOKEN=<Insert your Server Token>

Going further with Blackfire
----------------------------

Blackfire can also leverage :doc:`SymfonyCloud's webhook <webhooks>` to enable
automatic profiling and performance testing each time you deploy new code. Check
their `documentation <https://blackfire.io/docs/integrations/paas/symfonycloud#builds-level-enterprise>`_
for more information.
