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

这个命令会生成一个加密脚本 :file:`dist/foo.py` ，这也是一个正常的 Python 脚本，可以直接使用 Python 解释器执行::

    $ python dist/foo.py

查看所有生成的文件::

    $ ls dist/
    ...    foo.py
    ...    pyarmor_runtime_000000

除了加密脚本之外，可以看到还有另外一个目录 :file:`pyarmor_runtime_000000` ，这是运行加密脚本所依赖的一个 :term:`Python 包` 。

发布加密脚本
============

只拷贝加密脚本 :file:`dist/foo.py` 本身到 :term:`客户设备` 是无法运行的，必须把输出目录下面的 :term:`运行辅助包` 一起拷贝过去才可以。

为什么呢？看一下加密脚本 :file:`dist/foo.py` 的内容就明白了:

.. code-block:: python

    from pyarmor_runtime_000000 import __pyarmor__
    __pyarmor__(__name__, __file__, ...)

加密脚本需要从 ``pyarmor_runtime_000000`` 导入函数 ``__pyarmor__`` ，这个包也是加密脚本的依赖包，加密脚本可以被当作一个正常脚本和依赖包 :mod:`pyarmor_runtime_000000` 来使用。

.. important::

   因为依赖包 :mod:`pyarmor_runtime_000000` 包含使 :term:`扩展模块` ，所以加密脚本只能在相同系统，使用相同版本的 Python 才能运行。如果 :term:`客户设备` 的运行环境不一样，需要使用其他跨平台加密选项。

.. note::

   不需要安装 Pyarmor 到 :term:`客户设备` ，运行加密脚本不需要 Pyarmor

加密包
======

现在来加密一个包，使用选项 :option:`-O` 设置另外一个输出目录 :file:`dist2`::

    $ pyarmor gen -O dist2 src/mypkg

查看加密结果::

    $ ls dist2/
    ...    mypkg
    ...    pyarmor_runtime_000000

    $ ls dist2/mypkg/
    ...          __init__.py

测试一下导入加密后的包 :file:`dist2/mypkg`::

    $ cd dist2/
    $ python -C 'import mypkg'

如果包里面还有其他子目录需要加密，那么使用选项 :option:`-r` 来启用递归搜索模式::

    $ pyarmor gen -O dist2 -r src/mypkg

发布加密包
==========

虽然可以把整个目录 :file:`dist2` 直接拷贝到 :term:`客户设备` ，但是还有一种更好的方式，使用选项 :option:`-i` 把运行辅助包保存到包目录内部::

    $ pyarmor gen -O dist3 -r -i src/mypkg

查看输出目录::

    $ ls dist3/
    ...    mypkg

    $ ls dist3/mypkg/
    ...          __init__.py
    ...          pyarmor_runtime_000000

现在所有需要拷贝的文件都在加密包 :file:`dist3/mypkg` 内部，只需要整个包目录拷贝到 :term:`客户设备` 上面就可以了。

.. note::

   可以比较一下 :file:`dist3/mypkg/__init__.py` 和上一节生成到的加密文件 :file:`dist2/mypkg/__init__.py` 的内容更多的了解这个选项的作用。

封装加密包
------------

再说一次，加密脚本就是正常的 Python 脚本，所以其他用来封装 Python 脚本的工具，例如 distutils， setuptools，以及 wheel 都可以用来封装加密脚本。

假设包 ``mypkg`` 的目录结构如下::

    projects/
    └── src/
        └── mypkg/
            ├── __init__.py
            ├── utils.py
            └── config.json

首先创建一个输出目录 :file:`projects/dist6` 用来保存加密包::

    $ cd projects
    $ mkdir dist6

然后把所有数据文件到拷贝过去::

    $ cp -a src/mypkg dist6/

接下来生成加密包，把所有加密后的 ``.py`` 文件保存到输出目录 ``.py``::

    $ pyarmor gen -O dist6 -i src/mypkg

最终的输出如下::

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

比较一下 :file:`src/mypkg` 和 :file:`dist6/mypkg` ，唯一的区别是后者多了一个目录 ``pyarmor_runtime_000000`` ，最后要做的就是使用你熟悉的方式封装 :file:`dist6/mypkg`

还不了解如何封装 Python 包？请参考这里学习 `Python Packaging User Guide`_

.. _Python Packaging User Guide: https://packaging.python.org

设置加密脚本有效期
==================

使用选项 :option:`-e` 可以方便的设置加密脚本的有效期。例如，设置加密脚本有效期为30天::

    $ pyarmor gen -O dist4 -e 30 foo.py

运行一下加密脚本 :file:`dist4/foo.py` 来验证一下::

    $ python dist4/foo.py

加密脚本使用 NTP_ 服务器来验证是否过期，如果当前设备不能访问网络，会报错退出。

也可以使用另外一种格式 ``YYYY-MM-DD`` 来设置有效期，例如::

    $ pyarmor gen -O dist4 -e 2020-12-31 foo.py

运行一下 :file:`dist4/foo.py` 要进行验证::

    $ python dist4/foo.py

如果不需要验证网络时间，可以在有效期前面增加前缀 ``.`` 表示检查本地时间。例如::

    $ pyarmor gen -O dist4 -e .30 foo.py
    $ pyarmor gen -O dist4 -e .2020-12-31 foo.py

发布有时间限制的加密脚本和上面的方法是一样的，直接拷贝整个输出目录 :file:`dist4/` 到 :term:`客户设备`

绑定加密脚本到指定设备
======================

使用 Pyarmor 8.4.6+ 可以通过命令 `python -m pyarmor.cli.hdinfo` 直接得到:term:`客户设备` 的硬件信息如下::

    Default Harddisk Serial Number: 'HXS2000CN2A'
    Default Mac address: '00:16:3e:35:19:3d'
    Default IPv4 address: '128.16.4.10'

Pyarmor 8.4.6 之前的版本可以通过命令 `pyarmor-7 hdinfo` 查询硬件信息。

使用选项 :option:`-b` 来绑定硬件信息到加密角本。例如，绑定 :file:`dist5/foo.py` 到网卡以太网地址::

    $ pyarmor gen -O dist5 -b 00:16:3e:35:19:3d foo.py

使用相同的选项来绑定 IPv4 地址和硬盘序列号::

    $ pyarmor gen -O dist5 -b 128.16.4.10 foo.py
    $ pyarmor gen -O dist5 -b HXS2000CN2A foo.py

组合多种硬件信息使用下面的格式::

    $ pyarmor gen -O dist5 -b "00:16:3e:35:19:3d HXS2000CN2A" foo.py

只有设备的硬件信息都符合绑定的信息，加密脚本才能运行，否则报错退出。

发布绑定到设备的加密脚本和上面的方法是一样的，直接拷贝整个输出目录 :file:`dist4/` 到 :term:`客户设备`

关于加密脚本必须要知道的
========================

运行加密脚本需要一个 :term:`扩展模块` :mod:`pyarmor_runtime` ，它在运行辅助包 ``pyarmor_runtime_000000`` 目录下面::

    $ ls dist6/mypkg/pyarmor_runtime_000000
    ...    __init__.py
    ...    pyarmor_runtime.so

使用二进制的扩展模块意味着加密脚本需要有为各个平台的预编译的扩展模块 :mod:`pyarmor_runtime` ，所以加密脚本

* 只能运行在那些已经有预编译扩展模块的平台，所有支持的平台请参考 :doc:`../reference/environments`
* 只能使用相同版本 CPython interpreter 解释器来运行，例如使用 Python 3.8 加密的脚本，无法被 Python 3.9 运行
* 一般不能被第三方解释器，例如 PyPy， IronPython 或者 Jython 等来运行

还有，在 Android 系统下面， ``.py`` 脚本可以在任意目录下面运行，但是扩展模块是动态库，就必须在系统特定的目录下面才能运行。

下一步的教程
============

根据你的需要进行选择下一步的教程

这里有完整的 :doc:`安装教程 <installation>` 包含如下内容:

* 从 Github 库直接安装 Pyarmor
* 如何从 Python 脚本中调用 Pyarmor
* 完整卸载

接下来是 :doc:`obfuscation` 包含的内容有:

* 使用更多选项加密脚本和包
* 使用外部密钥文件限制加密脚本的运行
* 本地化错误信息
* 生成不需要 Python 环境就可以独立运行的加密脚本

还有 :doc:`advanced` ，有些功能在试用版中无法使用

* 如何使用两种不可逆的加密模式: RFT 模式和 BCC 模式 :sup:`pro`
* 定制错误退出方式
* 国际化错误消息
* 加密跨平台的加密脚本

很多用户可能对这里的内容感兴趣 :doc:`../how-to/security`

如何阅读本手册
==============

|Pyarmor| 有完备的文档系统，这里是帮助用户如何快速找到需要的相关内容

* :doc:`第一部分: 基础教程 <../part-1>` 适合第一次使用 Pyarmor 的用户，这里以实例的形式一步接着一步的说明了加密脚本和包的最常用的场景。也可以先看看 :doc:`getting-started`

* :doc:`第二部分: 应用实践 <../part-2>` 针对每一个特定的需求，说明在 Pyarmor 中应该如何去做，使用什么样的命令和选项去实现。阅读这部分内容需要对 Pyarmor 和 Python 都有一定的了解。

* :doc:`第三部分: 技术手册 <../part-3>` 从技术层的角度详细列出了所有的概念定义，命令手册，配置选项和错误信息代码。它适用于使用 Pyarmor 的高级用户，需要查找相关的参数和配置，了解这些配置项的可用值和不同值的作用和含义。

* :doc:`第四部分: 深入了解 <../part-4>` 这部分针对 Pyarmor 提供的功能，从如何实现的层面进行了详细的解释。阅读这部分内容需要完全掌握了 Pyarmor 使用到的主要概念，以及对 Python 脚本的执行过程有相当了解。它适用于需要对 Pyarmor 进行扩展和定制，以满足更高一层需求的用户。

* :doc:`第五部分: 许可模式和许可证类型 <../licenses>` 描述了 |Pyarmor| 的最终用户许可协议， |Pyarmor| 的许可模式，不同的许可类型，以及如何购买 |Pyarmor| 许可证。

想找某一个特定的关键字，搜一下 :ref:`总索引 <genindex>`, 或者浏览 :ref:`文档总目录 <mastertoc>`

还是没有找到？ 看一下 :ref:`如何在 Github 上提问 <asking questions>`.

.. include:: ../_common_definitions.txt
