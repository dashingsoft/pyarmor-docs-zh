.. _PyArmor不同平台动态链接库清单:

PyArmor不同平台动态链接库清单
=============================

版本: **5.1.5**

创建日期: 2019-02-19

PyArmor 的核心函数使用 C 来实现，对于常用的平台和部分嵌入式系统都已经
有编译好的动态库。

最常用平台的动态库已经打包在 PyArmor 的安装包里面，只要按照好之后即可使用，参考 `预安装的动态库清单`_ 。

其他平台的动态库并没有随着安装包发布，参考 `其他支持平台的动态库清单`_ 。 在这些平台下面，使用 PyArmor 之前，需要下载相应的动态库，并存放到 PyArmor 安装的路径之下（通常情况下和 pyarmor.py 在相同目录）。

如果需要在其他平台使用 PyArmor，请发送邮件到 <jondy.zhao@gmail.com>

.. _预安装的动态库清单:

预安装的动态库清单
------------------

.. list-table:: 表-1. 预安装的动态库清单
   :widths: 10 10 20 60
   :header-rows: 1

   * - 操作系统
     - CPU架构
     - 下载
     - 说明
   * - Windows
     - i686
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/platforms/win32/_pytransform.dll>`
     - 在 Cygwin 环境使用 i686-pc-mingw32-gcc 交叉编译
   * - Windows
     - AMD64
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/platforms/win_amd64/_pytransform.dll>`
     - 在 Cygwin 环境使用 x86_64-w64-mingw32-gcc 交叉编译
   * - Linux
     - i686
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/linux_i386/_pytransform.so>`
     - 使用 GCC 编译
   * - Linux
     - x86_64
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/linux_x86_64/_pytransform.so>`
     - 使用 GCC 编译
   * - MacOSX
     - x86_64, intel
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/platforms/macosx_x86_64/_pytransform.dylib>`
     - 使用 CLang 编译（MacOSX10.11）
   * - Linux
     - armv7
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/armv7/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 GCC 交叉编译
       32-bit Armv7 Cortex-A, hard-float, little-endian
   * - Linux
     - aarch64
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/armv8.64-bit/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 GCC 交叉编译
       64-bit Armv8 Cortex-A, little-endian

.. _其他支持平台的动态库清单:

其他支持平台的动态库清单
------------------------

.. list-table:: 表-2. 其他支持平台的动态库清单
   :widths: 10 10 20 60
   :header-rows: 1

   * - OS
     - Arch
     - Download
     - Description
   * - Windows
     - x86
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/platforms/vs2015/x86/_pytransform.dll>`
     - 使用 VS2015 编译
   * - Windows
     - x64
     - `_pytransform.dll <http://pyarmor.dashingsoft.com/downloads/platforms/vs2015/x64/_pytransform.dll>`
     - 使用 VS2015 编译
   * - Linux
     - armv5
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/armv5/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 GCC 交叉编译
       32-bit Armv5 (arm926ej-s)
   * - Linux
     - aarch32
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/armv8.32-bit/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 GCC 交叉编译
       32-bit Armv8 Cortex-A, hard-float, little-endian
   * - Linux
     - ppc64le
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/ppc64le/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) `gcc-powerpc64le-linux-gnu` 交叉编译
       适用于 POWER8
   * - iOS
     - arm64
     - `_pytransform.dylib <http://pyarmor.dashingsoft.com/downloads/platforms/ios.arm64/_pytransform.dylib>`
     - 使用 CLang 编译（iPhoneOS9.3sdk）
   * - FreeBSD
     - x86_64
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/freebsd/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 GCC 交叉编译
       不支持获取硬盘序列号
   * - Alpine Linux
     - x86_64
     - `_pytransform.so <http://pyarmor.dashingsoft.com/downloads/platforms/alpine/_pytransform.so>`
     - 在 Ubuntu 16.04 (x86_64) 使用 `musl-cross-make <https://github.com/richfelker/musl-cross-make>` 交叉编译（musl-1.1.21）