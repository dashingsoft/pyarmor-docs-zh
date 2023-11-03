==========
 高级教程
==========

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

.. _using rftmode:

使用 RFT 模式加密脚本 :sup:`pro`
================================

:term:`RFT 模式` 是一种不可逆的加密模式，它相当于把源代码重新写了一遍，把其中的函数，类，方法和变量的名称进行了重命名。

使用选项 :option:`--enable-rft` 启用 RTF 模式 [#]_::

    $ pyarmor gen --enable-rft foo.py

启用加密模式之后，这个脚本

.. code-block:: python
    :linenos:

    import sys

    def sum2(a, b):
        return a + b

    def main(msg):
        a = 2
        b = 6
        c = sum2(a, b)
        print('%s + %s = %d' % (a, b, c))

    if __name__ == '__main__':
        main('pass: %s' % data)

会被转换成为

.. code-block:: python
    :linenos:

    pyarmor__17 = __assert_armored__(b'\x83\xda\x03sys')

    def pyarmor__22(a, b):
        return a + b

    def pyarmor__16(msg):
        pyarmor__23 = 2
        pyarmor__24 = 6
        pyarmor__25 = pyarmor__22(pyarmor__23, pyarmor__24)
        pyarmor__14('%s + %s = %d' % (pyarmor__23, pyarmor__24, pyarmor__25))

    if __name__ == '__main__':
        pyarmor__16('pass: %s' % pyarmor__20)

RFT 模式不会修改模块属性 ``__all__`` 中列出的名称，也不会修改函数所有的参数名称。例如

.. code-block:: python
    :emphasize-lines: 3,5,9

    import re

    __all__ = ['make_scanner']

    def py_make_scanner(context):
        parse_obj = context.parse_object
        parse_arr = context.parse_array

    make_scanner = py_make_scanner

上面的脚本会转换成为

.. code-block:: python
    :emphasize-lines: 3,5,9

    pyarmor__3 = __assert_armored__(b'\x83e\x9d')

    __all__ = ['make_scanner']

    def pyarmor__1(context):
        pyarmor__4 = context.parse_object
        pyarmor__5 = context.parse_array

    make_scanner = pyarmor__1

如果需要查看脚本转换的结果，可以使用下面的命令启用跟踪 RFT 模式 [#]_::

    $ pyarmor cfg trace_rft=1
    $ pyarmor gen --enable-rft foo.py

启用这个功能之后转换后的脚本会保存到目录 ``.pyarmor/rft``::

    $ cat .pyarmor/rft/foo.py

现在运行加密后的脚本::

    $ python dist/foo.py

如果出现名称绑定的错误，可以尝试再次对其进行加密，这样可能会解决一些名称错误问题::

    $ pyarmor gen --enable-rft foo.py
    $ python dist/foo.py

如果重新两次加密之后还是无法运行加密脚本，请参考 :doc:`../topic/rftmode` 来解决名称错误问题。

.. [#] 这个功能仅在 :term:`Pyarmor 专家版` 中可用
.. [#] 这个功能仅用于 Python 3.9+

.. _using bccmode:

使用 BCC 模式加密脚本 :sup:`pro`
================================

BCC 模式会把部分函数直接转换成为二进制代码，在根本上避免被还原成为 Python 函数。

BCC 模式需要配置 :term:`C` 编译器，对于 `Linux` 和 `Darwin` 来说，一般不需要进行配置，只要默认的 ``gcc`` 和 ``clang`` 能工作就可以。在 `Windows` 环境下面，可以使用下面任意一种方式配置 ``clang.exe`` ，目前其它编译器还不支持:

* 如果已经有 ``clang.exe`` ，只要在其它路径直接运行 ``clang.exe`` 不出错就可以。如果文件存在，但是无法在任意路径直接运行，可以配置环境变量 ``PYARMOR_CC`` 来指定这个文件，例如::

      set PYARMOR_CC=C:\path\to\clang.exe

* 从 `LLVM 官网 <https://releases.llvm.org>`_ 下载并安装预编译版本
* 从 Pyarmor 官网下载 `clang-9.0.zip`__ ，压缩包大小约为 26M 左右，里面只有一个可执行文件，解压后存放在 :term:`根目录` 下面，默认是 ``%HOME%/.pyarmor``

__ https://pyarmor.dashingsoft.com/downloads/tools/clang-9.0.zip

配置好编译器之后，使用选项 :option:`--enable-bcc` 启用 BCC 模式 [#]_::

    $ pyarmor gen --enable-bcc foo.py

模块级别的代码不会转换转换成为 :term:`C` 的函数，模块的任何函数如果使用了不被支持的特性，也不会转换成为 :term:`C` 函数，这么没有使用 BCC 模式加密的函数会根据选项使用其他方式进行加密。

为了查看那些函数被转换成为 :term:`C` 函数，使用下面的方式启用跟踪模式::

    $ pyarmor cfg enable_trace=1
    $ pyarmor gen --enable-bcc foo.py

然后查看跟踪日志，日志中会显示那些转换的函数所在的脚本和行号::

    $ ls .pyarmor/pyarmor.trace.log
    $ grep trace.bcc .pyarmor/pyarmor.trace.log

    trace.bcc            foo:5:hello
    trace.bcc            foo:9:sum2
    trace.bcc            foo:12:main

运行加密后的脚本::

    $ python dist/foo.py

如果加密脚本出错，可以根据错误信息里面报告的函数名称，那么通过设置让 BCC 模式不要把转换该函数。例如，如果运行的时候错误和函数 ``sum2`` 有关，那么不转换这个函数::

    $ pyarmor cfg -p foo bcc:excludes "sum2"

选项 ``bcc:excludes`` 指定函数名称，选项 ``-p`` 指定模块名称。如果没有指定模块名称的话，其他所有模块中的同名函数也会被 BCC 模式忽略。增加更多的函数名称使用下面的命令格式::

    $ pyarmor cfg -p foo bcc:excludes + "hello"

如果这样加密脚本还是不能运行，请参考 :doc:`../topic/bccmode` 来解决问题。

.. [#] 这个功能仅在 :term:`Pyarmor 专家版` 中可用

定制错误处理方式
================

当运行密钥已经过期，或者和当前设备不匹配，以及加密脚本保护异常，默认的处理方式是 抛出运行错误的异常，并显示错误信息::

    $ pyarmor gen -e 2020-05-05 foo.py
    $ python dist/foo.py

    Traceback (most recent call last):
      File "dist/foo.py", line 2, in <module>
        from pyarmor_runtime_000000 import __pyarmor__
      File "dist/pyarmor_runtime_000000/__init__.py", line 2, in <module>
        from .pyarmor_runtime import __pyarmor__
    RuntimeError: this license key is expired (1:10937)

如果需要只显示错误信息，那么可以使用下面的方式进行配置::

    $ pyarmor cfg on_error=1

    $ pyarmor gen -e 2020-05-05 foo.py
    $ python dist/foo.py

    this license key is expired (1:10937)

如果需要直接退出，不显示任何信息，可以进行下面的配置::

    $ pyarmor cfg on_error=2

    $ pyarmor gen -e 2020-05-05 foo.py
    $ python dist/foo.py

    $

可以使用下面的任意一个命令恢复默认处理方式::

    $ pyarmor cfg on_error=0
    $ pyarmor cfg --reset on_error

这个设置仅仅影响的 Pyarmor 抛出的错误信息，不会影响脚本代码中本身抛出的异常

.. note::

   如果加密脚本不是被 Python 直接执行，这个选项可能无法不显示错误直接退出。

过滤加密字符串
==============

默认情况下选项 :option:`--mix-str` 会加密脚本中所有长度大于 8 的字符串。

但是这样有时候可能会影响性能，如果需要的话，可以通过定制，只加密指定的敏感字符串。

如果只需要加密长度大于 10 的字符串，可以使用下面的设置::

    $ pyarmor cfg mix.str:threshold 10

如果不需要加密两个字符串 ``__main__`` 和 ``xyz`` ，使用下面的命令进行过滤::

    $ pyarmor cfg mix.str:excludes ^ "__main__ xyz"

如果不需要加密长度超过 1000 的字符串，可以使用格式为 ``/pattern/`` 的正则表达式来实现。例如::

    $ pyarmor cfg mix.str:excludes ^ "/.{1000,}/"

重新恢复默认的过滤规则，使用下面的命令::

    $ pyarmor cfg mix.str:excludes = ""

如果只需要加密长度在 8 和 32 之间的字符串，可以使用正则表达式和下面的命令::

    $ pyarmor cfg mix.str:includes = "/.{8,32}/"

配置之后可以通过检查跟踪日志查看效果。

过滤需要保护的函数和模块
========================

选项 :option:`--assert-call` 和 :option:`--assert-import` 能够对函数和模块进行保护，确保它们是被加密的，从而避免一些对加密脚本注入式的攻击。

但是有时候可能会做一些错误的判断，去保护一些没有加密的函数或者模块，这时候就需要增加一个过滤规则把这些没有加密的函数和模块排除出去。

例如，告诉 :option:`--assert-import` 忽略模块 ``json`` 和 ``inspect`` 使用下面的规则::

    $ pyarmor cfg assert.import:excludes = "json inspect"

告诉 :option:`--assert-call` 忽略所有以 ``wintype_`` 开头的函数使用下面的规则::

    $ pyarmor cfg assert.call:excludes "/wintype_.*/"

除了保护了不该保护的函数之外，另外一种错误是有些加密函数却没有被自动识别出来并进行保护，这就需要使用下面的方法进行修正。

使用内联标识符修正加密脚本
==========================

在加密脚本之前，Pyarmor 扫描每一行，如果发现该行有内联标识符 ``# pyarmor:`` ， 那么会这一行进行额外处理：删除内联标识符和其后的一个空格，保留其后的所有内容。

例如，这个脚本经过处理之后

.. code-block:: python
    :emphasize-lines: 3,4

    print('start ...')

    # pyarmor: print('this script is obfuscated')
    # pyarmor: check_something()

就会变成下面的样子

.. code-block:: python
    :emphasize-lines: 3,4

    print('start ...')

    print('this script is obfuscated')
    check_something()

使用内联标识符修正加密脚本的一个实例就是使用 ``__assert_armored__`` 来保护额外的模块或者函数

默认情况下 :option:`--assert-import` 能够自动识别使用 ``import`` 语句导入的模块并进行保护，但是使用其他方法导入的模块，不会进行处理。例如，使用下面的方式导入的模块就不会进行处理

.. code-block:: python

    m = __import__('abc')

如果我们需要确保模块 ``abc`` 是加密的，没有被替换的，可以使用加密脚本特有的内置函数 :func:`__assert_armored__` 来检查 ``m``

.. code-block:: python

    m = __import__('abc')
    __assert_armored__(m)


但是这样修改后的脚本存在一个问题是没有加密的时候，这个脚本无法正常运行，很不方便调试。因为函数 :func:`__assert_armored__` 只有在加密脚本中才存在。

内联标识符很好的解决了这个问题，我们对上面的脚本进行修正如下就可以完美解决这个问题

.. code-block:: python
    :emphasize-lines: 2

    m = __import__('abc')
    # pyarmor: __assert_armored__(m)

同样，有时候 :option:`--assert-call` 也会遗漏一些需要保护的函数，这时候也可以使用内联标识符和 :func:`__assert_armored__` 进行人工保护。例如，下面的例子对函数 ``self.foo.meth`` 进行人工检查:

.. code-block:: python
    :emphasize-lines: 2

    # pyarmor: __assert_armored__(self.foo.meth)
    self.foo.meth(x, y, z)

错误信息支持多语言
==================

Pyarmor 提供错误信息的多语言功能，可以根据 :term:`客户设备` 的语言设置显示不同语言的错误信息。

为了支持多语言，首先创建 :file:`~/.pyarmor/messages.cfg`

    $ mkdir .pyarmor
    $ vi .pyarmor/message.cfg

这是一个 ``.ini`` 格式的文件，增加一个节 ``runtime.message`` 和选项 ``languages``:

.. code:: ini

  [runtime.message]

  languages = zh_CN zh_TW

  error_1 = invalid license
  error_2 = invalid license

这个例子中支持两种语言，其中语言代码和环境变量 :envvar:`LANG` 中的设置一样。错误信息我们只定制了前两个错误:

* error_1: 许可证已经过期
* error_2: 许可证不可用于当前设备

默认错误信息设置为 ``invalid license`` ，也就是说除了指定的语言之外，在其他语言环境都显示这个默认错误信息。

定制不同语言的错误消息分别使用两个节 ``runtime.message.zh_CN`` 和 ``runtime.message.zh_TW``

.. code:: ini

  [runtime.message]

  languages = zh_CN zh_TW

  error_1 = invalid license
  error_2 = invalid license

  [runtime.message.zh_CN]

  error_1 = 脚本超期
  error_2 = 未授权设备

  [runtime.message.zh_TW]

  error_1 = 腳本許可證已經過期
  error_2 = 腳本許可證不可用於當前設備

然后重新加密脚本::

    $ pyarmor gen foo.py

当加密脚本运行的时候，它会检查当前设备的 :envvar:`LANG` 来设置默认的语言。如果当前环境的语言设置不是 ``zh_CN`` 或者 ``zh_TW`` ，那么使用默认的错误信息。

环境变量 :envvar:`PYARMOR_LANG` 可以用来指定加密脚本的使用的错误信息语言。如果这个环境变量设置，会忽略系统环境变量 :envvar:`LANG` 。 例如，使用下面的方式运行加密脚本将总是显示繁体中文的错误信息::

    export PYARMOR_LANG=zh_TW
    python dist/foo.py

生成跨平台加密脚本
==================

.. versionadded:: 8.1

生成跨平台运行的加密脚本需要使用选项 :option:`--platform` 指定目标设备的平台名称，这里列出了所有支持的 :term:`运行平台` 名称。

例如，在一台 Darwin 开发设备上面生成可以运行在 Windows 下面的加密脚本，需要使用下面的命令::

    $ pyarmor gen --platform windows.x86_64 foo.py

Python 包 :mod:`pyarmor.cli.runtime` 提供了其他平台的预编译扩展模块，跨平台加密需要首先安装这个包，如果没有安装的话，会提示安装。

如果需要加密脚本可以运行在多个平台上，可以使用 :option:`--platform` 多次，指定每一个运行的平台。例如，下面的命令生成的加密脚本可以运行在多个 x86_64 的操作系统::

    $ pyarmor gen --platform windows.x86_64
                  --platform linux.x86_64 \
                  --platform darwin.x86_64 \
                  foo.py

支持多个 Python 版本的加密脚本
==============================

.. versionadded:: 8.3

下面是加密一个脚本 `foo.py` 同时支持 Python 3.8 和 3.9 的基本步骤

首先要为每一个 Python 版本安装 Pyarmor::

    $ python3.8 -m pip install pyarmor
    $ python3.9 -m pip install pyarmor

如果已经购买了 Pyarmor 的许可证，使用任意一个 Python 版本进行注册::

    $ python3.8 -m pyarmor.cli reg pyarmor-regfile-xxxx.zip

同时需要启用内置插件 ``MultiPythonPlugin``::

    $ python3.8 -m pyarmor.cli cfg plugins + "MultiPythonPlugin"

使用每一个 Python 版本分别加密脚本，保存在不同的目录::

    $ python3.8 -m pyarmor.cli gen -O dist1 foo.py
    $ python3.9 -m pyarmor.cli gen -O dist2 foo.py

最后使用辅助脚本 ``merge.py`` 来合并两个输出目录::

    $ python3.8 -m pyarmor.cli.merge -O dist dist1 dist2

最终输出的脚本在 ``dist``::

    $ python3.8 dist/foo.py
    $ python3.9 dist/foo.py

使用公共的运行辅助包
====================

:term:`运行辅助包` 可以单独生成，并且在加密脚本的时候直接引用，这样就不需要在每次加密脚本的时候都生成 :term:`运行辅助包` 。

首先是使用命令 :ref:`pyarmor gen runtime` 生成共用的运行辅助包::

    $ pyarmor gen runtime -O build/my_runtime1
    $ ls build/my_runtime1

然后在加密脚本的时候使用 :option:`--use-runtime` 引用这个运行辅助包::

    $ pyarmor gen --use-runtime build/my_runtime1 foo.py

发布加密脚本的时候需要把 :term:`运行辅助包` 拷贝到加密脚本的输出路径 `dist`::

    # 请把 pyarmor_runtime_000000 替换成为实际的名称
    $ ls build/my_runtime1/
    $ cp -a build/my_runime1/pyarmor_runtime_000000 dist/

如果需要生成支持多个平台的运行辅助包，使用下面的选项::

    $ pyarmor gen --platform windows.x86_64,linux.x86_64 build/my_runtime3

如果使用 :term:`外部密钥` ，那么生成加密脚本和运行辅助包的时候都需要使用选项 :option:`--outer` 。例如::

    $ pyarmor gen runtime --outer -O build/my_outer_runtime
    $ pyarmor gen --outer --use-runtime build/my_outer_runtime foo.py

    # 拷贝运行辅助包到加密脚本输出目录
    $ cp -a build/my_outer_runtime/pyarmor_runtime_000000 dist/

    # 生成外部密钥
    $ pyarmor gen key -e .10
    $ mv dist/pyarmor.rkey dist/pyarmor_runtime_000000

.. include:: ../_common_definitions.txt
