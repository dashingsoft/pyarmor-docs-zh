========================
本地化和国际化的工作原理
========================

加密过程
========

查找本地配置目录下面的 :file:`messages.cfg`
没有则查找全局配置目录下面的 :file:`messages.cfg`

如果存在定制的消息文件，那么把定制的消息存到运行辅助包里面

打开消息文件的编码方式默认是 ``utf-8`` ，如果需要使用其他编码方式，使用下面的命令进行配置::

    $ pyarmor cfg messages=messages.cfg:gbk

.. seealso:: :ref:`runtime errors`

运行加密脚本的过程
==================

确定默认语言

* 首先查看 :envvar:`PYARMOR_LANG`
* 其次查看 :attr:`sys._PARLANG`
* 最后查看 :envvar:`LANG`

查找定制的错误信息，首先匹配语言，然后匹配错误代码

没有找到则使用默认的错误信息

.. include:: ../_common_definitions.txt
