==========================
 保护运行时刻的数据安全性
==========================

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 的核心功能是保护 Python 脚本无法被反编译，通过多种不可逆加密模式的实现，已经能够实现 Python 脚本无法使用任何方式完全反编译出来。但是对于内存数据，包括运行时刻的数据保护，Pyarmor 没有进行太多的保护。如果保护的重点是运行时刻的数据，那么就需要额外的工具来进行保护。

Pyarmor 可以确保各种使用 Python 自身提供的机制无法非法获取运行时刻的数据，但是前提是运行加密脚本的 Python 的解释器和扩展模块 pyarmor_runtime 不能被替换或者修改，尤其是在运行时刻使用调试器直接修改内存代码段，这些就是需要额外的方法和工具来提供保护。常见的保护方式有

- 使用操作系统提供的签名验证方式确保可执行文件和动态库没有被替换和修改
- 使用第三方的可执行文件的保护工具例如 VMProtect 等保护 Python 以及 pyarmor_runtime.pyd/.so
- Pyarmor 提供了一些保护选项，能够绑定脚本到解释器，发现调试器（弱）就退出等功能
- Pyarmor 提供了脚本补丁，可以用来添加自定义的函数，进行检测调试器等各种自定义的保护，这种需要专业的能力，但是能够提供足够高的安全性，没有人知道用户自定义的保护代码，哪怕是简单的保护，对任何一个黑客来说都需要花费时间去破解。而对于那些公共的保护技术，往往有成熟的工具可以直接绕过

..
  - 对于 Windows 用户，pyarmor 提供了选项 ``--enable-themida`` 可以有效保护动态库实现反调试

**基本的实现步骤**

需要明白的一点是，如果运行 Python 脚本的解释器可以被定制，那么运行时刻的 Python 数据就没有秘密可言，所以必须要保证运行加密脚本的解释器不能被替换。

能实现这一点的加密模式目前 Pyarmor 只提供使用选项 :option:`--pack` 的约束模式，然后使用外部工具保证生成的动态库不能被替换和修改，这样才能够确保无法直接通过 Python 自身的机制来获取运行时刻的数据。

首先配置下列选项 [#]_::

    $ pyarmor cfg check_debugger=1 check_interp=1

接着使用下面的命令加密脚本 [#]_::

    $ pyarmor gen --mix-str --assert-call --assert-import --private --pack onedir foo.py

然后使用其他方式来保护 :file:`dist/foo/` 目录下面所有的可执行文件和动态库，外部工具要确保动态库不能被替换以及在运行时候的内存代码不能被修改。

典型的外部工具有 codesign，VMProtect 等。

.. rubric:: 备注

.. [#] 不要在 Intel i686 系列的平台上使用配置项 ``check_interp`` ，这个选项无法在这些平台工作。

.. [#] 如果使用选项 ``onefile`` 打包成为单个可执行文件，是不可能实现真正的保护效果呢。因为这个文件在执行的时候会解压到一个临时目录下面执行，真正的动态库等相关文件都在这个解压后的目录下面，需要保护的是这些解压后的动态库。

**脚本补丁**

为了进一步提高安全性，还可以使用脚本补丁来检查 PyInstaller 的装载代码，确保其没有被替换。

下面的例子只是演示如何实现，请不要直接在项目中使用，并且在不同 PyInstaller 版本中可能会出错，请根据自己的具体情况，参考这个示例编写自己的私有补丁。

.. code-block:: python
    :linenos:
    :emphasize-lines: 12-14

    # Hook script ".pyarmor/hooks/foo.py"

    def protect_self():
        from sys import modules

        def check_module(name, checklist):
            m = modules[name]
            for attr, value in checklist.items():
                if value != sum(getattr(m, attr).__code__.co_code):
                    raise RuntimeError('unexpected %s' % m)

        checklist__frozen_importlib = {}
        checklist__frozen_importlib_external = {}
        checklist_pyimod03_importers = {}

        check_module('_frozen_importlib', checklist__frozen_importlib)
        check_module('_frozen_importlib_external', checklist__frozen_importlib_external)
        check_module('pyimod03_importers', checklist_pyimod03_importers)

    protect_self()

目前脚本里面的检查点为空（高亮的行），为了得到真正的检查点，需要先使用下面的一个假函数替换真正的 ``check_module``

.. code-block:: python

        def check_module(name, checklist):
            m = modules[name]
            refs = {}
            for attr in dir(m):
                value = getattr(m, attr)
                if hasattr(value, '__code__'):
                    refs[attr] = sum(value.__code__.co_code)
            print('    checklist_%s = %s' % (name, refs))


运行下面的命令以得到真正的检查点，代码行会打印在控制台::

    $ pyinstaller foo.py
    $ pyarmor gen --pack dist/foo/foo foo.py

    ...
    checklist__frozen_importlib = {'__import__': 9800, ...}
    checklist__frozen_importlib_external = {'_calc_mode': 2511, ...}
    checklist_pyimod03_importers = {'imp_lock': 183, 'imp_unlock': 183, ...}

编辑脚本补丁，恢复原来的函数 ``check_module`` 并使用生成的代码替换空的检查点。

最后使用真正的补丁脚本来生成最终的包::

    $ pyinstaller foo.py
    $ pyarmor gen --pack dist/foo/foo foo.py

**启动补丁**

.. versionadded:: 8.3

用户还可以编写自己的代码去检查调试器和其他任何反调试代码，代码可以使用 Python 实现，在扩展模块 pyarmor_runtime 被装载的时候自动调用。

基本配置方式是创建一个脚本  :file:`.pyarmor/hooks/pyarmor_runtime.py` ，定义一个函数 :func:`bootstrap` 来进行额外的检查。例如：

.. code-block:: python

   def bootstrap(user_data):
       from ctypes import windll
       if windll.kernel32.IsDebuggerPresent():
           print('found debugger')
           return False

..
  **无法自动打包的处理**

  有些 PyInstaller 的版本打包的文件 Pyarmor 无法识别，所以无法自动打包，这种情况下需要手动打包

  首先下载 `pyinstxtractor.py`__

  解压打包好的文件::

      $ python pyinstxtractor.py dist/foo/foo

  把解压的 pyz 目录也进行加密::

      $ pyarmor cfg check_debugger=1 check_interp=1
      $ pyarmor gen --mix-str --assert-call --assert-import --restrict --pack dist/foo/foo foo.py foo_extracted/PYZ-00.pyz

  __ https://github.com/extremecoders-re/pyinstxtractor/blob/master/pyinstxtractor.py

.. include:: ../_common_definitions.txt
