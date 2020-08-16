Generic webhook integration
===========================

This integrations allows you to capture any push events on SymfonyCloud and
POST a JSON message describing the activity to the URL of your choice. You can
use this to further automate your workflow:

.. code-block:: terminal
   
   $ symfony integration:add --type=webhook

The URL will receive a POST message for every "Activity" that is triggered, and
the message will contain complete information about the entire state of the
project at that time. In practice most of the message can be ignored but is
available if needed. The most commonly used values are documented below.

It's also possible to set the integration to only send certain activity types,
or only activities on certain branches. The CLI will prompt you to specify which
ones to include or exclude. Leave at the default values to get all events on all
environments in a project.

Webhook payload
---------------

The following is an example of a webhook message. Specifically, this one was
created by a ``push`` event. You can find more details about the fields after
the payload content.

.. literalinclude:: ../_includes/webhook-payload.json
  :language: JSON

``id``
^^^^^^

A unique opaque value to identify the activity.

``project``
^^^^^^^^^^^

The Project ID for which the activity was triggered. Use this value if you want
to have multiple projects POST to the same URL.

``type``
^^^^^^^^

The ``type`` property specifies the event that happened. Its value is one of:

* ``project.modify.title``: The human-friendly title of the project
  has been changed.

* ``project.domain.create``: A new domain has been added to the project.
* ``project.domain.delete``: A domain associated with the project has been
   removed.
* ``project.domain.update``: A domain associated with the project has been
  updated, including modifying it's SSL certificate.

* ``environment.access.add``: A new user has been given access to the
  environment.
* ``environment.access.remove``: A user has been removed from the environment.

* ``environment.backup``: A user triggered a
  :doc:`snapshot </cookbooks/snapshots>`.
* ``environment.restore``: A user restored a
  :doc:`snapshot </cookbooks/snapshots>`.

* ``environment.push``: A user has deployed code to a branch, either existing or
  new.
* ``environment.activate``: A branch has been "activated", and an environment
  created for it.
* ``environment.initialize``: The main branch of the project has just been
  initialized with its first commit.
* ``environment.deactivate``: A branch has been "deactivated". The code is still
  there but the environment was destroyed.
* ``environment.synchronize``: An environment has had its data synchronized from
  its parent environment.
* ``environment.delete``: A branch was deleted.

* ``environment.variable.create``: A new variable has been created.
* ``environment.variable.delete``: A variable has been deleted.
* ``environment.variable.update``: A variable has been modified.

* ``environment.update.http_access``: HTTP access rules for an environment
  have been modified.
* ``environment.update.smtp``: Sending of emails has been enabled/disabled
  for an environment.
* ``environment.update.restrict_robots``: The block-all-robots feature has been
  enabled/disabled.

* ``environment.subscription.update``: The master environment has been resized
  because the subscription has changed. There are no content changes.

``environments``
^^^^^^^^^^^^^^^^

An array listing the environments that were involved in the activity. This is
usually a single-value array.

``result``
^^^^^^^^^^

Whether the activity was completed successfully or not. It should be ``success``
if all went as planned.

``created_at``, ``started_at``, ``completed_at``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These values are all timestamps in UTC. If you need only a point in time when
the action happened, use ``completed_at``.  You can also combine it with
``started_at`` to see how long the activity took.

``log``
^^^^^^^

A text description of the action that happened.  This is a human-friendly string
that may be displayed to a user but should not be parsed for data as its
structure is not guaranteed.

``payload.environment``
^^^^^^^^^^^^^^^^^^^^^^^

This block contains information about the environment itself, after the action
has taken place. The most notable properties of this key are:

* ``name`` (the name of the branch)
* ``machine_name`` (the name of the environment)
* ``head_commit`` (the Git commit ID that triggered the event)

``payload.user``
^^^^^^^^^^^^^^^^

The user that triggered the activity.

``deployment``
^^^^^^^^^^^^^^

This large block details all information about all services in the environment.
That includes the resulting configuration objects derived from
:doc:`/routes`, :doc:`/services/intro` and :doc:`/config`.

Most notably, the ``deployment.routes`` object's keys are all of the URLs made
available by the environment. Note that some will be redirects.  To find those
that are live URLs filter to those objects whose ``type`` property is
``upstream``.
