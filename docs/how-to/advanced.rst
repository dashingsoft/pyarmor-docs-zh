.. highlight:: console

========================
 解决加密过程中编码错误
========================

使用下面的命令可以设置加密脚本使用的编码为 ``utf-8``::

    $ pyarmor cfg encoding=utf-8

使用下面的命令可以设置定制的错误消息文件使用的编码为 ``gbk``::

    $ pyarmor cfg messages=messages.cfg:gbk

======================
 删除脚本中 Docstring
======================

使用下面的配置可以删除加密脚本中 DocString::

    $ pyarmor cfg optimize 2

.. customize-obfuscated-script
.. hidden-outer-runtime-key
.. check-debugger-by-yourself
.. protect-script-by-yourself

.. include:: ../_common_definitions.txt
