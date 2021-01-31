.. _支持的平台列表:

支持的平台列表
==============

PyArmor 的核心函数使用 C 来实现，对于常用的平台和部分嵌入式系统都已经
有编译好的动态库。

最常用平台的动态库已经打包在 PyArmor 的安装包里面，只要安装好之后即可
使用，参考 `预安装的动态库清单`_ 。

其他平台的动态库并没有随着安装包发布，参考 `其他平台的动态库清单`_ 。
在这些平台下面， `pyarmor` 会搜索 ``~/.pyarmor/platforms/SYSTEM/ARCH``
，其中 ``SYSTEM.ARCH`` 是一个 `标准平台名称`_ 。如果还没有下载，那么会
自动从远程服务器下载相应平台的动态库。

从 v6.2.0 开始， 新增加的 :ref:`超级模式` 和以前的动态库不一样，它是直接使用扩展
模块 :mod:`pytransform` ，可用的扩展模块列参看 `超级模式预编译扩展模块表`_

最新的全部支持的动态库详细列表可以参考 `pyarmor-core/platforms/index.json <https://github.com/dashingsoft/pyarmor-core/blob/master/platforms/index.json>`_

在同一个平台下面可能有多个可用的动态库，分别具备不同的特征，一般在标准
平台名称的后面增加一个数字来标识，组成一个唯一的平台 ID。

每一个特征都有自己的标志位

- 1: 反调试
- 2: JIT，动态代码
- 4: 高级模式
- 8: 超级模式
- 16: 虚拟模式

例如，动态库为 ``windows.x86_64.7`` 具备特征 反调试(1)，JIT(2)，高级模式(4)，而
``windows.x86_64.0`` 则表示没有任何额外的特征，相应的性能也最高。

在跨平台发布的时候需要注意，特征为 ``0`` 的动态库和其他具备特征的动态库是相互不
兼容的，为了提高安全性，没有任何特征的动态库使用的加密算法和有特征的库是不相同的。
所以，动态库 ``windows.x86_64.7`` 是无法和 ``linux.armv7.0`` 共用相同的加密脚本
的。

有些平台 `pyarmor` 无法自动识别，但是在 `其他平台的动态库清单`_ 中有可
用的动态库。可以直接下载下来，保存到这个平台的搜索路径
``~/.pyarmor/platforms/SYSTEM/ARCH`` 下面。如果不能确定存放的路径，可
以使用命令 ``pyarmor -d download`` 查看，在开始的时候会显示 `pyarmor`
去那里查找动态库。如果需要让 `pyarmor` 自动识别这个平台，请把下面这个
脚本的输出发送到 jondy.zhao@gmail.com

.. code-block:: python

   from platform import *
   print('system name: %s' % system())
   print('machine: %s' % machine())
   print('processor: %s' % processor())
   print('aliased terse platform: %s' % platform(aliased=1, terse=1))

   if system().lower().startswith('linux'):
       print('libc: %s' % libc_ver())
       print('distribution: %s' % linux_distribution())

如果需要在上面没有列出的平台使用 PyArmor，请发送邮件到
jondy.zhao@gmail.com

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

.. list-table:: 表-1. 预安装的动态库清单
   :name: 预安装的动态库清单
   :widths: 10 10 10 20 10 40
   :header-rows: 1

   * - 名称
     - 操作系统
     - CPU架构
     - 特征
     - 说明
   * - windows.x86
     - Windows
     - i686
     - 反调试、JIT、高级模式
     - 使用 i686-pc-mingw32-gcc 交叉编译
   * - windows.x86_64
     - Windows
     - AMD64
     - 反调试、JIT、高级模式
     - 使用 x86_64-w64-mingw32-gcc 交叉编译
   * - linux.x86
     - Linux
     - i686
     - 反调试、JIT、高级模式
     - 使用 GCC 编译
   * - linux.x86_64
     - Linux
     - x86_64
     - 反调试、JIT、高级模式
     - 使用 GCC 编译
   * - darwin.x86_64
     - MacOSX
     - x86_64, intel
     - 反调试、JIT、高级模式
     - 使用 CLang 编译（MacOSX10.11）

.. list-table:: 表-2. 其他平台的动态库清单
   :name: 其他平台的动态库清单
   :widths: 10 10 10 20 10 40
   :header-rows: 1

   * - 名称
     - 操作系统
     - CPU架构
     - 特征
     - 说明
   * - vs2015.x86
     - Windows
     - x86
     -
     - 使用 VS2015 编译
   * - vs2015.x86_64
     - Windows
     - x64
     -
     - 使用 VS2015 编译
   * - linux.arm
     - Linux
     - armv5
     -
     - 32-bit Armv5 (arm926ej-s)
   * - linxu.armv6
     - Linux
     - armv6
     - 反调试、JIT
     - 32-bit Armv6 (-marm -march=armv6 -mfloat-abi=hard)
   * - linux.armv7
     - Linux
     - armv7
     - 反调试、JIT
     - 32-bit Armv7 Cortex-A, hard-float, little-endian
   * - linux.aarch32
     - Linux
     - aarch32
     - 反调试、JIT
     - 32-bit Armv8 Cortex-A, hard-float, little-endian
   * - linux.aarch64
     - Linux
     - aarch64
     - 反调试、JIT
     - 64-bit Armv8 Cortex-A, little-endian
   * - linux.ppc64
     - Linux
     - ppc64le
     -
     - 适用于 POWER8
   * - darwin.arm64
     - iOS
     - arm64
     -
     - 使用 CLang 编译（iPhoneOS9.3sdk）
   * - freebsd.x86_64
     - FreeBSD
     - x86_64
     -
     - 不支持获取硬盘序列号
   * - musl.x86_64
     - Alpine Linux
     - x86_64
     -
     - 可用于 Docker（musl-1.1.21）
   * - musl.arm
     - Alpine Linux
     - arm
     -
     - 可用于 Docker（musl-1.1.21）, 32 bit Armv5T, hard-float, little-endian
   * - poky.x86
     - Inel Quark
     - i586
     -
     - 使用 i586-poky-linux 交叉编译
   * - android.aarch64
     - Android
     - aarch64
     -
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang
   * - android.armv7
     - Android
     - armv7l
     -
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/armv7a-linux-android21-clang
   * - android.x86
     - Android
     - x86
     -
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/i686-linux-android21-clang
   * - android.x86_64
     - Android
     - x86_64
     -
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/x86_64-linux-android21-clang
   * - uclibc.armv7
     - Linux
     - armv7l
     -
     - Build by armv7-buildroot-uclibceabihf-gcc
   * - windows.x86.21
     - Windows
     - i686
     - 反调试、高级模式、虚拟模式
     - 使用 i686-w64-mingw32-gcc 交叉编译
   * - windows.x86_64.21
     - Windows
     - AMD64
     - 反调试、高级模式、虚拟模式
     - 使用 x86_64-w64-mingw32-gcc 交叉编译

.. list-table:: Table-3. 超级模式预编译扩展模块表
   :name: 超级模式预编译扩展模块表
   :widths: 10 10 10 20 10 40
   :header-rows: 1

   * - 名称
     - 操作系统
     - CPU架构
     - 特征
     - 说明
   * - darwin.x86_64.11.py38
     - MacOSX
     - x86_64, intel
     - Anti-Debug, JIT, SUPER
     - Built by CLang with MacOSX10.11
   * - darwin.x86_64.11.py37
     - MacOSX
     - x86_64, intel
     - Anti-Debug, JIT, SUPER
     - Built by CLang with MacOSX10.11
   * - darwin.x86_64.11.py27
     - MacOSX
     - x86_64, intel
     - Anti-Debug, JIT, SUPER
     - Built by CLang with MacOSX10.11
   * - linux.x86_64.11.py38
     - Linux
     - x86_64
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.x86_64.11.py37
     - Linux
     - x86_64
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.x86_64.11.py27
     - Linux
     - x86_64
     - Anti-Debug, JIT, SUPER
     - Built by gcc, UCS4
   * - centos6.x86_64.11.py27
     - Linux
     - x86_64
     - Anti-Debug, JIT, SUPER
     - Built by gcc, UCS2
   * - windows.x86_64.11.py38
     - Windows
     - AMD64
     - Anti-Debug, JIT, SUPER
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86_64.11.py37
     - Windows
     - AMD64
     - Anti-Debug, JIT, SUPER
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86_64.11.py27
     - Windows
     - AMD64
     - Anti-Debug, JIT, SUPER
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86.11.py38
     - Windows
     - i386
     - Anti-Debug, JIT, SUPER
     - Cross compile by i686-w64-mingw32-gcc in cygwin
   * - windows.x86.11.py37
     - Windows
     - i386
     - Anti-Debug, JIT, SUPER
     - Cross compile by i686-w64-mingw32-gcc in cygwin
   * - windows.x86.11.py27
     - Windows
     - i386
     - Anti-Debug, JIT, SUPER
     - Cross compile by i686-w64-mingw32-gcc in cygwin
   * - linux.x86.11.py38
     - Linux
     - i386
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.x86.11.py37
     - Linux
     - i386
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.x86.11.py27
     - Linux
     - i386
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch64.11.py38
     - Linux
     - aarch64
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch64.11.py37
     - Linux
     - aarch64
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch64.11.py27
     - Linux
     - aarch64
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch32.11.py38
     - Linux
     - aarch32
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch32.11.py37
     - Linux
     - aarch32
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.aarch32.11.py27
     - Linux
     - aarch32
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.armv7.11.py38
     - Linux
     - armv7l
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.armv7.11.py37
     - Linux
     - armv7l
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - linux.armv7.11.py27
     - Linux
     - armv7l
     - Anti-Debug, JIT, SUPER
     - Built by gcc
   * - windows.x86_64.25.py38
     - Windows
     - AMD64
     - Anti-Debug, SUPER, VM
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86_64.25.py37
     - Windows
     - AMD64
     - Anti-Debug, SUPER, VM
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86_64.25.py27
     - Windows
     - AMD64
     - Anti-Debug, JIT, SUPER
     - Cross compile by x86_64-w64-mingw32-gcc in cygwin
   * - windows.x86.25.py38
     - Windows
     - i386
     - Anti-Debug, SUPER, VM
     - Cross compile by i686-w64-mingw32-gcc in cygwin
   * - windows.x86.25.py37
     - Windows
     - i386
     - Anti-Debug, SUPER, VM
     - Cross compile by i686-w64-mingw32-gcc in cygwin
   * - windows.x86.25.py27
     - Windows
     - i386
     - Anti-Debug, SUPER, VM
     - Cross compile by i686-w64-mingw32-gcc in cygwin

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
