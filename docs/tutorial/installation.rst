==============
 Installation
==============

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. _install-pypi:

Installation from PyPI
======================

Pyarmor_ packages are published on the PyPI_. The preferred tool for installing
packages from PyPI_ is :command:`pip`. This tool is provided with all modern
versions of Python.

On Linux or MacOS, you should open your terminal and run the following command::

    $ pip install -U pyarmor

On Windows, you should open Command Prompt (:kbd:`Win-r` and type
:command:`cmd`) and run the same command:

.. code-block:: doscon

    C:\> pip install -U pyarmor

After installation, type :command:`pyarmor --version` on the command prompt. If
everything worked fine, you will see the version number for the Pyarmor_ package
you just installed.

Installation from PyPI_ also allows you to install the latest development
release.  You will not generally need (or want) to do this, but it can be useful
if you see a possible bug in the latest stable release.  To do this, use the
``--pre`` flag::

    $ pip install -U --pre pyarmor

If you need generate obfuscated scripts to run in other platforms, install
:mod:`pyarmor.runtime`::

    $ pip install pyarmor.runtime

Installed command
-----------------

* :program:`pyarmor` is the main command to do everything. See :doc:`../reference/man`.

Start Pyarmor by Python interpreter
-----------------------------------

:program:`pyarmor` is same as the following command::

    $ python -m pyarmor.cli

Using virtual environments
==========================

When installing Pyarmor_ using :command:`pip`, use *virtual environments* which
could isolate the installed packages from the system packages, thus removing the
need to use administrator privileges.  To create a virtual environment in the
``.venv`` directory, use the following command::

    $ python -m venv .venv

You can read more about them in the `Python Packaging User Guide`_.

.. _Python Packaging User Guide: https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/#creating-a-virtual-environment

Installation from source
========================

You can install Pyarmor_ directly from a clone of the `Git repository`__.  This
can be done either by cloning the repo and installing from the local clone, on
simply installing directly via :command:`git`::

    $ git clone https://github.com/dashingsoft/pyarmor
    $ cd pyarmor
    $ pip install .

You can also download a snapshot of the Git repo in either `tar.gz`__ or
`zip`__ format.  Once downloaded and extracted, these can be installed with
:command:`pip` as above.

__ https://github.com/dashingsoft/pyarmor
__ https://github.com/dashingsoft/pyarmor/archive/master.tar.gz
__ https://github.com/dashingsoft/pyarmor/archive/master.zip


Run Pyarmor from Python script
==============================

Create a script :file:`tool.py`, pass arguments by yourself

.. code-block:: python

    from pyarmor.cli.__main__ import main_entry

    args = ['gen', 'foo.py']
    main(args)

Run it by Python interpreter::

    $ python tool.py

Clean uninstallation
====================

Run the following commands to make a clean uninstallation::

    $ pip uninstall pyarmor
    $ rm -rf ~/.pyarmor
    $ rm -rf ./.pyarmor

.. note::

   The path ``~`` may be different when logging by different
   user. ``$HOME`` is home path of current logon user, check the
   environment variable ``HOME`` to get the real path.

.. highlight:: default

.. Most Windows users do not have Python installed by default, so we begin with
   the installation of Python itself.  To check if you already have Python
   installed, open the *Command Prompt* (:kbd:`Win-r` and type :command:`cmd`).
   Once the command prompt is open, type :command:`python --version` and press
   Enter.  If Python is installed, you will see the version of Python printed to
   the screen.  If you do not have Python installed, refer to the `Hitchhikers
   Guide to Python's`__ Python on Windows installation guides. You must install
   `Python 3`__.

   Once Python is installed, you can install Sphinx using :command:`pip`.  Refer
   to the :ref:`pip installation instructions <install-pypi>` below for more
   information.

   __ https://docs.python-guide.org/
   __ https://docs.python-guide.org/starting/install3/win/


.. include:: ../_common_definitions.txt
