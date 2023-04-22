.. highlight:: none

====================
 构建加密脚本的环境
====================

这里列出了和命令 :command:`pyarmor` 相关的所有一切。

首先命令 :command:`pyarmor` 运行在 :term:`开发机器` 上面，使用 `supported Python versions`_ 运行在这些 `supported platforms`_

命令行选项， `配置文件选项`_ ， `加密插件和运行插件`_ 和一些环境变量影响和改变着命令 :command:`pyarmor` 的行为。

所有的命令行选项和相关的环境变量在 :doc:`man` 中有详细描述。

.. _supported python versions:

支持的 Python 版本
------------------

.. table:: 表-1. 不同功能支持的 Python 版本表
   :widths: auto

   ===================  =====  =========  =========  ==========  ======  =======  ==============
   Python               2.7    3.0~3.4    3.5~3.6    3.7~3.10    3.11    3.12+       备注
   ===================  =====  =========  =========  ==========  ======  =======  ==============
   RFT 模式              No       No         No          Y         Y       N/y      [#]_
   BCC 模式              No       No         No          Y         Y       N/y
   pyarmor 8 基础功能     No       No         No          Y         Y       N/y
   pyarmor-7             Y        Y          Y           Y         No      No
   ===================  =====  =========  =========  ==========  ======  =======  ==============

.. _supported platforms:

支持的平台
----------

.. table:: Table-2. 不同功能支持的平台列表
   :widths: auto

   ===================  ============  ========  =======  ============  =========  =======  =======
   OS                     Windows           Apple                    Linux
   -------------------  ------------  -----------------  -----------------------------------------
   Arch                  x86/x86_64    x86_64    arm64    x86/x86_64    aarch64    armv7    armv6
   ===================  ============  ========  =======  ============  =========  =======  =======
   Themida 保护              Y           No        No         No          No       No        No
   RFT 模式                  Y           Y         Y          Y           Y        Y         N/y
   BCC 模式                  Y           Y         Y          Y           Y        N/y       No
   pyarmor 8 基础功能         Y           Y         Y          Y           Y        Y         Y
   pyarmor-7 [#]_           Y           Y         Y          Y           Y        Y         Y
   ===================  ============  ========  =======  ============  =========  =======  =======

.. rubric:: 注释

.. [#] ``N/y`` 意味着现在还不支持，但是将来会支持
.. [#] pyarmor-7 支持更多的平台，参考 `Pyarmor 7.x 文档`__.

.. important::

   pyarmor-7 是 Pyarmor 7.x 的问题修正版本，只支持老版本的许可证。在新版本的许可证下面使用可能会报错 ``HTTP 401 error``

__ https://pyarmor.readthedocs.io/zh/v7.x/platforms.html

配置文件选项
------------

有三种类型的配置文件

* :term:`全局配置` 文件，一个 ``.ini`` 格式的文件 :file:`~/.pyarmor/config/global`
* :term:`本地配置` 文件，一个 ``.ini`` 格式的文件 :file:`.pyarmor/config`
* :term:`模块私有配置` 每一个模块可以有自己的私有配置，存放在 :term:`本地配置` 的目录下面

使用命令 :ref:`pyarmor cfg` 来查看和设置配置文件选项。

加密插件和运行插件
------------------

.. versionadded:: 8.x
                  这个功能尚未实现


====================
 运行加密脚本的环境
====================

加密脚本运行在 :term:`客户设备` 上面，支持的平台和 Python 版本和 `构建加密脚本的环境`_ 是一样的。

部分系统模块 :mod:`sys` 的属性和环境变量会被加密脚本使用来决定一些运行设置。

:attr:`sys._MEIPASS`

      借用 PyInstaller_ 的设置，加密脚本会在这里指定的路径下面搜索 :term:`外部密钥`

:attr:`sys._PARLANG`

      用来设置运行时刻使用的语言设置。

      如果这个属性存在，那么环境变量 :envvar:`LANG` 会被忽略

.. envvar:: LANG

      操作系统环境变量，加密脚本读取它来决定运行时刻的语言设置

.. envvar:: PYARMOR_LANG

      用来设置运行时刻使用的语言设置。

      如果这个变量存在，那么 :envvar:`LANG` 和 :attr:`sys._PARLANG` 都会被忽略

.. envvar:: PYARMOR_RKEY

      设置搜索 :term:`外部密钥` 的路径

.. _specialized builtin functions:

加密脚本内置函数
----------------

.. versionadded:: 8.x
                  这个功能尚未实现

有两个特殊的函数无需导入，可以直接在加密脚本中使用，通常它们一般用在 :term:`运行插件` 中。

.. function:: __pyarmor__(arg, kwarg, name, flag)

   `name` must be byte string ``b'hdinfo'`` or ``b'keyinfo'``

   `flag` must be ``1``

   When `name` is ``b'hdinfo'``, call it to get hardware information.

   `arg` could be

   - 0: get the serial number of first harddisk
   - 1: get mac address of first network card
   - 2: get ipv4 address of first network card
   - 3: get target machine name

   For example,

   .. code-block:: python

         __pyarmor__(0, None, b'hdinfo', 1)
         __pyarmor__(1, None, b'hdinfo', 1)

   In Linux, `kwarg` is used to get named network card or named harddisk. For example:

   .. code-block:: python

         __pyarmor__(0, name="/dev/vda2", b'hdinfo', 1)
         __pyarmor__(1, name="eth2", b'hdinfo', 1)

   In Windows, `kwarg` is used to get all network cards and harddisks. For example:

   .. code-block:: python

         __pyarmor__(0, name="/0", b'hdinfo', 1)    # First disk
         __pyarmor__(0, name="/1", b'hdinfo', 1)    # Second disk

         __pyarmor__(1, name="*", b'hdinfo', 1)
         __pyarmor__(1, name="*", b'hdinfo', 1)


   When `name` is ``b'keyinfo'``, call it to query user data in the runtime key.

   For example,

   .. code-block:: python

         __pyarmor__(None, None, b'keyinfo', 1)   # return user data (bytes)
         __pyarmor__(1, None, b'keyinfo', 1)      # return expired date (int)

   Raise :exc:`RuntimeError` if something is wrong.

.. function:: __assert_armored__(arg)

   `arg` is a module or callable object, if `arg` is obfuscated, it return `arg` self, otherwise, raise protection error. For example

.. code-block:: python

    m = __import__('abc')
    __assert_armored__(m)

    def hello(msg):
        print(msg)

    __assert_armored__(hello)
    hello('abc')

.. include:: ../_common_definitions.txt
