==========================
 详解可独立运行的加密脚本
==========================

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 的核心功能是加密脚本，加密完成之后生成可以独立运行的包完全是调用 PyInstaller_ 的相关功能。如果没有安装 PyInstaller_ 必须首先安装::

    $ pip install pyinstaller

PyInstaller_ 需要通过分析脚本源代码找到所有的依赖模块和包，并把这些依赖也自动的打到包里面。一旦加密之后， PyInstaller_ 就无法通过脚本找到依赖项目，所以直接打包加密脚本，执行的时候会导致模块找不到的问题。

为了解决这个问题，Pyarmor 8.0 提供了一个选项 :option:`--pack` ，通过对 PyInstaller_ 的打包过程进行一些特殊的处理，帮助 PyInstaller_ 正确找到加密脚本的所有依赖项目。

选项 :option:`--pack` 接受三种类型的值

- `onefile` 打包成为单个可执行文件
- `onedir`  打包到一个目录
- specfile  以 ``.spec`` 为后缀的文件名称

其中前两者适用于对 PyInstaller_ 不太了解和熟悉的用户使用，这是 PyInstaller_ 提供的两种打包模式，单文件和单目录。

后者则是针对已经能够使用 PyInstaller_ 的 specfile 进行打包的用户

自动打包模式
============

假设有一个这样的项目，其目录结构如下::

    project/
        ├── foo.py
        ├── queens.py
        ├── foo.spec
        └── joker/
            ├── __init__.py
            ├── queens.py
            └── config.json

我们使用下面的命令进行打包::

    $ cd project
    $ pyarmor gen --pack onefile foo.py

那么，Pyarmor 是如何生成一个包含加密脚本的可执行文件呢？

1. 首先 Pyarmor 会分析没有加密的脚本 `foo.py` 的代码，发现它需要导入模块 `queens.py` 和包 `joker`
2. 接下来 Pyarmor 会加密这三项到一个临时目录 `.pyarmor/pack/dist`
3. 然后 Pyarmor 调用 PyInstaller_ ，让它分析没有加密脚本的 `foo.py` 依赖关系，把 `foo.py` 使用到的所有系统包都记录保存下来
4. 最后 Pyarmor 再次调用 PyInstaller_ ，把临时目录 `.pyarmor/pack/dist` 下面的加密脚本，以及上一个步骤中发现的系统包 [#]_ 统统打包到一个文件里面，最后输出一个可执行文件 `dist/foo` 。

现在，让我们运行一下最终输出的可执行文件 `dist/foo` 或者 `dist/foo.exe`::

    $ ls dist/foo
    $ dist/foo

如果需要生成单个目录的包，只需要传递 `onedir` 给 :option:`--pack` 即可::

    $ pyarmor gen --pack onedir foo.py
    $ ls dist/foo
    $ dist/foo/foo

.. [#] 系统包没有进行加密

使用 specfile 进行打包
----------------------

在上面的示例项目中已经有一个 ``foo.spec`` 文件，可以打包没有加密的脚本为单个可执行文件，例如::

    $ pyinstaller foo.spec
    $ dist/foo

在这种情况下，可以直接把 ``foo.spec``  传递给 :option:`--pack` ，例如::

    $ pyarmor gen --pack foo.spec -r foo.py joker/

1. Pyarmor 首先根据加密选项对脚本进行加密，并保存到 `.pyarmor/pack/dist`
2. 然后读取 ``foo.spec`` 并创建一个补丁文件 ``foo.patched.spec`` ，这个补丁可以在打包的过程中使用加密脚本替换原来的脚本
3. 最后自动调用 PyInstaller_ ，使用这个打过补丁的 spec 文件来打包加密脚本

.. important::

   使用 specfile 打包的模式，无法自动加密依赖的脚本，需要人工在命令行指出需要加密的所有脚本，否则只有主脚本被加密。

检查打包的脚本是否被加密
------------------------

在脚本 ``foo.py`` 或者 ``joker/__init__.py`` 增加一行

.. code-block:: python

    print('this is __pyarmor__', __pyarmor__)

如果被加密了，那么可以正确打印出来。如果没有加密，就会抛出异常，因为只有加密脚本中才有内置名称 ``__pyarmor__`` 的定义。

使用其他 PyInstaller 选项
-------------------------

如果需要为应用程序增加图标，不显示控制台窗口等，只需要把 PyInstaller_ 的相关选项通过配置项 ``pack:pyi_options`` 传递给 Pyarmor 即可。

例如，使用 PyInstaller_ 选项 ``-w`` 不显示控制台窗口::

    $ pyarmor cfg pack:pyi_options = "-w"

接下来我们添加另外一个选项 ``-i`` 设置图标，需要注意的是在选项 ``-i`` 和其值之间必须使用一个空格进行分隔，不要使用等号 ``=`` 。例如::

    $ pyarmor cfg pack:pyi_options ^ "-i favion.ico"

在添加另外一个选项 ``--add-data``::

    $ pyarmor cfg pack:pyi_options ^ "--add-data joker/config.json:joker"

.. note::

    在 Windows 的命令行，也许需要使用双引号把 ``^`` 包含起来。例如::

        C:\Projects\build> pyarmor cfg pack:pyi_options "^" "-i favion.ico"

.. seealso:: :ref:`pyarmor cfg`

使用更多的加密选项
------------------

在 Darwin 系统，如果想让加密脚本能够同时工作在 Intel 和 Apple M1 框架下，可以传递额外的加密选项 ``--platform darwin.x86_64,darwin.arm64``::

    $ pyarmor gen --pack onefile --platform darwin.x86_64,darwin.arm64 foo.py

其他加密选项也都可以根据需要选用来增加安全性或者提高性能，例如::

    $ pyarmor gen --pack onefile --private foo.py

需要注意的是不是所有的选项都可以和 :option:`--pack` 一起使用，例如，使用 :option:`--restrict` 选项会导致加密的包出现保护异常。

自己动手打包加密脚本
====================

如果使用上面的方式出现问题，或者打包好的可执行文件出现执行错误，那么请使用下面的方式自己打包加密脚本。

这需要你了解 `如何使用 PyInstaller`__ 和 `如何使用 spec file`__ ，如果还不知道如何使用，请点击链接学习相关知识。

__ https://pyinstaller.org/en/stable/usage.html
__ https://pyinstaller.org/en/stable/spec-files.html

下面我们使用一个例子来说明如何手动打包加密脚本 ``/path/to/src/foo.py``

* 首先使用 Pyarmor 加密这个脚本 [#]_::

    $ cd project
    $ pyarmor gen -O obfdist -r foo.py joker/

* 然后使用下面的命令生成生成 ``foo.spec`` [#]_::

    $ pyi-makespec --onefile foo.py

* 接着修改 ``foo.spec`` ，插入下面的补丁代码到 ``pyz = PYZ`` 之前，这一步是重点

.. code-block:: python

    a = Analysis(
        ...
    )

    # Pyarmor patch start:

    obfpath = r'/path/to/obfdist'
    srcpath = r'/path/to/src'
    rtpkg = 'pyarmor_runtime_000000'
    rtext = 'pyarmor_runtime.so'

    def pyarmor_patcher(src, obfdist):

        # Make sure both of them are absolute paths
        src = os.path.abspath(src)
        obfdist = os.path.abspath(obfdist)

        count = 0
        for i in range(len(a.scripts)):
            if a.scripts[i][1].startswith(src):
                x = a.scripts[i][1].replace(src, obfdist)
                if os.path.exists(x):
                    a.scripts[i] = a.scripts[i][0], x, a.scripts[i][2]
                    count += 1
        if count == 0:
            raise RuntimeError('No obfuscated script found')

        for i in range(len(a.pure)):
            if a.pure[i][1].startswith(src):
                x = a.pure[i][1].replace(src, obfdist)
                if os.path.exists(x):
                    if hasattr(a.pure, '_code_cache'):
                        with open(x) as f:
                            a.pure._code_cache[a.pure[i][0]] = compile(
                                f.read(), a.pure[i][1], 'exec')
                    a.pure[i] = a.pure[i][0], x, a.pure[i][2]

    a.pure.append((
        rtpkg,
        os.path.join(obfpath, rtpkg, '__init__.py'),
        'PYMODULE'
    ))
    a.binaries.append((
        os.path.join(rtpkg, rtext),
        os.path.join(obfpath, rtpkg, rtext),
        'EXTENSION'
    ))
    pyarmor_patcher(srcpath, obfpath)

    # Pyarmor patch end.

    pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

* 最后直接使用打过补丁的 ``foo.spec`` 来打包，使用选项 `--clean` 避免补丁因为缓存的文件而失效::

    $ pyinstaller --clean foo.spec

请根据你的具体情况，做如下修改

* 使用实际目录替换 ``/path/to/src`` ，相对路径即可。例如当前路径，直接设置为空字符串
* 使用实际目录替换 ``/path/to/obfdist`` ，相对路径即可
* 使用实际名称替换 ``pyarmor_runtime_000000``
* 如果是 Windows 系统，需要替换 ``pyarmor_runtime.so`` 为 ``pyarmor_runtime.pyd``

**如何验证打包进去的是加密脚本**

方法一，可以在 ``foo.spec`` 的补丁代码增加一些 print 语句，验证加密脚本已经替换了原来的脚本

方法二，可以在主脚本增加一些调试语句进行判断，例如

.. code-block:: python

    print('this is __pyarmor__', __pyarmor__)

如果不是加密脚本，这个语句会报错，只有在加密脚本中才能正常打印。

.. rubric:: 备注

.. [#] 不要使用选项 :option:`-i` 和 :option:`--prefix` 加密脚本，其他选项可以尝试
.. [#] 其他 `PyInstaller`_ 的选项也可以在这里使用

替换打包模式
============

.. deprecated:: 8.5.4

    现在推荐使用选项 :option:`--pack` 的 ``onefile`` 或者 ``onedir`` 模式

首先调用 PyInstaller_ 将脚本打包成为单独的可执行文件或者打包到一个目录，然后把打包生成的可执行文件通过选项 :option:`--pack` 传递给 :ref:`pyarmor gen` 来实现::

    $ pyinstaller foo.py
    $ pyarmor gen --pack dist/foo/foo foo.py

Pyarmor 首先加密脚本，并把它们被存放到 :file:`.pyarmor/pack/dist` ，然后进行下面的额外处理:

* 提取可执行文件中内容到一个临时目录 :file:`.pyarmor/pack/`
* 使用加密脚本替换临时目录中同名的未加密脚本
* 把加密脚本的 :term:`运行辅助文件` 增加到临时目录中
* 根据把临时目录中所有内容重新生成可执行文件，并替换原来的可执行文件

需要注意的是，在这种方式下面，使用 PyInstaller 6.0+ 打包生成的可执行文件无法被正确处理，只能使用低版本 PyInstaller 进行打包。

.. important::

   只有命令行列出的脚本会被加密，如果需要加密其他脚本或者子目录，在命令行列出它们。例如::

      $ pyarmor gen --pack dist/foo/foo -r *.py dir1 dir2 ...

Apple M1 的 segment fault
=========================

在 Apple M1 上打包，如果生产的加密脚本运行时候发生崩溃，请首先检查运行辅助包的签名::

    $ codesign -v dist/foo/pyarmor_runtime_000000/pyarmor_runtime.so

如果签名非法，请重新进行签名::

    $ codesign -f -s dist/foo/pyarmor_runtime_000000/pyarmor_runtime.so

如果使用了 :option:`--enable-bcc` 或者 :option:`--enable-jit` 进行加密，那么还需要启用 `Allow Execution of JIT-compiled Code Entitlement`__

.. seealso:: `Using the latest code signature format`__

__ https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_allow-jit
__ https://developer.apple.com/documentation/xcode/using-the-latest-code-signature-format/

.. include:: ../_common_definitions.txt
