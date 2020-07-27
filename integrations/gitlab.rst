GitLab
======

The `GitLab <https://gitlab.com>`_ integration allows you to manage your
SymfonyCloud environments directly from your GitLab repository.

Features supported:

* Create a new environment when creating a branch or opening a pull request on
  GitLab.
* Rebuild the environment when pushing new code to GitLab.
* Delete the environment when merging a pull request.

Setup
-----

1. Generate a token
^^^^^^^^^^^^^^^^^^^

To integrate your SymfonyCloud project with an existing GitLab repository, you
first need to `generate a token on your GitLab user profile <https://gitlab.com/profile/personal_access_tokens>`_.

Fill the ``Name`` field (for example with "SymfonyCloud Integration") and
optionally set an expiration time.

Give it a description and then ensure the token has the following scopes:

 * ``api`` - Access your API
 * ``read_user`` - Read user information
 * ``read_repository`` - Read repositories

Copy the token and make a note of it (temporarily).

.. note::

   For the integration to work, your GitLab user needs to have permission to
   push code to the repository.

2. Enable the integration
^^^^^^^^^^^^^^^^^^^^^^^^^

Enable the GitLab integration as follows:

.. code-block:: terminal

   $ symfony integration:add --type=gitlab --gitlab-project=MY-NAMESPACE/MY-PROJECTNAME


.. tip::

   If your project is hosted on a private Gitlab instance you can use the
   ``--base-url`` option to point it to the URL for your instance. Do **not**
   append your namespace and project name.

Optional parameters:

* ``--build-merge-requests``: Track and deploy merge-requests (``true`` by
  default)
* ``--merge-requests-clone-parent-data`` : should merge requests clone the data
  from the parent environment (``true`` by default)
* ``--fetch-branches``: Track and deploy branches (``true`` by default)
* ``--prune-branches``: Delete branches that do not exist in the remote GitLab
  repository (``true`` by default)

3. Add the webhook
^^^^^^^^^^^^^^^^^^

When successful, the previous command outputs the configuration of the
integration.

#. Copy the ``Hook URL`` shown in the summary table.
#. Go to your GitLab repository and click ``Settings`` then ``Integrations``.
#. Paste the ``Hook URL``.
#. On the Triggers section choose ``Push events``, ``Tag push events`` and
   ``Merge Request events``.
#. Click on ``Add webhook``.

You can now start pushing code, creating new branches or opening merge requests
directly on your GitLab repository. You will see environments get automatically
created and updated on the SymfonyCloud side.

Types of environments
---------------------

Environments based on GitLab **merge requests** will have the correct 'parent'
environment on SymfonyCloud and they will be activated automatically with a copy
of the parent's data (unless the option ``merge-requests-clone-parent-data`` is
set to ``false``).

However, environments based on (non-merge-request) **branches** cannot have
parents; they will inherit directly from the production environment and start
inactive by default.
