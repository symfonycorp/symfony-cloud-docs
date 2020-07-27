Bitbucket
=========

The `Bitbucket <https://bitbucket.com/>`_  integration allows you to manage your
SymfonyCloud environments directly from your Bitbucket repository.

Supported:

* Create a new environment when creating a branch or opening a pull request on
  Bitbucket.
* Rebuild the environment when pushing new code to Bitbucket.
* Delete the environment when merging a pull request.

Setup
-----

.. caution::

   You must be an administrator of the Bitbucket repository to be able to setup
   this integration.

1. Setup an OAuth consumer
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. tip::

   This step is only required once and you can reuse the same consumer for
   several projects.

To integrate your Bitbucket repositories with SymfonyCloud you will have to
create an `OAuth consumer <https://confluence.atlassian.com/bitbucket/oauth-on-bitbucket-cloud-238027431.html>`_
for your Workspace.

#. Go to your Bitbucket Workspace and click ``Settings``.
#. Under ``APPS AND FEATURES`` click ``OAuth Consumers``.
#. Click the ``Add consumer`` button.
#. Fill out the information for the consumer. In order for the integration to
   work correctly, it's required that you include:

   * **Name**: Give the consumer a recognizable name, like
     ``SymfonyCloud integration``.
   * **Callback URL**: The URL users will be redirected to after access
     authorization. It is sufficient to set this value to ``http://localhost``.
   * **Set as a private consumer**: At the bottom of the ``Details`` section,
     select the ``This is a private consumer`` checkbox.
   * **Permissions**: Sets the integration permissions for SymfonyCloud as
     follows. These permissions will allow SymfonyCloud to mirror actions from
     the Bitbucket repositories.

     * **Account** - Email, Read
     * **Repositories** - Read, Write
     * **Pull requests** - Read, Write

#. After you have completed the form, click ``Save``.
#. Once saved, you will see your consumer listed in the ``OAuth consumers``
   section. Open that item, it will expose two values that you will need
   to complete the integration using the CLI: ``Key`` and ``Secret``.

2. Enable the integration
^^^^^^^^^^^^^^^^^^^^^^^^^

Enable the Bitbucket integration as follows:

.. code-block:: terminal

   $ symfony integration:add --type=bitbucket --repository=MY-NAMESPACE/MY-PROJECTNAME

Required configuration options:

* ``key``: The Bitbucket OAuth consumer key
* ``secret``: The Bitbucket OAuth consumer secret

Optional parameters:

* ``--build-pull-requests``: Track and deploy pull-requests (``true`` by
  default)
* ``--fetch-branches``: Track and deploy branches (``true`` by default)
* ``--prune-branches``: Delete branches that do not exist in the remote
  BitBucket repository (``true`` by default)
* ``--resync-pull-requests``: Re-sync pull request environment data on every
  build (``true`` by default)

3. Finish
^^^^^^^^^

Connect your Bitbucket repository to SymfonyCloud:

#. Copy the ``Hook URL`` shown in the summary table.
#. Go to the repository page on Bitbucket, click on the ``Repository Settings``
   item, choose ``Webhooks`` and click ``Add webhook``
#. Paste the ``Hook URL``, choose the following triggers, and click ``Add
   webhook``:

   * "Repository"

     * "Push"
     * "Updated"

   * "Pull Request"

     * "Created"
     * "Updated"
     * "Merged"
     * "Declined"

You can now start pushing code, creating new branches or opening pull requests
directly on your Bitbucket repository. You will see environments get
automatically created and updated on the SymfonyCloud side.

Types of environments
---------------------

Environments based on Bitbucket **pull requests** will have the correct 'parent'
environment on SymfonyCloud and will be activated automatically with a copy of
the parent's data.

However, environments based on (non-pull-request) **branches** cannot have
parents; they will inherit directly from the production environment and start
inactive by default.
