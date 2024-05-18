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

   ===================  =====  =========  ==========  ======  ======  =======  ==============
   Python               2.7    3.0~3.6    3.7~3.10     3.11    3.12    3.13+       备注
   ===================  =====  =========  ==========  ======  ======  =======  ==============
   RFT 模式              No       No          Y         Y       Y       N/y      [#]_
   BCC 模式              No       No          Y         Y       Y       N/y
   pyarmor 8 基础功能     No       No          Y         Y       Y       N/y
   pyarmor-7             Y        Y           Y         No      No      No
   ===================  =====  =========  ==========  ======  ======  =======  ==============

.. _supported platforms:

支持的平台
==========

.. table:: 表-2. 不同功能支持的平台列表（一）
   :widths: auto

   ===================  ============  ========  =======  ============  =========  =======  =======
   OS                     Windows           Apple                    Linux [#]_
   -------------------  ------------  -----------------  -----------------------------------------
   Arch                  x86/x86_64    x86_64    arm64    x86/x86_64    aarch64    armv7    armv6
   ===================  ============  ========  =======  ============  =========  =======  =======
   Themida 保护              Y           No        No         No          No       No        No
   RFT 模式                  Y           Y         Y          Y           Y        Y         No
   BCC 模式                  Y           Y         Y          Y           Y        N/y       No
   pyarmor 8 基础功能         Y           Y         Y          Y           Y        Y         No
   pyarmor-7 [#]_           Y           Y         Y          Y           Y        Y         Y
   ===================  ============  ========  =======  ============  =========  =======  =======

.. table:: 表-3. 不同功能支持的平台列表（二） [#]_
   :widths: auto

   ===================  ============  =========  =========  ============  =========  =======  =======
   OS                     FreeBSD        Alpine Linux                      Android
   -------------------  ------------  --------------------  -----------------------------------------
   Arch                    x86_64      x86_64     aarch64    x86/x86_64    aarch64    armv7    armv6
   ===================  ============  =========  =========  ============  =========  =======  =======
   RFT 模式                  Y            Y          Y           Y           Y          Y       No
   BCC 模式                  Y            Y          Y           Y           Y          Y       No
   pyarmor 8 基础功能          Y            Y          Y           Y           Y          Y       No
   pyarmor-7                 Y            Y          Y           Y           Y          Y       Y
   ===================  ============  =========  =========  ============  =========  =======  =======

.. table:: 表-4. 不同功能支持的平台列表（三） [#]_
   :widths: auto

   ===================  =============  ==================================================
   OS                       Linux                Linux, Alpine Linux
   -------------------  -------------  --------------------------------------------------
   Arch                  loongarch64         ppc64le, mips32el/64el, riscv64 [#]_
   ===================  =============  ==================================================
   RFT 模式                  Y                    Y
   BCC 模式                  N                    N
   JIT 功能                  N                    Y
   pyarmor 8 基础功能         Y                    Y
   ===================  =============  ==================================================

.. rubric:: 注释

.. [#] ``N/y`` 意味着现在还不支持，但是将来会支持
.. [#] 这里的 Linux 使用的是 glibc
.. [#] pyarmor-7 支持更多的平台，参考 `Pyarmor 7.x 文档`__.
.. [#] 下表中平台是在 Pyarmor 8.3 新增加的
.. [#] 这些框架在 Pyarmor 8.5.9 中新增加的并且未经测试，预编译的扩展模块分别存放在包 `pyarmor.cli.core.linux`__ and `pyarmor.cli.core.alpine`__
.. [#] 框架 riscv64 仅支持 Python 3.10+

.. important::

   pyarmor-7 是 Pyarmor 7.x 的问题修正版本，只支持老版本的许可证。在新版本的许可证下面使用可能会报错 ``HTTP 401 error``

__ https://pyarmor.readthedocs.io/zh/v7.x/platforms.html
__ https://pypi.org/project/pyarmor.cli.core.linux/#files
__ https://pypi.org/project/pyarmor.cli.core.alpine/#files

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

    .. py:staticmethod:: post_script(ctx, res, source)

       如果该方法存在，那么它会在加密脚本生成之后被调用

       这个方法主要用来对生成的加密脚本进行修改定制，通过修改 `source` 并返回修改后的脚本内容

       :param Context ctx: 加密环境
       :param FileResource res: 实例 `pyarmor.cli.resource.FileResource`
       :param str source: 加密后的脚本内容

    .. py:staticmethod:: post_build(ctx, inputs, outputs, pack=None)

       如果该方法存在，那么当所有加密文件都生成之后会被 :ref:`pyarmor gen` 调用

       :param Context ctx: 加密环境
       :param list inputs: 所有命令行输入的脚本和包名称
       :param list outputs: 输出目录
       :param str pack: 要么是 None，要么是选项 :option:`--pack` 指定的文件名称

    .. py:staticmethod:: post_key(ctx, keyfile,**keyinfo)

       如果该方法存在，那么当 :term:`外部密钥` 文件生成之后会被 :ref:`pyarmor gen key` 调用

       :param Context ctx: 加密环境
       :param str keyfile: 输出的外部密钥文件
       :param dict keyinfo: 外部密钥绑定的信息

       参数 ``keyinfo`` 中可能的项目有

       :key expired: None 或者有效期（epoch）
       :key devices: None 或者设备信息列表
       :key data: None 或者是绑定的私有数据（Bytes）
       :key period: None 或者是定时检查运行密钥的周期（秒）

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

    $ pyarmor cfg plugins - "fooplugin"

.. _hooks:

脚本补丁
========

.. versionadded:: 8.2

脚本补丁也是一个 Python 脚本，在加密的时候被嵌入到脚本的最前面，相当于直接在脚本的前面插入了插件源代码，这样加密脚本运行的时候就会首先执行插件代码。

当加密脚本的时候，Pyarmor 会在 :term:`本地配置` 和 :term:`全局配置` 目录下面查看有没有路径 ``hooks`` ，如果有的话，那么会查看有没有和加密脚本同名的文件，有的话就会被插入到加密脚本中。

例如， ``.pyarmor/hooks/foo.py`` 就是 ``foo.py`` 的补丁，而 ``.pyarmor/hooks/joker.card.py`` 是 ``joker/card.py`` 的补丁。

脚本补丁就是一个普通的 Python 脚本，但是它可以用两个特殊的内置函数 :func:`__pyarmor__` and :func:`__assert_armored__` 来做一些加密脚本特有的事情。

需要注意的是补丁是直接插入到脚本的模块级别的代码中，所以要避免名称冲突，影响了原来脚本的执行。

.. seealso:: :func:`__pyarmor__`  :func:`__assert_armorred__`

特殊脚本补丁
------------

.. versionadded:: 8.3

一般的脚本补丁是嵌入到了加密脚本中，如果需要在运行加密脚本之前就进行一些定制或者额外的检查，那么就需要使用到特殊的脚本补丁 ``.pyarmor/hooks/pyarmor_runtime.py`` ，这个脚本补丁可以定义在加密脚本执行之前就被调用的函数。

首先创建脚本 ``.pyarmor/hooks/pyarmor_runtime.py`` ，然后定义一个函数 :func:`bootstrap` ，这个函数在扩展模块 `pyarmor_runtime` 初始化的过程中被调用，其他代码都会被忽略。

.. function:: bootstrap(user_data)

   :param bytes user_data: 运行密钥里面的用户自定义数据
   :return: 如果返回 False ，那么扩展模块 pyarmor_runtime 初始化失败，并且抛出保护异常
            返回其他任何值，继续执行加密脚本
   :raises SystemExit: 直接退出，不显示调用堆栈
   :raises ohter Exception: 退出并且显示调用堆栈

An example script:

.. code-block:: python

    def bootstrap(user_data):
        # 必须在函数内容导入需要的名称，不要在模块级别导入
        import sys
        import time
        from struct import calcsize

        print('user data is', user_data)

        # 检查平台，不支持 32 位
        if sys.platform == 'win32' and calcsize('P'.encode()) * 8 == 32:
            raise SystemExit('no support for 32-bit windows')

        # 在 Windows 平台下面检查是否有调试器存在
        if sys.platform == 'win32':
            from ctypes import windll
            if windll.kernel32.IsDebuggerPresent():
                print('found debugger')
                return False

        # 在这个例子中，传入的自定义数据是时间戳
        if time.time() > int(user_data.decode()):
            return False

验证一下这个脚本，首先拷贝这个脚本到 ``.pyarmor/hooks/pyarmor_runtime.py`` ，然后执行下面的命令::

    $ pyarmor gen --bind-data 12345 foo.py
    $ python dist/foo.py

    user data is b'12345'
    Traceback (most recent call last):
      File "dist/foo.py", line 2, in <module>
      ...
    RuntimeError: unauthorized use of script (1:10325)

====================
 运行加密脚本的环境
====================

加密脚本运行在 :term:`客户设备` 上面

支持的 Python 版本和平台
========================

运行加密脚本支持的平台和 Python 版本和 `生成加密脚本的环境`_ 是一样的。

环境变量
========

这里的环境变量会被加密脚本使用来决定一些运行设置

.. envvar:: LANG

      操作系统环境变量，加密脚本读取它来决定运行时刻的语言设置

.. envvar:: PYARMOR_LANG

      用来设置运行时刻使用的语言设置。

      如果这个变量存在，那么 :envvar:`LANG` 会被忽略

.. envvar:: PYARMOR_RKEY

      设置搜索 :term:`外部密钥` 的路径

第三方解释器的支持
==================

对于第三方的解释器（例如 Jython 等）以及通过嵌入 Python C/C++ 代码调用加密脚本，只要第三方解释器能够和 CPython :term:`扩展模块` 兼容，就可以使用加密脚本。查看第三方解释器的文档，确认它是否支持 CPython 的扩展模块，

已知的一些问题

* `PyPy` 无法运行加密脚本，因为它完全不同于 `CPython` 。

* 在 Linux 下面 装载 Python 动态库 `libpythonXY.so` 的时候 `dlopen` 必须设置 `RTLD_GLOBAL` ，否则加密脚本无法运行。

* Boost::python，默认装载 Python 动态库是没有设置 `RTLD_GLOAL` 的，运行加密脚本的时候会报错 "No PyCode_Type found" 。解决方法就是在初始化的调用方法 `sys.setdlopenflags(os.RTLD_GLOBAL)` ，这样就可以共享动态库输出的函数和变量。

* 模块 `ctypes` 必须存在并且 `ctypes.pythonapi._handle` 必须被设置为 Python 动态库的句柄，PyArmor 会通过该句柄获取 Python C API 的地址。

* WASM 目前不支持，因为这需要把运行库的代码也编译成为 WASM，但是 WASM 是很容易就被反编译成为原来的 C 代码，为了安全性，所以目前没有支持 WASM 的计划。如果有更多的用户提出这个需求，会考虑实现一个轻量级的运行库，只支持能够运行 RFT 模式的加密脚本，但是目前还没有开发计划。

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

   :param object arg:  模块，函数或者方法
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
