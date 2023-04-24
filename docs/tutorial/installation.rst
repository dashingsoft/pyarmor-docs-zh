==============
 完整安装教程
==============

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. _install-pypi:

从 PyPI 直接安装
================

Pyarmor_ 发布在 PyPI_ 上面，最方便的方式就是使用命令 :command:`pip` 直接安装。

在 Linux 和 MacOS，直接打开命令终端并运行下面的命令::

    $ pip install -U pyarmor

在 Windows 环境下，需要使用 :kbd:`Win-r` 打开命令输入框，然后输入 :command:`cmd` 打开命令窗口，并运行下面的命令:

.. code-block:: doscon

    C:\> pip install -U pyarmor

安装完成之后，输入命令 :command:`pyarmor --version` 并回车。如果安装成功，会显示安装的 Pyarmor 的版本信息。

如果需要跨平台发布加密脚本，还需要安装另外一个包 :mod:`pyarmor.cli.runtime`::

    $ pip install pyarmor.cli.runtime

并不是所有的平台都支持 Pyarmor，所有支持的运行平台请查看 :doc:`../reference/environments`

安装的命令
----------

* :program:`pyarmor` 是最重要的一个，所有的工作基本都由它来完成，详细使用方法请参考 :doc:`../reference/man`
* :program:`pyarmor-7` 是为了和老版本兼容的命令，它等价于 Pyarmor 7.x 的修正版本。

使用 Python 解释器直接运行 Pyarmor
----------------------------------

:program:`pyarmor` 等价于下面的命令::

    $ python -m pyarmor.cli

从 Github 上面进行安装
======================

也可以直接从 `Pyarmor Github`__ 安装 Pyarmor。下载库到本地，然后使用 pip 进行安装::

    $ git clone https://github.com/dashingsoft/pyarmor
    $ cd pyarmor
    $ pip install .

你可以直接下载一个库的压缩文件 `tar.gz`__ 或者 `zip`__ ，解压之后在使用 pip 进行安装。

__ https://github.com/dashingsoft/pyarmor
__ https://github.com/dashingsoft/pyarmor/archive/master.tar.gz
__ https://github.com/dashingsoft/pyarmor/archive/master.zip


在 Python 脚本中调用 Pyarmor
============================

首先创建一个任意的脚本，例如 :file:`tool.py`

.. code-block:: python

    from pyarmor.cli.__main__ import main_entry

    args = ['gen', 'foo.py']
    main(args)

然后运行这个脚本::

    $ python tool.py

以上只是一个示例说明，具体使用的加密选项和参数可以通过各种方式进行传递。

完全卸载
========

使用下面的命令可以完全卸载 Pyarmor::

    $ pip uninstall pyarmor
    $ pip uninstall pyarmor.cli.core
    $ pip uninstall pyarmor.cli.runtime
    $ rm -rf ~/.pyarmor
    $ rm -rf ./.pyarmor

.. include:: ../_common_definitions.txt
