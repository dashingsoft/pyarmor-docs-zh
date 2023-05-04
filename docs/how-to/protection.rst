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

- 使用操作系统提供的签名验证方式确保可执行文件和动态库没有被替换和修改
- 使用第三方的可执行文件的保护工具例如 VMProtect 等保护 Python 以及 pyarmor_runtime.pyd/.so
- Pyarmor 提供了一些保护选项，能够绑定脚本到解释器，发现调试器（弱）就退出等功能
- Pyarmor 提供了脚本补丁，可以用来添加自定义的函数，进行检测调试器等各种自定义的保护，这种需要专业的能力，但是能够提供足够高的安全性，没有人知道用户自定义的保护代码，哪怕是简单的保护，对任何一个黑客来说都需要花费时间去破解。而对于那些公共的保护技术，往往有成熟的工具可以直接绕过

..
  - 对于 Windows 用户，pyarmor 提供了选项 ``--enable-themida`` 可以有效保护动态库实现反调试

第二方面的保护主要有 Pyarmor 提供，目前保护数据方面最安全的模式是使用 pack 和约束模式，能够保证无法直接通过 Python 自身的机制来获取运行时刻的数据。

**基本的实现步骤**

首先是使用 PyInstaller 打包::

    $ pyinstaller foo.py

接着使用下面的命令加密脚本::

    $ pyarmor cfg check_debugger=1 check_interp=1
    $ pyarmor cfg assert.call:auto_mode="or" assert.call:includes = "*"
    $ pyarmor cfg assert.call:auto_mode="or" assert.call:includes = "*"

    $ pyarmor gen --mix-str --assert-call --assert-import --restrict --pack dist/foo/foo foo.py

然后使用其他方式来保护 :file:`dist/foo/` 目录下面所有的可执行文件和动态库，外部工具要确保动态库不能被替换以及在运行时候的内存代码不能被修改。

典型的外部工具有 codesign，VMProtect 等。

.. rubric:: 备注

如果直接使用 PyInstaller 打包成为单个可执行文件，这个文件在执行的时候会解压到一个临时目录下面执行，需要保证第三方工具或者签名工具对解压后的文件也能够提供保护，否则是没有保护效果的，因为真正的动态库等相关文件都在这个解压后的目录下面。

**自定义补丁**

.. versionadded:: 8.x

                  该功能尚未实现

用户可以编写自己的代码去检查调试器和其他任何反调试代码，代码可以使用 Python 和 C 实现，在扩展模块 pyarmor_runtime 被装载的时候自动调用。

基本配置方式是创建一个脚本  :file:`.pyarmor/hooks/pyarmor_runtime.py` 或者文件 :file:`.pyarmor/hooks/pyarmor_runtime.c` ，这个脚本会被嵌入到运行密钥中，并且在扩展模块 pyarmor_runtime 初始化的时候运行。


.. include:: ../_common_definitions.txt
