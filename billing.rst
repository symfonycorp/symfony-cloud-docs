Billing FAQ
===========

.. note::

   Official prices are available on `SymfonyCloud pricing page`_.

When and how is my project billed?
----------------------------------

Your project is billed monthly based on the subscription creation date (when the
project has been created) or based on the trial end.

Billing occurs at the beginning of each period, when the invoice is issued you
are paying for the upcoming period. For example, if your project is created on
July 2nd without trial, an invoice is issued for the selected plan for the
period from July 2nd to August 2nd.

If you change your plan or addons during the billing period, `prorations`_ are
going to be applied to your next invoice. If your project has important pending
items, an intermediary invoice can be issued before the beginning of the next
period.

.. _prorations:

How are managed prorations?
---------------------------

SymfonyCloud relies on Stripe to compute prorations down to the second. Stripe
computes the remaining time until the end of the current billing period, add a
negative item to the invoice for the previous monthly amount prorated to the
remaining time and add another item for the new monthly amount also prorated.
The next invoice will include the billing for the upcoming billing period, plus
the proration lines for the previous billing period.

.. note::

   Switching plan back and forth will generate several of these proration lines.

For example, upgrading from the development plan to the Standard production plan
on the 31st for a project billing on the 2nd will generate the following invoice
lines:

=========== =================================================== ======== ========== ======
Reference   Designation                                         Quantity Unit price Total
=========== =================================================== ======== ========== ======
SC_DEV       | Unused time on 1 × SymfonyCloud Development plan prorated  €10.00    -€0.70
             | From July 31, 2020 to August 2, 2020
SC_STANDARD  | Remaining time on 1 × SymfonyCloud Standard plan prorated  €50.00     €3.49
             | From July 31, 2020 to August 2, 2020
SC_STANDARD  | 1 × SymfonyCloud Standard plan
             | From August 2, 2020 to September 2, 2020         1         €50.00    €50.00
----------- --------------------------------------------------- -------- ---------- ------
\           **Total before taxes**                                                  €52.79
=========== =================================================== ======== ========== ======

Am I gonna be charged after trial?
----------------------------------

SymfonyCloud billing is opt-in based. You will be billed only if you explicitely
want to keep your project around after trial and accept billing using ``symfony
project:billing:accept`` or if you move to a Production plan using ``symfony
project:scale``.

If you do not upscale your project or accept billing, you project will be
automatically suspended at the end of your trial. If you do no pay in the days
following its suspension, your project will be automatically deleted.

What is the difference between development and production plans?
----------------------------------------------------------------

The development plan is intended to let you test and prepare your project before
going live. You do not have dedicated resources and can not attach production
domains to your main environment.

When a project is attached to a production plan, the Main environment runs on
different hosts with dedicated ressources and can serve traffic to custom
production domain names.

How do I change my project plan?
--------------------------------

To upscale your project one step up (from Standard to Medium for example), use
``symfony project:scale``. To downscale one step down, use ``symfony
project:scale --down``.

You can also scale directly to a specific plan using ``symfony project:scale
<plan>`` (replace ``plan`` with the plan name from `SymfonyCloud pricing
page`_).

How do I add more storage to my project?
----------------------------------------

Your subscription comes with 5GB of persistent storage per environment. Adding
more storage is made per increment of 5GB and is billed per increment and per
environment according to prices shown on `SymfonyCloud pricing page`_.

Add more storage using ``symfony project:edit storage X`` (replacing ``X`` by
the desired storage amount in Gigabytes).

How do I add more environments to my project?
---------------------------------------------

Your subscription comes with 1 main and 3 non-production and active
environments. You can add more non-production environments by increment of 3
billed per increment according to prices shown on `SymfonyCloud pricing page`_.

Add more environments using ``symfony project:edit environments X`` (replacing
``X`` by the number of environments).

How do I cancel my project?
---------------------------

You can cancel your SymfonyCloud subscription at any time using ``symfony
project:delete``. You will be prompted if you want to delete your project
immediately or automatically at the end of your current billing period.

How can I change my billing details?
------------------------------------

At the moment it is not possible to edit your billing details (name, address,
country, etc) yourself. If you need to change them, please file a ticket at
`SymfonyCloud Support`_.

You can change your payment method information at any time using ``symfony
project:billing:update-card``.

What happens in case of payment failure?
----------------------------------------

Because SymfonyCloud hosts your (paid) projects and holds potentially important
data, we do not cancel projects right away on payment failures.

In the eventuality that your project experiences a payment failure, we retry
charging your payment method several times over 10 days with an interval of 1 to
7 days between each attempt and notify you of the failure each time. If fixing
the failure requires you to change your payment method information please use
the ``symfony project:billing:update-card`` command.

If payment is still failed after several tries, you project will be
automatically suspended. You can then manually retry payment and unsuspend your
project by using the  ``symfony project:billing:unsuspend`` command.

If payment is not recovered by the end of the latest billing period, your
suscription will be cancelled and your project will be automatically deleted
about 30 days after this cancellation. If you wish to recover your project
during this grace period please contact `SymfonyCloud Support`_.

How can I change a project owner?
---------------------------------

Changing a project owner is possible but requires manual intervention from our
staff, please contact us on `SymfonyCloud Support`_.

If the current project data can be discarded, a simpler solution might be to
create a new project. As such the procedure is as follows:

#. Create a new project with the new owner account and note the created project
   ID: ``symfony project:create --no-link``
#. Invite the former project owner to the new project: ``symfony -p <ID>
   user:add <email>``
#. Link the current working directory to the new project: ``symfony link --force
   <ID>``
#. Deploy to the new project: ``symfony deploy``

.. note::

    Don't forget to setup environment variables if required.

.. caution::

    Don't forget to delete the former project once the migration is completed.

.. _SymfonyCloud Support: https://symfony.com/cloud/support
.. _SymfonyCloud pricing page: https://symfony.com/cloud/pricing
