Frequently Asked Questions
==========================

.. note::

   For questions related to billing, please head to the :doc:`billing` section.

What is accounted in the storage quota?
---------------------------------------

Your storage quota (as shown by ``symfony project:info``) grants a maximum
storage capacity per environment. For example, the default project storage quota
of 5GB grants 5GB for each environment created.

This quota is shared by your :doc:`services </services/intro>` and
:doc:`applications </config>`. Each one defines its own quota using the ``disk``
configuration key. The sum of these quotas can not be bigger than your global
project quota, in such case, an error message will be shown during ``symfony
deploy``.

The space used to store your source code, your application container images, and
the temporary files (``/tmp``) does not account in the storage quota.

How much storage is available in ``/tmp``?
------------------------------------------

Each container comes with 4GB of temporary storage available at ``/tmp``.

How to rename a project?
------------------------

You can rename your project using ``symfony project:rename``.
