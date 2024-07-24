==========
 基础教程
==========

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

本教程仅适用于 Pyarmor 8.0+，使用下面的命令来查看版本信息::

    $ pyarmor --version

如果 Pyarmor 的版本小于 8，那么请选择相应版本的文档（在页面的左下角可以选择版本），或者升级 Pyarmor 到正确版本。

在整个教程中，使用的例子结构如下::

    project/
        ├── foo.py
        ├── queens.py
        └── joker/
            ├── __init__.py
            ├── queens.py
            └── config.json

Pyarmor 使用 :ref:`pyarmor gen` 加密不同的脚本，它提供了丰富的选项，以满足不同应用关于性能和安全方面的各种需求。

这里仅仅介绍的是常用的一些功能，关于完整的命令选项请查阅 :doc:`../reference/man`

调试模式和跟踪日志
==================

当加密过程出现问题的时候，检查控制台的输出日志可以帮助发现问题所在，而使用选项 ``-d`` 启用调试模式会打印更多的信息帮助发现错误::

    $ pyarmor -d gen foo.py

跟踪日志用来记录那些函数被 Pyarmor 使用什么方式进行了保护，它通过下面的方式启用::

    $ pyarmor cfg enable_trace=1

启用之后，每一次执行命令 :ref:`pyarmor gen` 都会生成一个跟踪日志文件 :file:`.pyarmor/pyarmor.trace.log` 记录相关的保护信息。例如::

    $ pyarmor gen foo.py
    $ cat .pyarmor/pyarmor.trace.log

    trace.co             foo:1:<module>
    trace.co             foo:5:hello
    trace.co             foo:9:sum2
    trace.co             foo:12:main

每一行开头的 ``trace.co`` 表示是默认的加密模式，后面的函数名称表示该函数使用默认的方式进行加密。

使用下面的方式禁用跟踪日志::

    $ pyarmor cfg enable_trace=0

.. program:: pyarmor gen

使用更多的选项加密脚本
======================

对于脚本，可以使用这些选项来增加安全性::

    $ pyarmor gen --enable-jit --mix-str --assert-call --private foo.py

选项 :option:`--enable-jit` 通过使用动态指令生成技术处理某些敏感数据来增加安全性。

选项 :option:`--mix-str` [#]_ 能够加密脚本中所有长度大于 8 的字符串。

选项 :option:`--assert-call` 能够确保加密脚本中的函数不会被替换。

选项 :option:`--private` 能够确保加密脚本的属性不能直接被 Python 解释器直接导入查看。

例如，

.. code-block:: python
    :emphasize-lines: 1,10

    data = "abcdefgxyz"

    def fib(n):
        a, b = 0, 1
        while a < n:
            print(a, end=' ')
            a, b = b, a+b

    if __name__ == '__main__':
        fib(n)

字符串常量 ``abcdefgxyz`` 和函数 ``fib`` 会被以如下方式进行保护

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

如果函数 ``fib`` 是加密函数，那么 ``__assert_call__(fib)`` 返回原来的函数，否则抛出保护异常。

为了查看那些函数和字符串被保护，可以启用并检查跟踪日志::

    $ pyarmor cfg enable_trace=1
    $ pyarmor gen --mix-str --assert-call fib.py
    $ cat .pyarmor/pyarmor.trace.log

    trace.assert.call    fib:10:'fib'
    trace.mix.str        fib:1:'abcxyz'
    trace.mix.str        fib:9:'__main__'
    trace.co             fib:1:<module>
    trace.co             fib:3:fib

.. [#] :option:`--mix-str` 在试用版中不可用

使用更多的选项加密包
====================

如果是加密 :term:`Python 包` ，使用其他两个选项::

    $ pyarmor gen --enable-jit --mix-str --assert-call --assert-import --restrict joker/

选项 :option:`--assert-import` 可以检查导入的模块，确保是没有被替换的加密模块。

选项 :option:`--restrict` 可以确保加密模块只能在加密脚本内部使用，而不能被外部脚本导入。

默认情况下 ``__init__.py`` 中定义的函数和名称是可以被外部脚本使用的，其他模块定义的函数和名称是不能被外部脚本使用。让我们创建一个测试脚本 :file:`dist/a.py` 来验证一下

.. code-block:: python

    import joker
    print('import joker OK')
    from joker import queens
    print('import joker.queens OK')

运行这个测试脚本::

    $ cd dist
    $ python a.py
    ... import joker OK
    ... RuntimeError: unauthorized use of script

如果需要导出包中的其他模块，要么不使用选项 :option:`--restrict` ，要么单独配置模块 ``joker.queens`` 不使用约束模式::

    $ pyarmor cfg -p joker.queens restrict_module=0

再次加密和测试一下，这次应该可以正常运行::

    $ pyarmor gen --restrict joker/

    $ cd dist/
    $ python a.py
    ... import joker OK
    ... import joker.queens

拷贝数据文件
============

很多包都有数据文件，并且运行的时候需要这些数据文件，但是默认情况下不会被 Pyarmor 拷贝到输出路径。

但是 Pyarmor 提供了三种方法可以解决这个问题

1. 在加密之前先把整个包全部拷贝到输出目录，然后加密脚本，这样加密脚本仅仅覆盖原来的 ``.py`` 文件，而数据文件保留不变::

     $ mkdir dist/joker
     $ cp -a joker/* dist/joker
     $ pyarmor gen -O dist -r joker/

2. 通过配置选项，让 Pyarmor 自动拷贝所有数据文件::

     $ pyarmor cfg data_files=*
     $ pyarmor gen -O dist -r joker/

如果只需要拷贝 ``*.yaml`` 和 ``*.json`` 文件，使用下面的配置命令::

     $ pyarmor cfg data_files="*.yaml *.json"

3. 自己编写 :term:`加密插件` 来拷贝需要的数据文件

周期性检查运行密钥
==================

使用下面的命令生成的加密脚本，运行的时候会每隔一个小时对运行密钥进行一次检查::

    $ pyarmor gen --period 1 foo.py

绑定加密脚本到多个设备
======================

使用选项 :option:`-b` 多次可以绑定加密脚本到多个设备。

例如，两台设备 A 和 B，以太网地址分别是 ``66:77:88:9a:cc:fa`` 和 ``f8:ff:c2:27:00:7f`` ，使用下面的命令绑定加密脚本到这两台设备::

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

外部运行密钥也可以存放在其他路径，请参阅 :term:`外部密钥` 中的说明

如何本地化错误信息
==================

运行错误信息可以使用本地语言进行替换定制，例如加密脚本过期之后可以提示用户自己设定的错误信息。

首先在当前目录创建子目录 :file:`.pyarmor` ，然后在其中创建文件 :file:`messages.cfg`::

    $ mkdir .pyarmor
    $ vi .pyarmor/messages.cfg

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

生成可独立运行的加密脚本
========================

这里的打包是指生成可以在没有 Python 环境独立运行的可执行文件。

Pyarmor 需要使用 PyInstaller_ 打包好的可执行文件，请首先安装 PyInstaller_::

    $ pip install pyinstaller

单个可执行文件模式
------------------

.. versionadded:: 8.5.4

生成一个单独的可执行文件直接使用下面的命令即可::

    $ pyarmor gen --pack onefile foo.py

这个命令会加密 ``foo.py`` ，然后加密和 ``foo.py`` 同目录的，并且被引用到的其他模块或者包，最后调用 PyInstaller_ 把加密脚本打包成为一个单独的可执行文件 ``dist/foo``::

    $ ls dist/foo
    $ dist/foo

需要注意的是对于脚本引用到系统模块，以及任何不在当前目录下的模块和包，都没有进行加密处理。

.. important::

   在命令行中的脚本 ``foo.py`` 必须是没有加密的，加密的脚本是无法直接打包的

单个目录模式
------------

.. versionadded:: 8.5.4

生成单个目录模式的包直接使用下面的命令即可::

    $ pyarmor gen --pack onedir foo.py

这个命令基本和上面的命令类似，只是在最后调用 PyInstaller_ 的时候传入的是单个目录的模式。查看最后的输出目录::

    $ ls dist/foo
    $ dist/foo/foo

使用 spec 文件打包加密脚本
--------------------------

.. versionadded:: 8.5.8

如果已经有现成的 spec 文件能够成功打包没有加密的脚本，例如::

    $ pyinstaller foo.spec
    $ dist/foo

可以直接把 ``foo.spec``  传递给 :option:`--pack` ，来自动打包加密脚本。例如::

    $ pyarmor gen --pack foo.spec -r foo.py joker/
    $ dist/foo

在这种模式无法自动加密依赖的脚本，需要人工在命令行指出需要加密的所有脚本，否则不会被加密。

如果打包出现问题，或者需要使用更多打包方面的功能，请参阅 :doc:`../topic/repack`

.. include:: ../_common_definitions.txt
