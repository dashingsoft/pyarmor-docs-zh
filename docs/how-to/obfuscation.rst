.. highlight:: console

==========================
 打包脚本的时候保护系统库
==========================

.. versionadded:: 8.2

Pyarmor 对使用 PyInstaller 打包进行发布的方式提供了特别的保护，这种模式下面可以对系统库进行加密和保护，确保其不能被替换。下面是必须使用的选项::

    $ pyinstaller foo.py
    $ pyarmor gen --assert-call --assert-import --restrict --pack dist/foo/foo foo.py

其他选项可以根据需要进行配置。

.. seealso:: :doc:`protection`

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
