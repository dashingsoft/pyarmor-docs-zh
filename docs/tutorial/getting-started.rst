==========
 入门教程
==========

.. highlight:: console

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

Pyarmor 是什么
==============

|Pyarmor| 是一个用于加密和保护 |Python| 脚本的工具。它能够在运行时刻保护 |Python| 脚本代码不被泄露，设置加密后脚本的使用期限，绑定加密脚本到硬盘、网卡等硬件设备。

功能特点:

- **无缝替换**: 加密后的脚本依然是一个有效的 `.py` 文件，在大多数情况下可以直接替换原来的 `.py` 脚本，而不影响脚本的使用。
- **均衡加密**: 提供了丰富的加密选项来平衡安全性和性能，能够满足大多数应用对安全性和性能的要求。
- **不可逆加密**: 能够直接重命名源代码中的函数，类，方法，变量和参数。
- **转换成为 C 代码**: 能够把模块中部分函数转换成为 C 代码，然后使用高优化选项直接编译 C 代码为机器指令来保护 Python 函数
- **限制加密脚本的使用范围**: 可以绑定加密脚本到指定的设备或者设置加密脚本的有效期
- **Themida 保护**: 使用 Themida 保护加密脚本（仅 Windows 平台可用）

安装
====

Pyarmor_ 是一个发布在 PyPI_ 的 Python 包，最方便的方式就是直接使用命令 :command:`pip` 进行安装。

在 Linux 或者 Apple 平台，直接打开终端，然后执行下面的命令::

    $ pip install -U pyarmor

在 Windows 下面，使用 :kbd:`Win-r` 弹出命令输入框，输入 :command:`cmd` 之后回车打开控制台，在控制台输入安装命令:

.. code-block:: doscon

    C:\> pip install -U pyarmor

安装完成之后，输入命令 :command:`pyarmor --version` 并回车执行。如果安装成功，会显示出 Pyarmor 的版本信息。

不是所有的平台都被 Pyarmor 支持，所有支持的平台和架构请查看 :doc:`../reference/environments`

加密脚本
========

.. program:: pyarmor gen

下面是最简单的加密命令，用来加密一个脚本 :file:`foo.py`::

    $ pyarmor gen foo.py

子命令 ``gen`` 能够被替换成为 ``g`` 或者 ``generate``::

    $ pyarmor g foo.py
    $ pyarmor generate foo.py

This command generates an obfuscated script :file:`dist/foo.py`, which is a valid Python script, run it by Python interpreter::

    $ python dist/foo.py

Check all generated files in the default output path::

    $ ls dist/
    ...    foo.py
    ...    pyarmor_runtime_000000

There is an extra Python package :file:`pyarmor_runtime_000000`, which is required to run the obfuscated script.

Distributing the obfuscated script
----------------------------------

Only copy :file:`dist/foo.py` to another machine doesn't work, instead copy all the files in the :file:`dist/`.

Why? It's clear after checking the content of :file:`dist/foo.py`:

.. code-block:: python

    from pyarmor_runtime_000000 import __pyarmor__
    __pyarmor__(__name__, __file__, ...)

Actually the obfuscaetd script can be taken as normal Python script with dependent package :mod:`pyarmor_runtime_000000`, use it as it's not obfuscated.

.. important::

   Please run this obfuscated in the machine with same Python version and same platform, otherwise it doesn't work. Because :mod:`pyarmor_runtime_000000` has an :term:`extension module`, it's platform-dependent and bind to Python version.

.. note::

   DO NOT install Pyarmor in the :term:`Target Device`, Python interpreter could run the obfuscated scripts without Pyarmor.

Obfuscating one package
=======================

Now let's do a package. :option:`-O` is used to set output path :file:`dist2` different from the default::

    $ pyarmor gen -O dist2 src/mypkg

Check the output::

    $ ls dist2/
    ...    mypkg
    ...    pyarmor_runtime_000000

    $ ls dist2/mypkg/
    ...          __init__.py

All the obfuscated scripts in the :file:`dist2/mypkg`, test it::

    $ cd dist2/
    $ python -C 'import mypkg'

If there are sub-packages, using :option:`-r` to enable recursive mode::

    $ pyarmor gen -O dist2 -r src/mypkg

Distributing the obfuscated package
-----------------------------------

Also it works to copy the whole path :file:`dist2` to another machine. But it's not convience, the better way is using :option:`-i` to generate all the required files inside package path::

    $ pyarmor gen -O dist3 -r -i src/mypkg

Check the output::

    $ ls dist3/
    ...    mypkg

    $ ls dist3/mypkg/
    ...          __init__.py
    ...          pyarmor_runtime_000000

Now everything is in the package path :file:`dist3/mypkg`, just copy the whole path to any target machine.

.. note::

   Comparing current :file:`dist3/mypkg/__init__.py` with above section :file:`dist2/mypkg/__init__.py` to understand more about obfuscated scripts

Expiring obfuscated scripts
===========================

It's easy to set expire date for obfuscated scripts by :option:`-e`. For example, generate obfuscated script with the expire date to 30 days::

    $ pyarmor gen -O dist4 -e 30 foo.py

Run the obfuscated scripts :file:`dist4/foo.py` to verify it::

    $ python dist4/foo.py

It checks network time, make sure your machine is connected to internet.

Let's use another form to set past date ``2020-12-31``::

    $ pyarmor gen -O dist4 -e 2020-12-31 foo.py

Now :file:`dist4/foo.py` should not work::

    $ python dist4/foo.py

If expire date has a leading ``.``, it will check local time other than NTP_ server. For examples::

    $ pyarmor gen -O dist4 -e .30 foo.py
    $ pyarmor gen -O dist4 -e .2020-12-31 foo.py

For this form internet connection is not required in target machine.

Distributing the expired script is same as above, copy the whole directory :file:`dist4/` to target machine.

Binding obfuscated scripts to device
====================================

Suppose got target machine hardware informations::

    IPv4:                        128.16.4.10
    Enternet Addr:               00:16:3e:35:19:3d
    Hard Disk Serial Number:     HXS2000CN2A

Using :option:`-e` to bind hardware information to obfuscated scripts. For example, bind :file:`dist5/foo.py` to enternet address::

    $ pyarmor gen -O dist5 -b 00:16:3e:35:19:3d foo.py

So :file:`dist5/foo.py` only could run in target machine.

It's same to bind IPv4 and serial number of hard disk::

    $ pyarmor gen -O dist5 -b 128.16.4.10 foo.py
    $ pyarmor gen -O dist5 -b HXS2000CN2A foo.py

It's possible to combine some of them. For example::

    $ pyarmor gen -O dist5 -b "00:16:3e:35:19:3d HXS2000CN2A" foo.py

Only both enternet address and hard disk are matched machine could run this obfuscated script.

Distributing scripts bind to device is same as above, copy the whole directory :file:`dist5/` to target machine.

Packaging obfuscated scripts
============================

Remeber again, the obfuscated script is normal Python script, use it as it's not obfuscated.

Suppose package ``mypkg`` structure like this::

    projects/
    └── src/
        └── mypkg/
            ├── __init__.py
            ├── utils.py
            └── config.json

First make output path :file:`projects/dist6` for obfuscated package::

    $ cd projects
    $ mkdir dist6

Then copy package data files to output path::

    $ cp -a src/mypkg dist6/

Next obfuscate scripts to overwrite all the ``.py`` files in :file:`dist6/mypkg`::

    $ pyarmor gen -O dist6 -i src/mypkg

The final output::

    projects/
    ├── README.md
    └── src/
        └── mypkg/
            ├── __init__.py
            ├── utils.py
            └── config.json
    └── dist6/
        └── mypkg/
            ├── __init__.py
            ├── utils.py
            ├── config.json
            └── pyarmor_runtime_000000/__init__.py

Comparing with :file:`src/mypkg`, the only difference is :file:`dist6/mypkg` has an extra sub-package ``pyarmor_runtime_000000``. The last thing is packaging :file:`dist6/mypkg` as your prefer way.

New to Python packaging? Refer to `Python Packaging User Guide`_

.. _Python Packaging User Guide: https://packaging.python.org

Something need to know
======================

There is binary `extension module`_ :mod:`pyarmor_runtime` in extra sub-package ``pyarmor_runtime_000000``, here it's package content::

    $ ls dist6/mypkg/pyarmor_runtime_000000
    ...    __init__.py
    ...    pyarmor_runtime.so

Generally using binary extensions means the obfuscated scripts require :mod:`pyarmor_runtime` be created for different platforms, so they

* only works for platforms which provides pre-built binaries
* may not be compatible with different builds of CPython interpreter
* often will not work correctly with alternative interpreters such as PyPy, IronPython or Jython

For example, when obfuscating scripts by Python 3.8, they can't be run by Python 3.7, 3.9 etc.

Another disadvantage of relying on binary extensions is that alternative import mechanisms (such as the ability to import modules directly from zipfiles) often won't work for extension modules (as the dynamic loading mechanisms on most platforms can only load libraries from disk).

What to read next
=================

There is a complete :doc:`installation <installation>` guide that covers all the possibilities:

* install pyarmor by source
* call pyarmor from Python script
* clean uninstallation

Next is :doc:`obfuscation`. It covers

* using more option to obfuscate script and package
* using outer file to store runtime key
* localizing runtime error messages
* packing obfuscated scripts and protect system packages

And then :doc:`advanced`, some of them are not available in trial pyarmor

* 2 irreversible obfuscation: RFT mode, BCC mode :sup:`pyarmor-pro`
* Customization error handler
* plugin and hooks
* runtime error internationalization
* cross platform, multiple platforms and multiple Python version

Also you may be instersting in this guide :doc:`../how-to/security`

如何阅读本手册
==============

|Pyarmor| 有完备的文档系统，这里是帮助用户如何快速找到需要的相关内容

* :doc:`第一部分: 基础教程 <../part-1>` 适合第一次使用 Pyarmor 的用户，这里以实例的形式一步接着一步的说明了加密脚本和包的最常用的场景。也可以先看看 :doc:`getting-started`

* :doc:`第二部分: 应用实践 <../part-2>` 针对每一个特定的需求，说明在 Pyarmor 中应该如何去做，使用什么样的命令和选项去实现。阅读这部分内容需要对 Pyarmor 和 Python 都有一定的了解。

* :doc:`第三部分: 技术手册 <../part-3>` 从技术层的角度详细列出了所有的概念定义，命令手册，配置选项和错误信息代码。它适用于使用 Pyarmor 的高级用户，需要查找相关的参数和配置，了解这些配置项的可用值和不同值的作用和含义。

* :doc:`第四部分: 深入专题 <../part-4>` 这部分针对 Pyarmor 提供的功能，从如何实现的层面进行了详细的解释。阅读这部分内容需要完全掌握了 Pyarmor 使用到的主要概念，以及对 Python 脚本的执行过程有相当了解。它适用于需要对 Pyarmor 进行扩展和定制，以满足更高一层需求的用户。

* :doc:`第五部分: 许可模式和许可证类型 <../licenses>` 描述了 |Pyarmor| 的最终用户许可协议， |Pyarmor| 的许可模式，不同的许可类型，以及如何购买 |Pyarmor| 许可证。

想找某一个特定的关键字，搜一下 :ref:`总索引 <genindex>`, 或者浏览 :ref:`文档总目录 <mastertoc>`

还是没有找到？ 看一下 :ref:`如何在 Github 上提问 <asking questions>`.

.. include:: ../_common_definitions.txt
