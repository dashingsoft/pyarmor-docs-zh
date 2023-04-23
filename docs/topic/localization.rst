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

所有支持的错误代码

.. code-block:: ini

  [runtime.message]

    error_1 = 脚本许可证已经过期
    error_2 = 脚本许可证不可用于当前设备
    error_3 = 非法使用脚本

    error_4 = 缺少运行许可文件
    error_5 = 脚本不支持当前 Python 版本
    error_6 = 脚本不支持当前系统
    error_7 = 加密模块的数据格式不正确

    error_8 = 加密函数的数据格式不正确


运行加密脚本的过程
==================

确定默认语言

* 首先查看 :envvar:`PYARMOR_LANG`
* 其次查看 :attr:`sys._PARLANG`
* 最后查看 :envvar:`LANG`

查找定制的错误信息，首先匹配语言，然后匹配错误代码

没有找到则使用默认的错误信息

.. include:: ../_common_definitions.txt
