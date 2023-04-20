====================
 激活和注册 Pyarmor
====================

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor reg

Initial Registration
====================

First read :doc:`Pyarmor License <../licenses>` to purchase one Pyarmor License.

An activation file like :file:`pyarmor-regcode-xxxx.txt` will be sent to you by email. This file is used to initial registration.

At the first time to register Pyarmor, :option:`-p` (product name) should be set. If not set, this Pyarmor license is bind to ``TBD``, and could not be used for commercial product.

It need internet connection for intial registration.

For non-profits usage
---------------------

For internal use or any non-profits use, run this command::

    $ pyarmor reg pyarmor-regcode-xxxx.txt

For commercial usage
--------------------

Assume this license is used to protect your product ``Robot Studio``, initial registration by this command::

    $ pyarmor reg -p "Robot Studio" pyarmor-regcode-xxxx.txt

Pyarmor will show registration information and ask your confirmation. If everything is fine, type :kbd:`yes` and :kbd:`Enter` to continue.

If initial registration is successful, it prints final license information in the console. And a registration file named :file:`pyarmor-regfile-xxxx.zip` for this license is generated in the current path at the sametime. This file is used for next registration in other machines.

Activation file :file:`pyarmor-keycode-xxxx.txt` can be uses only 10 times, after that it doesn't work. So once initial registration is successful, using registration file :file:`pyarmor-regfile-xxxx.zip` for next registration.

Please keep this registration file carefully. If lost, Pyarmor is not responsible for keeping this license. In this case, if continue to use Pyarmor, needs purchase new one.

Once register successfully, product name can't be changed.

Product name is not decided
---------------------------

When product is in developing, and product name is not decide. Initial registration with product ``TBD``. For example::

    $ pyarmor reg -p "TBD" pyarmor-regcode-xxxx.txt

It can be changed once later, before product starts selling, the real name must be set by this command::

    $ pyarmor reg -p "Robot Studio" pyarmor-regcode-xxxx.txt

Registeration in other machines
===============================

Once initial registeration successfully, it generates registration file named :file:`pyarmor-regfile-xxxx.zip` at the same time.

Copy this file to other machines, then run the following command::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

Check the registration information::

    $ pyarmor -v

Registeration in Docker or CI pipeline
--------------------------------------

It's no problem to run Pyarmor in Docker or CI pipeline to obfuscate user's application. Register pyarmor with :file:`pyarmor-regfile-xxxx.zip` same as above. But It's not allowed to distribute pakcage pyarmor and any Pyarmor License to customer. And don't run too many build dockers.

Upgrade Pyarmor from prior to 8.0
=================================

Refer to :ref:`upgrade old license <upgrading old license>`

.. include:: ../_common_definitions.txt
