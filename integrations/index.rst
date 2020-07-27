Integrations
============

SymfonyCloud has integrations with several service providers and technologies.
Those integrations usually rely on webhooks those providers trigger on specific
events and allow SymfonyCloud to react to them. For example, the integration
with a Git provider can automatically create new environment when a developer
create a new branch and deploy the code on each push.

.. note::

   Only the project owner and project administrators can manage integrations.

Integrations are configured per-project and can be listed using the
``symfony integrations`` command:

.. code-block:: terminal

   $ symfony integrations
   ┌───────────────┬──────────────┬──────────────────────────────────────────────┐
   │      ID       │     Type     │                   Summary                    │
   ├───────────────┼──────────────┼──────────────────────────────────────────────┤
   │ xy46lhhuf6n3t │ health.email │ From:                                        │
   │               │              │ To:                                          │
   │               │              │  - #admins                                   │
   │ kcrujtzar34w2 │ github       │ Repository: team/team                        │
   │               │              │ Hook URL: https://cloud.gw.symfony.com/[...] │
   └───────────────┴──────────────┴──────────────────────────────────────────────┘

You can also show more details using ``symfony integration:get`` and the
integration ID:

.. code-block:: terminal

   $ symfony integration:get xy46lhhuf6n3t
   ┌──────────────┬───────────────┐
   │   Property   │     Value     │
   ├──────────────┼───────────────┤
   │ from_address │ <nil>         │
   │ id           │ xy46lhhuf6n3t │
   │ recipients   │ - '#admins'   │
   │ role         │               │
   │ type         │ health.email  │
   └──────────────┴───────────────┘

Available integrations
----------------------

 .. toctree::
    :maxdepth: 1
    :titlesonly:

    bitbucket
    blackfire
    github
    gitlab
    hipchat
    webhooks

Add a new integration
---------------------

Use the ``symfony integration:add`` command to configure a new integration. A
wizard will help to gather all the information required.

.. tip::

   Alternatively, you can use flags to provide some configuration if you know it
   ahead of time, you can use ``symfony integration:add --help`` to list every
   available options.

Depending on the integration type, you might have to use the provided
``Hook URL`` on the provider side.

.. note::

   **Limitation:** Updating an integration is planned but not possible at the
   moment.

Delete an integration
---------------------

Integrations can be deleted at any time using the ``symfony integration:delete``
command.