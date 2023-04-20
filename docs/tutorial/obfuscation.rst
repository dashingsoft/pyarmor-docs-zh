==========
 基础教程
==========

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

We'll assume you have Pyarmor 8.0+ installed already. You can tell Pyarmor is installed and which version by running the following command in a shell prompt (indicated by the $ prefix)::

    $ pyarmor --version

If Pyarmor is installed, you should see the version of your installation. If it isn't, you'll get an error.

This tutorial is written for Pyarmor 8.0+, which supports Python 3.7 and later. If the Pyarmor version doesn't match, you can refer to the tutorial for your version of Pyarmor by using the version switcher at the bottom right corner of this page, or update Pyarmor to the newest version.

Throughout this tutorial, assume run :command:`pyarmor` in project path which includes::

    project/
        ├── foo.py
        ├── queens.py
        └── joker/
            ├── __init__.py
            ├── queens.py
            └── config.json

Pyarmor uses :ref:`pyarmor gen` with rich options to obfuscate scripts to meet the needs of different applications.

Here only introduces common options in a short, using any combination of them as needed. About usage of each option in details please refer to :ref:`pyarmor gen`

Debug mode and trace log
========================

When someting is wrong, check console log to find what Pyarmor does, and use :option:`-d` to enable debug mode to print more information::

    $ pyarmor -d gen foo.py

Trace log is useful to check what're protected by Pyarmor, enable it by this command::

    $ pyarmor cfg enable_trace=1

After that, :ref:`pyarmor gen` will generate a logfile :file:`.pyarmor/pyarmor.trace.log`. For example::

    $ pyarmor gen foo.py
    $ cat .pyarmor/pyarmor.trace.log

    trace.co             foo:1:<module>
    trace.co             foo:5:hello
    trace.co             foo:9:sum2
    trace.co             foo:12:main

Each line starts with ``trace.co`` is reported by code object protector. The first log says ``foo.py`` module level code is obfuscated, second says function ``hello`` at line 5 is obfuscated, and so on.

Enable both debug and trace mode could show much more information::

    $ pyarmor -d gen foo.py

Disable trace log by this command::

    $ pyarmor cfg enable_trace=0

.. program:: pyarmor gen

More options to protect script
==============================

For scripts, use these options to get more security::

    $ pyarmor gen --enable-jit --mix-str --assert-call --private foo.py

Using :option:`--enable-jit` tells Pyarmor processes some sentensive data by ``c`` function generated in runtime.

Using :option:`--mix-str` [#]_ could mix the string constant (length > 4) in the scripts.

Using :option:`--assert-call` makes sure function is obfuscated, to prevent called function from being replaced by special ways

Using :option:`--private` makes the script could not be imported by plain scripts

For example,

.. code-block:: python
    :emphasize-lines: 1,10

    data = "abcxyz"

    def fib(n):
        a, b = 0, 1
        while a < n:
            print(a, end=' ')
            a, b = b, a+b

    if __name__ == '__main__':
        fib(n)

String constant ``abcxyz`` and function ``fib`` will be protected like this

.. code-block:: python
    :emphasize-lines: 1,10

    data = __mix_str__(b"******")

    def fib(n):
        a, b = 0, 1
        while a < n:
            print(a, end=' ')
            a, b = b, a+b

    if __name__ == '__main__':
        __assert_call__(fib)(n)

If function ``fib`` is obfuscated, ``__assert_call__(fib)`` returns original function ``fib``. Otherwise it will raise protection exception.

To check which function or which string are protected, enable trace log and check trace logfile::

    $ pyarmor cfg enable_trace=1
    $ pyarmor gen --mix-str --assert-call fib.py
    $ cat .pyarmor/pyarmor.trace.log

    trace.assert.call    fib:10:'fib'
    trace.mix.str        fib:1:'abcxyz'
    trace.mix.str        fib:9:'__main__'
    trace.co             fib:1:<module>
    trace.co             fib:3:fib

.. [#] :option:`--mix-str` is not available in trial version

More options to protect package
===============================

For package, remove :option:`--private` and append 2 extra options::

    $ pyarmor gen --enable-jit --mix-str --assert-call --assert-import --restrict joker/

Using :option:`--assert-import` prevents obfsucated modules from being replaced with plain script. It checks each import statement to make sure the modules are obfuscated.

Using :option:`--restrict` makes sure the obfuscated module is only available inside package. It couldn't be imported from any plain script, also not be run by Python interpreter.

By default ``__init__.py`` is not restricted, all the other modules are invisible from outside. Let's check this, first create a script :file:`dist/a.py`

.. code-block:: python

    import joker
    print('import joker OK')
    from joker import queens
    print('import joker.queens OK')

Then run it::

    $ cd dist
    $ python a.py
    ... import joker OK
    ... RuntimeError: unauthorized use of script

In order to export ``joker.queens``, config this module is not restrict::

    $ pyarmor cfg -p joker.queens restrict_module=0

Then obfuscate this package with restrict mode::

    $ pyarmor gen --restrict joker/

Now do above test again, it should work::

    $ cd dist/
    $ python a.py
    ... import joker OK
    ... import joker.queens

Copying package data files
==========================

Many packages have data files, but they're not copied to output path.

There are 2 ways to solve this problem:

1. Before generating the obfuscated scripts, copy the whole package to output path, then run :ref:`pyarmor gen` to overwite all the ``.py`` files::

     $ mkdir dist/joker
     $ cp -a joker/* dist/joker
     $ pyarmor gen -O dist -r joker/

2. Changing default configuration let Pyarmor copy data files::

     $ pyarmor cfg data_files=*
     $ pyarmor gen -O dist -r joker/

Checking runtime key periodically
=================================

Checking runtime key every hour::

    $ pyarmor gen --period 1 foo.py

Binding to many machines
========================

Using :option:`-b` many times to bind obfuscated scripts to many machines.

For example, machine A and B, the ethernet addresses are ``66:77:88:9a:cc:fa`` and ``f8:ff:c2:27:00:7f`` respectively. The obfuscated script could run in both of machine A and B by this command ::

    $ pyarmor gen -b "66:77:88:9a:cc:fa" -b "f8:ff:c2:27:00:7f" foo.py

使用外部文件存放运行密钥
========================

在加密脚本的时候指定使用外部密钥::

    $ pyarmor gen --outer foo.py

在这种情况下，加密后的脚本无法直接运行::

    $ python dist/foo.py

需要先使用 :ref:`pyarmor gen key` 创建一个外部密钥::

    $ pyarmor gen key -e 3

上面的命令会生成外部密钥文件 ``dist/pyarmor.rkey`` ，拷贝这个文件到运行辅助包::

    $ cp dist/pyarmor.rkey dist/pyarmor_runtime_000000/

这样可以正常运行加密脚本 :file:`dist/foo.py`::

    $ python dist/foo.py

让我们在生成一个新的运行密钥文件，存放在另外一个目录::

    $ pyarmor gen key -O dist/key2 -e 10

    $ ls dist/key2/pyarmor.rkey

拷贝这个新文件到运行辅助包去替换原来的::

    $ cp dist/key2/pyarmor.rkey dist/pyarmor_runtime_000000/

外部运行密钥必须至少包含一个约束条件，要么是有效期，要么是设备信息。

外部运行密钥的名称默认是 ``pyarmor.rkey``

外部运行密钥必须存放在以下任意一个路径下面

1. 运行辅助包所在路径
2. 环境变量 :envvar:`PYARMOR_RKEY` 指定的路径
3. :attr:`sys._MEIPASS`  指定的路径
4. 当前路径

如果加密脚本在上面的任何一个路径都没有找到运行密钥文件，那么会报错退出

如何本地化错误信息
==================

运行错误信息可以使用本地语言进行替换定制，例如加密脚本过期之后可以提示用户自己设定的错误信息。

首先在当前目录创建子目录 :file:`.pyarmor` ，然后在其中创建文件 :file:`messages.cfg`::

    $ mkdir .pyarmor
    $ vi .pyarmor/message.cfg

编辑这个文件。这是一个 ``.ini`` 格式的文件，按照自己的需要修改 ``error_N`` 后面对应的错误信息

.. code-block:: ini

  [runtime.message]

    error_1 = 脚本许可证已经过期
    error_2 = 脚本许可证不可用于当前设备
    error_3 = 缺少运行许可文件
    error_4 = 非法使用脚本

    error_5 = 脚本不支持当前 Python 版本
    error_6 = 脚本不支持当前系统

    error_7 = 加密模块的数据格式不正确
    error_8 = 加密函数的数据格式不正确

然后需要重新加密脚本::

    $ pyarmor gen foo.py

如果你想所有的运行密钥错误都显示同样的错误信息，而其他类型的错误还是使用默认值，
那么，修改成为下面的样子

.. code-block:: ini

  [runtime.message]

    error_1 = 未授权使用脚本
    error_2 = 未授权使用脚本

然后重新加密脚本使之生效。

Packing obfuscated scripts
==========================

Pyarmor need PyInstaller to pack scripts first, then replace plain scripts with obfuscated ones in bundle.

Packing to one file
-------------------

First packing script to one file by PyInstaller with option ``-F``::

    $ pyinstaller -F foo.py

It generates one bundle file ``dist/foo``, pass this to pyarmor::

    $ pyarmor gen -O obfdist --pack dist/foo foo.py

This command will obfuscate ``foo.py`` first, then repack ``dist/foo``, replace the original ``foo.py`` with ``obfdist/foo.py``, and append all the runtime files to bundle.

The final output is still ``dist/foo``::

    $ dist/foo

Packing to one folder
---------------------

First packing script to one foler by PyInstaller::

    $ pyinstaller foo.py

It generates one bundle folder ``dist/foo``, and an executable file ``dist/foo/foo``, pass this executable to pyarmor::

    $ pyarmor gen -O obfdist --pack dist/foo/foo foo.py

Like above section, ``dist/foo/foo`` will be repacked with obfuscated scripts.

Now run it::

    $ dist/foo/foo

More information about pack feature, refer to :doc:`../topic/repack`

.. include:: ../_common_definitions.txt
