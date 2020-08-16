GitHub
======

The `GitHub <https://github.com>`_ integration allows you to manage your
SymfonyCloud environments directly from your GitHub repository.

Features supported:

* Create a new environment when creating a branch or opening a pull request on
  GitHub.
* Rebuild the environment when pushing new code to GitHub.
* Delete the environment when merging a pull request.

Setup
-----

1. Generate a token
^^^^^^^^^^^^^^^^^^^

To integrate your project with an existing GitHub repository, you first need to
`generate a new token <https://github.com/settings/tokens/new?description=SymfonyCloud&scopes=public_repo,repo,read:org>`_.

Give it a description and then ensure the token has the following scopes:

* To integrate with public repositories: ``public_repo``
* To integrate with your own private repositories: ``repo``
* To integrate with your organization's private repositories: ``repo``
  and ``read:org``

Copy the token and make a note of it (temporarily).

.. note::

   For the integration to work, your GitHub user needs to have permission to
   push code to the repository.

2. Enable the integration
^^^^^^^^^^^^^^^^^^^^^^^^^

Enable the GitHub integration as follows:

.. code-block:: terminal

   $ symfony integration:add --type=github --repository=USER/REPOSITORY

where

* ``USER`` is your GitHub username or organization
* ``REPOSITORY`` is the name of the repository in GitHub (not the git address)

Optional parameters:

* ``--fetch-branches``: Track and deploy branches (``true`` by default)
* ``--prune-branches``: Delete branches that do not exist in the remote GitHub
  repository (``true`` by default)
* ``--build-pull-requests``: Track and deploy pull-requests (``true`` by
  default)
* ``--build-pull-requests-post-merge``: ``false`` to have SymfonyCloud build
  the branch specified in a PR. ``true`` to build the result of merging the PR
  (``false`` by default)
* ``--pull-requests-clone-parent-data``: Set to ``false`` to disable cloning of
  parent environment data when creating a PR environment, so each PR
  environment starts with no data. (``true`` by default)

3. Add the webhook
^^^^^^^^^^^^^^^^^^

When successful, the previous command outputs the configuration for the
integration.

#. Copy the hook URL shown in the summary table.
#. Go to your GitHub repository, click ``Settings``, select the
   ``Webhooks and Services`` tab, and click ``Add webhook``.
#. Paste the hook URL, choose ``application/json`` for the content type, choose
   ``Send me everything`` for the events you want to receive, and click
   ``Add webhook``.

You can now start pushing code, creating new branches or opening pull requests
directly on your GitHub repository.

Types of environments
---------------------

Environments based on GitHub **pull requests** will have the correct "parent"
environment on SymfonyCloud and they will be activated automatically with a copy
of the parent's data (unless the option ``--pull-requests-clone-parent-data`` is
set to ``false``).

However, environments based on (non-pull-request) **branches** cannot have
parents; they will inherit directly from the production environment and start
inactive by default.
