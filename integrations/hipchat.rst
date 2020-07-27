HipChat
=======

The `HipChat <https://hipchat.com>`_  integration allows you to send
notifications about your SymfonyCloud activity directly to HipChat.

Setup
-----

1. Find the HipChat ROOM-ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* In the HipChat web administrative UI, go to ``Admin`` > ``Rooms`` and click on
  the room to link notifications.
* Note down the "APP ID" listed in the ``Room Details`` on the Room's 'Summary'
  page (you can also find the ID from the URL).

2. Generate a room-specific HIPCHAT-TOKEN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Click on the Room's 'Tokens' page in the sidebar.
* In the ``Create New Token`` section specify ``SymfonyCloud`` as the token's
  label and click "Create" button.
* Note down the Token value.

3. Enable the integration
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: terminal

   $ symfony integration:add --type=hipchat --room=ROOM-ID --token=HIPCHAT-TOKEN

There are a number of optional parameters who's **default values** are:

* ``--events=*`` (All Events)
* ``--environments=*`` (All Environments)
* ``--excluded-environments=`` (Empty)
* ``--states=complete`` (Complete state only)

You're given a chance to customize these parameters in an interactive shell
prompt, or you may **override the defaults on the command line**:

* ``--states=pending,in_progress,complete`` (All states)
