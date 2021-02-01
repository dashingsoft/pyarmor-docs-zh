.. _支持的平台列表:

支持的平台列表
==============

PyArmor 的核心函数使用 C 来实现，对于常用的平台和部分嵌入式系统都已经有编译好的
动态库。

最常用平台的动态库已经打包在 PyArmor 的安装包里面，只要安装好之后即可使用::

    windows.x86
    windows.x86_64
    linux.x86
    linux.x86_64
    darwin.x86_64/x86

其他平台的动态库并没有随着安装包发布，在这些平台下面运行 `pyarmor` 的时候，默认
情况下会搜索路径 ``~/.pyarmor/platforms/SYSTEM/ARCH/N/`` 去查找相应平台的动态库，
其中 ``SYSTEM.ARCH`` 是一个 `标准平台名称`_ 。 ``N`` 一般为数值，用来标示动态库
的特征，关于不同特征的动态库参考下面的说明。如果没有发现任何可用的动态库，那么会
自动从远程服务器查找和下载相应平台的动态库，目前支持的其他平台包括::

    darwin.arm64
    linux.arm
    linux.armv6
    linux.armv7
    linux.aarch32
    linux.aarch64
    linux.ppc64
    android.aarch64
    android.armv7
    android.x86
    android.x86_64
    uclibc.armv7
    centos6.x86_64
    freebsd.x86_64
    musl.x86_64
    musl.arm
    musl.mips32
    poky.x86

一些 Linux 平台使用不同的 c 库，例如 Docker 基于 Alpine Linux，使用的是 `musl`
，有的嵌入系统使用的是 `uclibc` ，在平台名称中使用第一个标识符来区别。

其中 ``centos6`` 是一个特殊名称，用来标示 ``glibc`` 版本小于 2.14 的 Linux 平台。

:ref:`超级模式` 是直接使用扩展模块 :mod:`pytransform` ，不同版本的 Python 分别对
应相应的一个扩展模块，并保存在 ``~/.pyarmor/platforms/SYSTEM/ARCH/N/pyXY`` 下面。
最后的一级目录是对应的 Python 版本，对于超级模式来说，特征值一般就是 ``11`` ，例
如 ``linux/x86_64/11/py38`` 用来存放 64位 Linux 下 Python38 的超级模式动态库。

目前超级模式支持的所有平台和架构如下

.. list-table:: 表-1. 超级模式预编译扩展模块表
   :name: 超级模式预编译扩展模块表
   :header-rows: 1

   * - 平台名称
     - 支持架构
     - 特征值
     - Python 版本
     - 说明
   * - darwin
     - x86_64
     - 11
     - 27, 37, 38, 39
     -
   * - darwin
     - aarch64
     - 11
     - 38, 39
     - ios/darwin arm64
   * - linux
     - x86, x86_64, aarch64, aarch32, armv7
     - 11
     - 27, 37, 38, 39
     -
   * - centos6
     - x86_64
     - 11
     - 27, 37, 38, 39
     - Linux with glibc < 2.14
   * - windows
     - x86, x86_64
     - 11, 25
     - 27, 37, 38, 39
     -

最新的全部支持的动态库详细列表可以参考 `pyarmor-core/platforms/index.json <https://github.com/dashingsoft/pyarmor-core/blob/master/platforms/index.json>`_

有些平台 pyarmor 无法自动识别，但是有可用的动态库。可以直接下载下来，保存到平台
的搜索路径 ``~/.pyarmor/platforms/SYSTEM/ARCH/N/`` 下面。如果不能确定存放的路径，
可以使用命令 ``pyarmor -d download`` 查看，在输出日志中会显示 pyarmor 去那里查找
动态库。

动态库特征值
-----------

在同一个平台下面可能有多个可用的动态库，分别具备不同的特征，一般在标准平台名称的
后面增加一个数字来标识，组成一个唯一的平台 ID。

每一个特征都有自己的标志位

- 1: 反调试
- 2: JIT，动态代码
- 4: 高级模式
- 8: 超级模式
- 16: 虚拟模式

例如，动态库为 ``windows.x86_64.7`` 具备特征 反调试(1)，JIT(2)，高级模式(4)，而
``windows.x86_64.0`` 则表示没有任何额外的特征，相应的性能也最高。

对于超级模式来说，还需要把 Python 版本标示出来，例如 ``windows.x86.11.py37``

在跨平台发布的时候需要注意，特征为 ``0`` 的动态库和其他具备特征的动态库是相互不
兼容的，为了提高安全性，没有任何特征的动态库使用的加密算法和有特征的库是不相同的。
所以，动态库 ``windows.x86_64.7`` 是无法和 ``linux.armv7.0`` 共用相同的加密脚本
的。


.. _标准平台名称:

标准平台名称
------------

这些名称可用于命令 :ref:`obfuscate`, :ref:`build`, :ref:`runtime`,
:ref:`download` 中来指定平台名称。

* windows.x86
* windows.x86_64
* linux.x86
* linux.x86_64
* darwin.x86_64
* vs2015.x86
* vs2015.x86_64
* linux.arm
* linux.armv6
* linux.armv7
* linux.aarch32
* linux.aarch64
* android.aarch64
* android.armv7 (从 5.9.3 开始支持）
* android.x86 (从 6.6.1 开始支持）
* android.x86_64 (从 6.6.1 开始支持）
* uclibc.armv7 (从 5.9.4 开始支持）
* linux.ppc64
* darwin.arm64
* freebsd.x86_64
* musl.x86_64 （在 6.3.1 更名，原来的名字 alpine.x86_64）
* musl.arm （在 6.3.1 更名，原来的名字 alpine.arm）
* musl.mips32 （从 6.3.1 开始支持）
* poky.x86

.. _如何人工下载和配置动态库:

如何人工下载和配置动态库
------------------------

在联网的情况下，PyArmor 可以自动下载和配置需要的动态库，在不联网的机器上则需要把
预先下载的动态库放置在相应的目录下面。

首先下载 ``platforms/index.json`` ，如果是使用 pip 安装的话，可以忽略这一步，因
为这个文件会被自动安装的。在没有联网的机子上运行相应的命令，会出现如下提示，例如::

    pyarmor.py o --advanced 2 test.py

    INFO     PyArmor Version 6.4.2
    INFO     Target platforms: Native
    INFO     Getting remote file: https://github.com/dashingsoft/pyarmor-core/raw/r34.8/platforms/index.json
    INFO     Could not get file from https://github.com/dashingsoft/pyarmor-core/raw/r34.8/platforms: <urlopen error timed out>
    INFO     Getting remote file: https://pyarmor.dashingsoft.com/downloads/r34.8/index.json
    INFO     Could not get file from https://pyarmor.dashingsoft.com/downloads/r34.8: <urlopen error timed out>
    ERROR    No platform list file /data/user/.pyarmor/platforms/index.json found

上面提示中有两个下载地址，选择其中一个在联网的机子上下载 ``index.json`` ，例如

https://pyarmor.dashingsoft.com/downloads/r34.8/index.json

然后把下载的文件拷贝到没有联网机子上，保存在提示中的位置。例如，示例中的提示地址::

    /data/user/.pyarmor/platforms/index.json

需要注意不同版本的 PyArmor 都有自己对应的 ``index.json`` ，必须保持一致。

接下来再次运行相应的命令，这时候同样会提示下载的动态库的地址，例如::

    pyarmor o --advanced 2 test.py

    ...
    INFO Use capsule: /root/.pyarmor/.pyarmor_capsule.zip
    INFO Output path is: /root/supervisor/dist
    INFO Taget platforms: []
    INFO Update target platforms to: [u'linux.x86_64.11.py27']
    INFO Generating super runtime library to dist
    INFO Search library for platform: linux.x86_64.11.py27
    INFO Found available libraries: [u'linux.x86_64.11.py27']
    INFO Target path for linux.x86_64.11.py27: /home/jondy/.pyarmor/platforms/linux/x86_64/11/py27
    INFO Downloading library file for linux.x86_64.11.py27 ...
    INFO Getting remote file: https://github.com/dashingsoft/pyarmor-core/raw/r34.8/platforms/linux.x86_64.11.py27/pytransform.so
    INFO Could not get file from https://github.com/dashingsoft/pyarmor-core/raw/r34.8/platforms: <urlopen error [Errno 111] Connection refused>
    INFO Getting remote file: https://pyarmor.dashingsoft.com/downloads/r34.8/linux.x86_64.11.py27/pytransform.so
    INFO Could not get file from https://pyarmor.dashingsoft.com/downloads/r34.8: <urlopen error [Errno 111] Connection refused>
    ERROR Download library file failed

按照提示的任意一个地址下载相应的动态库，例如

https://github.com/dashingsoft/pyarmor-core/raw/r34.8/platforms/linux.x86_64.11.py27/pytransform.so

然后保存到日志 ``INFO Target path`` 后面列出的路径，例如，这里是::

    /home/jondy/.pyarmor/platforms/linux/x86_64/11/py27

对于 PyArmor 6.5.5 之前的版本，没有保存提示路径。可以直接存放到
``~/.pyarmor/platforms/`` 加上平台路径，平台路径一般就是把平台名称中的点替换为路
径分隔符，例如，平台名称 ``linux.x86_64.11.py27`` 的存放路径就是
``~/.pyarmor/platforms/linux/x86_64/11/py27``

请注意检查下载的动态库的 sha256 的值，要确保其和 ``index.json`` 文件中对应的值一
致。

另外所有版本的动态库和对应的 ``index.json`` 都存放在 github 库 `pyarmor-core`

https://github.com/dashingsoft/pyarmor-core

也可以直接在上面下载对应版本的动态库，PyArmor 每一个版本都有一个对应的 tag ，例
如这里 PyArmor 是 6.4.2 ，对应的核心库 tag 是 ``r34.8`` ，所以可以切换这个库里面
到 tag ``r34.8`` ，然后在目录 `platforms` 下面下载对应的动态库。

.. note::

   如果存在 DSN 问题，执行 ``ping pyarmor.dashingsoft.com`` 提示主机名找不到，请
   增加一行到 ``/etc/hosts``::

       119.23.58.77 pyarmor.dashingsoft.com
