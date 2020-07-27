Notifications
=============

SymfonyCloud can trigger notifications when various events happen on a project,
in any environment.

Available notifications
-----------------------

.. note::

   At this time, only low disk space warning notification are available, but
   others may be added in the future.

Low-disk warning
^^^^^^^^^^^^^^^^

If any integration is configured, SymfonyCloud will monitor disk space usage on
all applications and services in your cluster.

* If and when available disk space drops below 20%, a warning notification is
  generated.
* If and when available disk space drops below 10%, a critical notification is
  generated.
* If and when available disk space goes back above 20% after previously having
  been lower, an all-clear notification is generated.

.. tip::

   Notifications are generated every 5 minutes, so there may be a brief delay
   between when the threshold is crossed and when the notification is actually
   triggered.

Configuring notifications
-------------------------

Health notifications can be set up via the CLI through a number of different
channels each one detailed bellow.

Email notifications
^^^^^^^^^^^^^^^^^^^

A notification can trigger an email to be sent, from an address of your choosing
to one or more addresses of your choosing.

To do so, register a ``health.email`` integration as follows:

.. code-block:: terminal

   $ symfony integration:add --type=health.email \
           --from-address=you@example.com \
           --recipients=them@example.com \
           --recipients=others@example.com


The ``from-address`` is whatever address you want the email to appear to be
from. You must specify one or more ``recipients``, each as its own flag. It is
completely fine to use the same address for both ``from-address`` and
``recipients``.

.. tip::

   You can use ``#admins`` as a value for ``recipients``, this will notify every
   project administrator, removing the requirement for you to update the list
   when users changes. By default, every new project come with a
   ``health.email`` integration setup with this value.

.. caution::

   When using a custom ``from-address``, depending on the configuration of the
   recipient mail server (including SPF and DKIM DNS entries), the email can be
   marked as spam or lost.

Slack notifications
^^^^^^^^^^^^^^^^^^^

A notification can trigger a message to be sent as a Slack bot. First, create a
new custom `bot user <https://api.slack.com/bot-users>`_ for your Slack group
and configure the channels you wish it to live in. Note the API Token provided
by Slack.

Then register the Slack bot with SymfonyCloud using a ``health.slack``
integration:

.. code-block:: terminal

   $ symfony integration:add --type=health.slack --channel='#channelname'

That will trigger the corresponding bot to post a notification to the
``#channelname`` channel in your Slack group.

PagerDuty notifications
^^^^^^^^^^^^^^^^^^^^^^^

A notification can trigger a message to be sent via PagerDuty. First, create a
new `PagerDuty integration <https://support.pagerduty.com/docs/services-and-integrations>`_
that uses the Events API v2. Copy the "Integration Key" as known as the
"routing key" for the integration.

Now register a ``health.pagerduty`` integration:

.. code-block:: terminal

   $ symfony integration:add --type=health.pagerduty --routing-key=YOUR_ROUTING_KEY

Any notification will now trigger an alert in PagerDuty.
