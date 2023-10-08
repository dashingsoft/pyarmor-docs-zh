==========================
 详解可独立运行的加密脚本
==========================

.. highlight:: console

.. program:: pyarmor gen

使用 Pyarmor 8.0 生成可以独立运行的加密脚本，必须首先调用 PyInstaller_ 将脚本打包成为单独的可执行文件或者打包到一个目录，然后把打包生成的可执行文件通过选项 :option:`--pack` 传递给 :ref:`pyarmor gen` 来实现::

  pyinstaller foo.py
  pyarmor gen --pack dist/foo/foo foo.py

如果没有选项 :option:`--pack` ，只是把脚本进行加密。至于那些脚本被加密，请参考 :ref:`pyarmor gen` 中说明，这里不会自动加密依赖项。

如果有选项 :option:`--pack` ，那么默认输出目录是 :file:`.pyarmor/pack/dist` ，在加密完成之后，还进行下面的额外处理:

* 提取可执行文件中内容到一个临时目录 :file:`.pyarmor/pack/`
* 使用加密脚本替换临时目录中同名的未加密脚本
* 把加密脚本的 :term:`运行辅助文件` 增加到临时目录中
* 根据把临时目录中所有内容重新生成可执行文件，并替换原来的可执行文件

需要注意的是只有命令行列出的脚本会被加密，如果需要加密其他脚本或者子目录，在命令行列出它们。例如::

  pyarmor gen --pack dist/foo/foo -r *.py dir1 dir2 ...

在 Darwin 平台，如果需要生成支持 Intel 和 Apple Silicon 的加密脚本，使用额外选项 ``--platform darwin.x86_64,darwin.arm64`::

  pyarmor gen --pack dist/foo/foo --platform darwin.x86_64,darwin.arm64 foo.py

**自己动手打包加密脚本**

如果使用上面的方式出现问题，或者打包好的可执行文件出现执行错误，那么请使用下面的方式自己打包加密脚本。

这需要你了解 `如何使用 PyInstaller`__ 和 `如何使用 spec file`__ ，如果还不知道如何使用，请点击链接学习相关知识。

__ https://pyinstaller.org/en/stable/usage.html
__ https://pyinstaller.org/en/stable/spec-files.html

下面我们使用一个例子来说明如何手动打包加密脚本 ``/path/to/src/foo.py``

* 首先使用 Pyarmor 加密这个脚本 [#]_::

    cd /path/to/src
    pyarmor gen -O obfdist -a foo.py

* 然后把 :term:`运行辅助包` 移到当前目录 [#]_::

    mv obfdist/pyarmor_runtime_000000 ./

* 如果已经有 ``foo.spec`` ，需要把运行辅助包增加到 ``hiddenimports``

.. code-block:: python

    a = Analysis(
        ...
        hiddenimports=['pyarmor_runtime_000000'],
        ...
    )

* 如果还没有这个文件，使用下面的命令生成 ``foo.spec`` [#]_::

    pyi-makespec --hidden-import pyarmor_runtime_000000 foo.py

* 修改 ``foo.spec`` ，插入补丁代码到 ``a = Analysis`` 之后，这一步是重点

.. code-block:: python

    a = Analysis(
        ...
    )

    # Pyarmor patch start:

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
                            a.pure._code_cache[a.pure[i][0]] = compile(f.read(), a.pure[i][1], 'exec')
                    a.pure[i] = a.pure[i][0], x, a.pure[i][2]

    pyarmor_patcher(r'/path/to/src', r'/path/to/obfdist')

    # Pyarmor patch end.

    pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

* 最后直接使用打过补丁的 ``foo.spec`` 来打包，同时使用选项 `--clean` 避免补丁因为缓存的文件而失效::

    pyinstaller --clean foo.spec

请根据你的具体情况，做如下修改

* 使用实际目录替换 ``/path/to/src`` 和 ``/path/to/obfdist``
* 使用实际名称替换 ``pyarmor_runtime_000000``

**如何验证打包进去的是加密脚本**

方法一，可以在 ``foo.spec`` 的补丁代码增加一些 print 语句，验证加密脚本已经替换了原来的脚本

方法二，可以在主脚本增加一些调试语句进行判断，例如

.. code-block:: python

    print('this is __pyarmor__', __pyarmor__)

如果不是加密脚本，这个语句会报错，只有在加密脚本中才能正常打印。

.. rubric:: 备注

.. [#] 不要使用选项 :option:`-i` 和 :option:`--prefix` 加密脚本，其他选项可以尝试
.. [#] 这一步的目的只是为了方便让 `PyInstaller` 能够找到运行辅助包而不需要增加额外路径
.. [#] 其他 `PyInstaller`_ 的选项也可以在这里使用

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
