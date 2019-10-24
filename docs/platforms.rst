.. _支持的平台列表:

支持的平台列表
==============

PyArmor 的核心函数使用 C 来实现，对于常用的平台和部分嵌入式系统都已经
有编译好的动态库。

最常用平台的动态库已经打包在 PyArmor 的安装包里面，只要安装好之后即可
使用，参考 `预安装的动态库清单`_ 。

其他平台的动态库并没有随着安装包发布，参考 `其他平台的动态库清单`_ 。
在这些平台下面，使用 PyArmor 之前，需要下载相应的动态库，并存放到
PyArmor 安装的路径之下（通常情况下和 pyarmor.py 在相同目录）。

如果需要在上面没有列出的平台使用 PyArmor，请发送邮件到 jondy.zhao@gmail.com

.. list-table:: 表-1. 预安装的动态库清单
   :name: 预安装的动态库清单
   :widths: 10 10 20 60
   :header-rows: 1

   * - 操作系统
     - CPU架构
     - 特征
     - 下载
     - 说明
   * - Windows
     - i686
     - 反调试、JIT、高级模式
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/win32/_pytransform.dll>`_
     - 使用 i686-pc-mingw32-gcc 交叉编译
   * - Windows
     - AMD64
     - 反调试、JIT、高级模式
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/win_amd64/_pytransform.dll>`_
     - 使用 x86_64-w64-mingw32-gcc 交叉编译
   * - Linux
     - i686
     - 反调试、JIT、高级模式
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/linux_i386/_pytransform.so>`_
     - 使用 GCC 编译
   * - Linux
     - x86_64
     - 反调试、JIT、高级模式
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/linux_x86_64/_pytransform.so>`_
     - 使用 GCC 编译
   * - MacOSX
     - x86_64, intel
     - 反调试、JIT、高级模式
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/latest/macosx_x86_64/_pytransform.dylib>`_
     - 使用 CLang 编译（MacOSX10.11）

.. list-table:: 表-2. 其他平台的动态库清单
   :name: 其他平台的动态库清单
   :widths: 10 10 20 60
   :header-rows: 1

   * - 操作系统
     - CPU架构
     - 特征
     - 下载
     - 说明
   * - Windows
     - x86
     -
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/vs2015/x86/_pytransform.dll>`_
     - 使用 VS2015 编译
   * - Windows
     - x64
     -
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/latest/vs2015/x64/_pytransform.dll>`_
     - 使用 VS2015 编译
   * - Linux
     - armv5
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv5/_pytransform.so>`_
     - 32-bit Armv5 (arm926ej-s)
   * - Linux
     - armv7
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv7/_pytransform.so>`_
     - 32-bit Armv7 Cortex-A, hard-float, little-endian
   * - Linux
     - aarch32
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv8.32-bit/_pytransform.so>`_
     - 32-bit Armv8 Cortex-A, hard-float, little-endian
   * - Linux
     - aarch64
     - 反调试、JIT
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/armv8.64-bit/_pytransform.so>`_
     - 64-bit Armv8 Cortex-A, little-endian
   * - Linux
     - ppc64le
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/ppc64le/_pytransform.so>`_
     - 适用于 POWER8
   * - iOS
     - arm64
     -
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/latest/ios.arm64/_pytransform.dylib>`_
     - 使用 CLang 编译（iPhoneOS9.3sdk）
   * - FreeBSD
     - x86_64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/freebsd/_pytransform.so>`_
     - 不支持获取硬盘序列号
   * - Alpine Linux
     - x86_64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/alpine/_pytransform.so>`_
     - 可用于 Docker（musl-1.1.21）
   * - Alpine Linux
     - arm
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/alpine.arm/_pytransform.so>`_
     - 可用于 Docker（musl-1.1.21）, 32 bit Armv5T, hard-float, little-endian
   * - Inel Quark
     - i586
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/intel-quark/_pytransform.so>`_
     - 使用 i586-poky-linux 交叉编译
   * - Android
     - aarch64
     -
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/latest/android.aarch64/_pytransform.so>`_
     - Build by android-ndk-r20/toolchains/llvm/prebuilt/linux-x86_64/bin/aarch64-linux-android21-clang
