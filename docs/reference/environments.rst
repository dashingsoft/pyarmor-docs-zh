.. highlight:: none

====================
 生成加密脚本的环境
====================

这里列出了和命令 :command:`pyarmor` 相关的所有一切。

首先命令 :command:`pyarmor` 运行在 :term:`开发机器` 上面，使用 `supported Python versions`_ 运行在这些 `supported platforms`_

命令行选项， `配置文件选项`_ ， `加密插件`_ ， `脚本补丁`_ 和一些环境变量影响和改变着命令 :command:`pyarmor` 的行为。

所有的命令行选项和相关的环境变量在 :doc:`man` 中有详细描述。

.. _supported python versions:

支持的 Python 版本
==================

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
==========

.. table:: Table-2. 不同功能支持的平台列表
   :widths: auto

   ===================  ============  ========  =======  ============  =========  =======  =======
   OS                     Windows           Apple                    Linux
   -------------------  ------------  -----------------  -----------------------------------------
   Arch                  x86/x86_64    x86_64    arm64    x86/x86_64    aarch64    armv7    armv6
   ===================  ============  ========  =======  ============  =========  =======  =======
   Themida 保护              Y           No        No         No          No       No        No
   RFT 模式                  Y           Y         Y          Y           Y        Y         No
   BCC 模式                  Y           Y         Y          Y           Y        N/y       No
   pyarmor 8 基础功能         Y           Y         Y          Y           Y        Y         No
   pyarmor-7 [#]_           Y           Y         Y          Y           Y        Y         Y
   ===================  ============  ========  =======  ============  =========  =======  =======

.. rubric:: 注释

.. [#] ``N/y`` 意味着现在还不支持，但是将来会支持
.. [#] pyarmor-7 支持更多的平台，参考 `Pyarmor 7.x 文档`__.

.. important::

   pyarmor-7 是 Pyarmor 7.x 的问题修正版本，只支持老版本的许可证。在新版本的许可证下面使用可能会报错 ``HTTP 401 error``

__ https://pyarmor.readthedocs.io/zh/v7.x/platforms.html

配置文件选项
============

有三种类型的配置文件

* :term:`全局配置` 文件，一个 ``.ini`` 格式的文件 :file:`~/.pyarmor/config/global`
* :term:`本地配置` 文件，一个 ``.ini`` 格式的文件 :file:`.pyarmor/config`
* :term:`模块私有配置` 每一个模块可以有自己的私有配置，存放在 :term:`本地配置` 的目录下面

使用命令 :ref:`pyarmor cfg` 来查看和设置配置文件选项。

.. _plugins:

加密插件
========

.. versionadded:: 8.2

.. program:: pyarmor gen

加密插件是一个 Python 脚本，在生成输出文件之后被调用，可以对输出文件进行一些修改和调整，也可以做任何需要的事情。

加密插件常用的场景:

- 对输出目录的文件进行额外处理
- 修改加密脚本中导入运行辅助包的语句，以满足特殊的导入需求
- 增加注释信息到 :term:`外部密钥` 文件，这样通过注释使用外部脚本就可以读取运行配置信息
- 修改运行辅助包中扩展模块 :mod:`pyarmor_runtime` 的后缀，以避免名称冲突
- 使用 `install_name_tool` 修改扩展模块 :mod:`pyarmor_runtime` 的依赖包，解决有些 Darwin 环境无法装载扩展模块的问题

一个加密插件的脚本可以定义一个或者多个插件类，同时必须定义属性 ``__all__`` 来输出脚本中定义的插件类，没有输出的插件类不起作用

插件类的定义如下

.. py:class:: PluginName

    .. py:staticmethod:: post_build(ctx, inputs, outputs, pack=None)

       如果该方法存在，那么当所有加密文件都生成之后会被 :ref:`pyarmor gen` 调用

       :param Context ctx: 加密环境
       :param list inputs: 所有命令行输入的脚本和包名称
       :param list outputs: 输出目录
       :param str pack: 要么是 None，要么是选项 :option:`--pack` 指定的文件名称

    .. py:staticmethod:: post_key(ctx, keyfile, expired=None, devices=None, data=None, period=None)

       如果该方法存在，那么当 :term:`外部密钥` 文件生成之后会被 :ref:`pyarmor gen key` 调用

       :param Context ctx: 加密环境
       :param str keyfile: 输出的外部密钥文件
       :param long expired: None 或者有效期（epoch）
       :param list devices: None 或者设备信息列表
       :param str data: None 或者是绑定的私有数据
       :param int period: None 或者是定时检查运行密钥的周期（秒）

    .. py:staticmethod:: post_runtime(ctx, source, dest, platform)

       如果该方法存在，那么当每一个运行平台的扩展模块 ``pyarmor_runtime`` 被生活之后会被 :ref:`pyarmor gen` 调用，如果是生成多平台的脚本，它可能被调用多次。

       :param Context ctx: 加密环境
       :param str source: 预编译的扩展模块文件名称
       :param str dest: 输出的扩展模块文件名称
       :param str platform: 扩展模块对应的 :term:`运行平台` 名称

为了启用插件脚本，需要进行配置。配置的时候不需要输入扩展名 ``.py`` ，直接使用脚本名称。例如::

    $ pyarmor cfg plugins + "script name"

Pyarmor 按照顺序依次搜索同名的脚本:

- 当前路径
- :term:`本地配置` 目录，通常是 ``.pyarmor/``
- :term:`全局配置` 目录，通常就是 ``~/.pyarmor/``

这里有一个示例插件脚本 ``fooplugin.py``

.. code-block:: python

    __all__ = ['EchoPlugin']

    class EchoPlugin:

        @staticmethod
        def post_runtime(ctx, source, dest, platform):
            print('-------- test fooplugin ----------')
            print('ctx is', ctx)
            print('source is', source)
            print('dest is', dest)
            print('platform is', platform)

把它保存到 ``.pyarmor/fooplugin.py`` 并启用它::

    $ pyarmor cfg plugins + "fooplugin"

加密一个脚本，在控制台可以看到插件运行的输出信息::

    $ pyarmor gen foo.py

禁用插件使用下面的方式::

    $ pyarmor cfg plgins - "fooplugin"

.. _hooks:

脚本补丁
========

.. versionadded:: 8.2

脚本补丁也是一个 Python 脚本，在加密的时候被嵌入到脚本的最前面，相当于直接在脚本的前面插入了插件源代码，这样加密脚本运行的时候就会首先执行插件代码。

当加密脚本的时候，Pyarmor 会在 :term:`本地配置` 和 :term:`全局配置` 目录下面查看有没有路径 ``hooks`` ，如果有的话，那么会查看有没有和加密脚本同名的文件，有的话就会被插入到加密脚本中。

例如， ``.pyarmor/hooks/foo.py`` 就是 ``foo.py`` 的补丁，而 ``.pyarmor/hooks/joker.card.py`` 是 ``joker/card.py`` 的补丁。

脚本补丁就是一个普通的 Python 脚本，但是它可以用两个特殊的内置函数 :func:`__pyarmor__` and :func:`__assert_armored__` 来做一些加密脚本特有的事情。

需要注意的是补丁是直接插入到脚本的模块级别的代码中，所以要避免名称冲突，影响了原来脚本的执行。

.. There is a special hook script ``.pyarmor/hooks/pyarmor_runtime.py`` will be embedded into extension `pyarmor_runtime` init function.

.. seealso:: :func:`__pyarmor__`  :func:`__assert_armorred__`

====================
 运行加密脚本的环境
====================

加密脚本运行在 :term:`客户设备` 上面，支持的平台和 Python 版本和 `生成加密脚本的环境`_ 是一样的。

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
================

.. versionadded:: 8.2

有两个特殊的函数无需导入，可以直接在加密脚本中使用，通常它们一般用在 :term:`脚本补丁` 中。

.. function:: __pyarmor__(arg, kwarg, name, flag)

   :param bytes name: 必须是 ``b'hdinfo'`` 或者 ``b'keyinfo'``
   :param int flag: 必须是 ``1``

   **查询当前设备硬件信息**

   当 ``name`` 等于 ``b'hdinfo'`` 时候，调用这个函数可以获取当前设备硬件信息

   :param int arg: 获取不同类型的设备信息，有效值：0，1，2，3
   :param str kwarg: None，或者设备名称
   :return: arg 为 0 返回硬盘序列号
   :return: arg 为 1 返回网卡以太网地址
   :return: arg 为 2 返回IPv4 地址
   :return: arg 为 3 返回当前设备的名称
   :rtype: 字符串

   例如，

   .. code-block:: python

         __pyarmor__(0, None, b'hdinfo', 1)
         __pyarmor__(1, None, b'hdinfo', 1)

   在 Linux 系统， ``kwarg`` 可以用来指定网卡或者硬盘的设备名称，例如:

   .. code-block:: python

         __pyarmor__(0, "/dev/vda2", b'hdinfo', 1)
         __pyarmor__(1, "eth2", b'hdinfo', 1)

   在 Windows 系统， ``kwarg`` 还可以用来得到全部网卡或者全部硬盘的信息。例如:

   .. code-block:: python

         __pyarmor__(0, "/0", b'hdinfo', 1)    # First disk
         __pyarmor__(0, "/1", b'hdinfo', 1)    # Second disk

         __pyarmor__(1, "*", b'hdinfo', 1)
         __pyarmor__(1, "*", b'hdinfo', 1)

   **查询运行密钥中的数据和有效期**

   当 ``name`` 等于 ``b'keyinfo'`` 的时候，调用这个函数可以获取运行密钥中信息

   :param int arg: 指定获取的信息，有效值：0，1
   :param kwarg: None，暂时没有使用
   :return: arg 为 0 返回运行密钥绑定的私有数据，没有绑定的数据返回 ``b''``
   :rtype: Bytes
   :return: arg 为 1 返回运行密钥的有效期 (epoch)，如果没有设置有效期返回 -1
   :rtype: Long
   :return: 如果发生错误，那么返回 None

   例如，

   .. code-block:: python

         print('bind data is', __pyarmor__(0, None, b'keyinfo', 1))
         print('expired epoch is' __pyarmor__(1, None, b'keyinfo', 1))

.. function:: __assert_armored__(arg)

   :param object arg:  模块或者函数
   :returns: 如果 ``arg`` 指定的对象是被加密过的，那么返回 ``arg`` 自身，否则抛出保护异常

   例如

   .. code-block:: python

       m = __import__('abc')
       __assert_armored__(m)

       def hello(msg):
           print(msg)

       __assert_armored__(hello)
       hello('abc')

.. include:: ../_common_definitions.txt
