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
从远程服务器下载相应平台的动态库。

在同一个平台下面可能有多个可用的动态库，分别具备不同的特征，一般在标准
平台名称的后面增加一个数字来标识，组成一个唯一的平台 ID。例如，具备全
部特征的动态库为 ``windows.x86_64.7`` ，而没有反调试等特征的动态库为
``windows.x86_64.0`` 。

在跨平台发布的时候需要注意，不同特征的动态库相互不兼容。例如，当前运行
的平台使用的是全特征的动态库 ``windows.x86_64.7`` ，那么是无法直接跨平
台加密使用 `0` 特征的动态库，例如 ``linux.armv7.0`` 等。目前常用的平台
是全特征的，但是其他平台大部分都不是全特征的。

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
* linux.armv7
* linux.aarch32
* linux.aarch64
* android.aarch64
* linux.ppc64
* darwin.arm64
* freebsd.x86_64
* alpine.x86_64
* alpine.arm
* poky.x86

.. list-table:: 表-1. 预安装的动态库清单
   :name: 预安装的动态库清单
   :widths: 10 10 10 20 10 40
   :header-rows: 1

   * - 名称
     - 操作系统
     - CPU架构
     - 特征
     - 下载
     - 说明
   * - windows.x86
     - Windows
     - i686
     - 反调试、JIT、高级模式
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/win32/_pytransform.dll>`_
     - 使用 i686-pc-mingw32-gcc 交叉编译
   * - windows.x86_64
     - Windows
     - AMD64
     - 反调试、JIT、高级模式
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/win_amd64/_pytransform.dll>`_
     - 使用 x86_64-w64-mingw32-gcc 交叉编译
   * - linux.x86
     - Linux
     - i686
     - 反调试、JIT、高级模式
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/linux_i386/_pytransform.so>`_
     - 使用 GCC 编译
   * - linux.x86_64
     - Linux
     - x86_64
     - 反调试、JIT、高级模式
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/linux_x86_64/_pytransform.so>`_
     - 使用 GCC 编译
   * - darwin.x86_64
     - MacOSX
     - x86_64, intel
     - 反调试、JIT、高级模式
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/latest/macosx_x86_64/_pytransform.dylib>`_
     - 使用 CLang 编译（MacOSX10.11）

.. list-table:: 表-2. 其他平台的动态库清单
   :name: 其他平台的动态库清单
   :widths: 10 10 10 20 10 40
   :header-rows: 1

   * - 名称
     - 操作系统
     - CPU架构
     - 特征
     - 下载
     - 说明
   * - vs2015.x86
     - Windows
     - x86
     -
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/vs2015/x86/_pytransform.dll>`_
     - 使用 VS2015 编译
   * - vs2015.x86_64
     - Windows
     - x64
     -
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/vs2015/x64/_pytransform.dll>`_
     - 使用 VS2015 编译
   * - linux.arm
     - Linux
     - armv5
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv5/_pytransform.so>`_
     - 32-bit Armv5 (arm926ej-s)
   * - linux.armv7
     - Linux
     - armv7
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv7/_pytransform.so>`_
     - 32-bit Armv7 Cortex-A, hard-float, little-endian
   * - linux.aarch32
     - Linux
     - aarch32
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv8.32-bit/_pytransform.so>`_
     - 32-bit Armv8 Cortex-A, hard-float, little-endian
   * - linux.aarch64
     - Linux
     - aarch64
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv8.64-bit/_pytransform.so>`_
     - 64-bit Armv8 Cortex-A, little-endian
   * - linux.ppc64
     - Linux
     - ppc64le
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/ppc64le/_pytransform.so>`_
     - 适用于 POWER8
   * - darwin.arm64
     - iOS
     - arm64
     -
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/latest/ios.arm64/_pytransform.dylib>`_
     - 使用 CLang 编译（iPhoneOS9.3sdk）
   * - freebsd.x86_64
     - FreeBSD
     - x86_64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/freebsd/_pytransform.so>`_
     - 不支持获取硬盘序列号
   * - alpine.x86_64
     - Alpine Linux
     - x86_64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/alpine/_pytransform.so>`_
     - 可用于 Docker（musl-1.1.21）
   * - alpine.arm
     - Alpine Linux
     - arm
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/alpine.arm/_pytransform.so>`_
     - 可用于 Docker（musl-1.1.21）, 32 bit Armv5T, hard-float, little-endian
   * - poky.x86
     - Inel Quark
     - i586
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/intel-quark/_pytransform.so>`_
     - 使用 i586-poky-linux 交叉编译
   * - android.aarch64
     - Android
     - aarch64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/android.aarch64/_pytransform.so>`_
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang
