==========================
 保护运行时刻的数据安全性
==========================

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 的核心功能是保护 Python 脚本无法被反编译，通过多种不可逆加密模式的实现，已经能够实现 Python 脚本无法使用任何方式完全反编译出来。但是对于内存数据，包括运行时刻的数据保护，Pyarmor 没有进行太多的保护。

如果保护的重点是运行时刻的数据，那么就需要从两方面进行保护：

1. 保护 Python 进程不能被调试工具修改和控制
2. 保护各种使用 Python 自身提供的机制去非法获取数据

缺少任何一方面的保护，都无法真正实现运行时刻的数据安全。

第一方面的保护方式有

* 使用操作系统提供的签名验证方式确保可执行文件和动态库没有被替换和修改
* 使用第三方的可执行文件的保护工具例如 VMProtect 等保护 Python 以及 pyarmor_runtime.pyd/.so
* Pyarmor 提供了一些保护选项，能够绑定脚本到解释器，发现调试器（弱）就退出等功能
* Pyarmor 提供了脚本补丁，可以用来添加自定义的函数，进行检测调试器等各种自定义的保护，这种需要专业的能力，但是能够提供足够高的安全性，没有人知道用户自定义的保护代码，哪怕是简单的保护，对任何一个黑客来说都需要花费时间去破解。而对于那些公共的保护技术，往往有成熟的工具可以直接绕过

第二方面的保护主要有 Pyarmor 提供，目前保护数据方面最安全的模式是使用 pack 和约束模式，能够保证无法直接通过 Python 自身的机制来获取运行时刻的数据。

..
  首先是使用 PyInstaller 打包并配置加密环境::

      $ pyinstaller foo.py

      $ pyarmor cfg protect_system_package=1
      $ pyarmor cfg check_debugger=1
      $ pyarmor cfg self_contained=1
      $ pyarmor cfg bind_interps="xxxx"

  对于 Windows 平台，直接使用下面的命令进行保护::

      $ pyarmor gen --pack dist/foo --enable-themida foo.py

  一般不需要在使用其他第三方保护工具，当然如果能够满足性能需求，也可以使用第三方保护工具来进一步增加安全性。

  对于其他平台，使用下面的命令加密脚本::

      $ pyarmor gen --pack dist/foo foo.py

  然后使用第三方的可执行文件保护工具来保护生成的可执行文件和动态库。

.. include:: ../_common_definitions.txt
