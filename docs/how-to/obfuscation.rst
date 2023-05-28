.. highlight:: console
.. program:: pyarmor gen

==========================
 打包脚本的时候保护系统库
==========================

.. versionadded:: 8.2
.. versionchanged:: 8.2.2
                    不在支持使用选项 :option:`--restrict` 组合 :option:`--pack` 来保护系统库。

Pyarmor 对使用 PyInstaller 打包进行发布的方式提供了特别的保护，这种模式下面可以对系统库进行加密和保护。其实现的思路就是把所有的依赖包也列出了进行加密。

下面是一个示例，说明如何加密一个脚本 ``foo.py`` 并同时保护系统依赖库.

我们需要使用 PyInstaller 提供的功能来列出 ``foo.py`` 所有的依赖库。

首先生成 ``foo.spec``::

    $ pyi-makespec foo.py

然后修改这个 ``foo.spec``:

.. code-block:: python

    a = Analysis(
        ...
    )

    # Patched by Pyarmor to generate file.list
    _filelist = []
    _package = None
    for _src in sort([_src for _name, _src, _type in a.pure]):
        if _src.endswith('__init__.py'):
            _package = _src.replace('__init__.py', '')
            _filelist.append(_package)
        elif _package is None:
            _filelist.append(_src)
        elif not _src.startswith(_package):
            _package = None
            _filelist.append(_src)
    with open('file.list', 'w') as _file:
        _file.write('\n'.join(_filelist))
    # End of patch

接下来使用这个修改后的文件打包 ``foo.py`` ，同时生成包含所有依赖库的文件 :file:`file.list`::

    $ pyinstaller foo.py

最后使用下面的选项加密脚本并重新打包::

    $ pyarmor gen --assert-call --assert-import --pack dist/foo/foo foo.py @file.list

这个例子只是说明了基本的实现方法和步骤，请根据自己的实际情况编写自己的补丁脚本和使用必要的加密选项。如有必要，还可以人工修改生成的依赖库文件 :file:`file.list`

========================
 解决加密过程中编码错误
========================

默认的脚本编码为 ``utf-8`` ，当加密脚本的时候出现编码错误，可以使用配置项指定正确的文件编码。例如下面的命令可以设置加密脚本使用的编码为 ``gbk``::

    $ pyarmor cfg encoding=gbk

同样也可以为定制的错误消息文件 ``messages.cfg`` 指定编码。例如，使用下面的命令可以设置定制的错误消息文件使用的编码为 ``gbk``::

    $ pyarmor cfg messages=messages.cfg:gbk

======================
 删除脚本中 Docstring
======================

使用下面的配置可以删除加密脚本中 DocString::

    $ pyarmor cfg optimize 2

配置项 ``optimize`` 可用值和作用请参考 `compile`__

__ https://docs.python.org/3.11/library/functions.html#compile

.. include:: ../_common_definitions.txt
