.. highlight:: console

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

.. customize-obfuscated-script
.. hidden-outer-runtime-key
.. check-debugger-by-yourself
.. protect-script-by-yourself

.. include:: ../_common_definitions.txt
