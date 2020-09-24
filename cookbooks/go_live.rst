Going Live
==========

.. tip::

    The order of operations is not really important, but if you are
    migrating a project from another provider, you should first :ref:`configure
    domains <setup_domain>` on the SymfonyCloud side, and only then :ref:`switch
    DNS <switch_dns>` over.

1. Production plan
------------------

First, check via ``project:info`` that your project is not on the development
plan, as domains can only be attached to production plans:

.. code-block:: terminal

   $ symfony project:info
    Project ID:		nq3xa5jtywvyc
    Project title:	Symfony Documentation
    Status:		ok provisioned
    Owner:		oss-cloud@symfony.com
    Region:		Europe (West)
    Plan:		Development
    Environments:	95/255 actives
    Storage:		5 Gb
    Users:		6/6

To move to a production plan, use the ``project:scale`` command:

.. code-block:: terminal

   $ symfony project:scale
   Scaling from Development to Standard...

     CPU    dev -> 1
     Memory dev -> 0.8GB
     Apps   dev -> 3     (including workers and services)


   WARNING You are going to be charged for scaling up the project.
           Your credit card will be charged at the end of each month.

   Do you confirm? [Y/n] Y
   Project updated

.. _setup_domain:

2. Setup a domain
-----------------

Attach a domain to your project via ``domain:attach`` to allow SymfonyCloud edge
layer to route requests to the project for a specific domain:

.. code-block:: terminal

   $ symfony domain:attach example.com
   Waiting for activity 22jdotpwtrxz6 (Tugdual Saunier added domain example.com):
    Building application 'app' (runtime type: php:7.3, tree: a4a225e)
      Reusing existing build for this tree ID

    Provisioning certificates
      (Next refresh will be at 2019-05-28 11:39:30+00:00.)
      Environment certificates
      - certificate 84c845e: expiring on 2019-06-25 11:39:30+00:00, covering example.com


    Re-deploying environment nq3xa5jtywvyc-master-7rqtwti

   Activity 22jdotpwtrxz6 succeeded
   The domain example.com has been attached.

.. tip::

    Multiple domains can be attached to a project. :ref:`wildcard_routes` (like
    ``*.example.com``) are also supported using the ``.symfony/routes.yaml``
    file.

If you require access to the project before the domain name switch is done, you
can create a ``hosts`` file entry on your computer and point it directly to one
SymfonyCloud IP. To get SymfonyCloud IP addresses, run
``symfony account:ips --ingress``.

On macOS and Linux, you can add that IP to your ``/etc/hosts`` file. On Windows
the file is named ``c:\Windows\System32\Drivers\etc\hosts``. You will need to be
an admin user to be able to modify that file. After adding the line the file will
look something like:

.. code-block:: text

   ##
   # Host Database
   #
   # localhost is used to configure the loopback interface
   # when the system is booting.  Do not change this entry.
   ##
   127.0.0.1		localhost
   255.255.255.255	broadcasthost
   ::1			localhost

   52.49.93.188 example.com

.. caution::

    Remember to remove this entry after you :ref:`switched DNS <switch_dns>`.

3. HTTPS
--------

On SymfonyCloud, all environments support both HTTP and HTTPS.

If you don't bring your own certificates, SymfonyCloud provisions certificates
using `Let's Encrypt <https://letsencrypt.org/>`_. Note that this setup requires
that the :ref:`DNS switch <switch_dns>` step is completed first.

.. note::

    It can take a couple of minutes for Let's Encrypt to provision the initial
    certificates. This is expected, and implies that the first deployment after
    attaching a domain may take longer than usual.

Alternatively, you can :ref:`bring your own certificates <custom-ssl>`. This is
particularly useful when switching over to SymfonyCloud as this avoids any
interruption in HTTPS serving.

.. caution::

    When using wildcard domains, SymfonyCloud can not provision certificates on
    your behalf. You have to provision it on your end and upload them to
    SymfonyCloud.

.. _switch_dns:

4. Switch DNS
-------------

Configure your DNS provider to point the project domain(s) to SymfonyCloud.

The precise way to do so varies depending on your registrar, but nearly all
registrars should allow you to set a **CNAME**. Some might call it "Alias" or
similar alternate names:

1. Obtain the **CNAME** target by running ``symfony cname``;

2. Add a **CNAME** record for your desired domain (``www.example.com``) using
   the value obtained before.

.. note::

    Depending on your registrar and the TTL you set, it could take anywhere from
    15 minutes to 72 hours for the DNS change to fully propagate across the
    Internet.

APEX
~~~~

SymfonyCloud expects you to use a **CNAME** for all DNS records but **CNAME**
don't work with APEX domains.

Depending on your project constraints, several options are available:

Using a DNS provider with custom type records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Some DNS providers have found a way around the CNAME-on-Apex limitation by doing
their own internal lookup behind the scenes:

* CNAME Flattening at `CloudFlare <https://www.cloudflare.com>`_
* ANAME at `easyDNS <https://www.easydns.com>`_
* ANAME at `DNS Made Easy <https://www.dnsmadeeasy.com>`_
* ANAME at `Name.com <https://www.name.com>`_
* ALIAS at `DNSimple <https://dnsimple.com>`_

.. tip::

    We recommend that you check support for dynamic APEX domains before
    registering your domain name with a DNS Provider.

Using a DNS provider with APEX domain forwarding
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are willing to make the ``www.`` version of your project the canonical
version, some registrars or DNS providers may provide a domain redirect feature.
The following DNS providers are known to support both APEX forwarding and
advanced DNS configurations simultaneously:

* `Namecheap <https://www.namecheap.com/support/knowledgebase/article.aspx/385/2237/how-do-i-set-up-a-url-redirect-for-a-domain>`_

Using a www redirection service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If your preferred registrar/DNS provider doesn't support either custom records
or the APEX domain forwarding options above, the following free services both
allow blind redirects and allow you to use a **CNAME** record to SymfonyCloud
for ``www.example.com`` and an **A** record to their service at ``example.com``,
which will in turn send a redirect.

* `WWWizer <https://wwwizer.com>`_

Using A records
^^^^^^^^^^^^^^^

If you cannot use a DNS provider that supports aliases or a redirection service,
it is possible to use **A** records.

This process has a few limitations:

* Should SymfonyCloud ever need to change one of its IPs, your configuration
  will need to be manually updated (some requests will be lost in the meantime).

* Using an IP means that your requests will be directly pointing at a
  SymfonyCloud router, bypassing their load-balancing functionality. Should one
  of them go offline for maintenance (as happens periodically for upgrades),
  approximately 1/3 of requests will go to the offline router and be lost,
  making the project appear offline.

Using **A** records is **strongly discouraged** and should only be used as a
last resort. You can get SymfonyCloud public IPs via
``symfony account:ips --ingress``.

5. Bonuses
----------

The following steps are not required but good practices when going live with a
SymfonyCloud project.

Scaling
~~~~~~~

With SymfonyCloud, you get a set of resources according to the project monthly
plan. When those resources are being exhausted by a peak in the traffic, a
project can be scaled up in a matter of seconds with ``symfony project:scale``.
Once the peak of traffic is done, scale down with ``symfony project:scale
--down``. At the end of the billing period, the project invoice will include a
pro-ration for the period where the project was on a higher plan.

To ensure maximum performance, be sure to read our documentation about
:ref:`service sizing <service_size>`, :ref:`container sizing <config-size>`, and
:ref:`PHP sizing <php_size>`. Finally, you can leverage features from services
like :doc:`Blackfire <../integrations/blackfire>`.

Automatic snapshots and certificate renewal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. caution::

   You first need to set up an :doc:`API token </cookbooks/api_tokens>` and set
   it up in your project environment variables. Then you can configure the
   appropriate tasks.

It's **strongly** recommended that you set up automatic :doc:`snapshots
</cookbooks/snapshots>` and automatic :doc:`certificate </cookbooks/https>`
deployment cron tasks. The following snippet is generally sufficient but see the
links above for more details, and don't forget to tweak the cron schedules
listed to match your use case.

.. code-block:: yaml

   crons:
       snapshot:
           # Take a snapshot automatically every night at 3 am (UTC).
           spec: '0 3 * * *'
           cmd: |
               if [ "$SYMFONY_BRANCH" = master ]; then
                   croncape symfony env:snapshot:create --no-wait
               fi
       renewcert:
           # Force a redeploy at 4 am (UTC) on the 14th and 28th of every month.
           spec: '0 4 14,28 * *'
           cmd: |
               if [ "$SYMFONY_BRANCH" = master ]; then
                   croncape symfony env:redeploy --no-wait
               fi

Health checks
~~~~~~~~~~~~~

While not required, it is recommended that you set up health notifications to
let you know if your project is experiencing issues such as running low on disk
space. Notifications can be sent via email, Slack, or PagerDuty:

.. code-block:: terminal

   $ symfony integration:add
   Please select the type of integration you want to add:
   [0] bitbucket
   [1] github
   [2] gitlab
   [3] health.email
   [4] health.pagerduty
   [5] health.slack
   [6] hipchat
   [7] webhook
    > 3
   From (email): cloud@example.com
   Recipients: team@example.com
   Created integration efelccbyabjdi (type: health.email).
   ┌──────────────┬────────────────────┐
   │   Property   │       Value        │
   ├──────────────┼────────────────────┤
   │ from_address │ cloud@example.com  │
   │ id           │ efelccbyabjdi      │
   │ recipients   │ - team@example.com │
   │ type         │ health.email       │
   └──────────────┴────────────────────┘


Monitoring
~~~~~~~~~~

While not required nor integrated, we recommend you to set up external
monitoring.

CDN
~~~

While not required nor integrated, you can set up a CDN in front of your
project. CDNs let you cache public pages and assets at the edge making your
websites load faster.
