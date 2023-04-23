=================
深入了解 BCC 模式
=================

.. highlight:: console

.. program:: pyarmor gen

BCC 模式会把部分函数直接转换成为二进制代码，在根本上避免被还原成为 Python 函数。

BCC 模式需要配置 :term:`C` 编译器，对于 `Linux` 和 `Darwin` 来说，一般不需要进行配置，只要默认的 ``gcc`` 和 ``clang`` 能工作就可以。在 `Windows` 环境下面，可以使用下面任意一种方式配置 ``clang.exe`` ，目前其它编译器还不支持:

* 如果已经有 ``clang.exe`` ，只要在其它路径直接运行 ``clang.exe`` 不出错就可以。如果文件存在，但是无法在任意路径直接运行，可以配置环境变量 ``PYARMOR_CC`` 来指定这个文件，例如::

      set PYARMOR_CC=C:\path\to\clang.exe

* 从 `LLVM 官网 <https://releases.llvm.org>`_ 下载并安装预编译版本
* 从 Pyarmor 官网下载 `clang-9.0.zip`__ ，压缩包大小约为 26M 左右，里面只有一个可执行文件，解压后存放在 :term:`根目录` 下面，默认是 ``%HOME%/.pyarmor``

__ https://pyarmor.dashingsoft.com/downloads/tools/clang-9.0.zip

启用 BCC 模式
=============

配置好编译器之后，使用选项 :option:`--enable-bcc` 启用 BCC 模式::

    $ pyarmor gen --enable-bcc foo.py

模块级别的代码不会转换转换成为 :term:`C` 的函数，模块的任何函数如果使用了不被支持的特性，也不会转换成为 :term:`C` 函数，这么没有使用 BCC 模式加密的函数会根据选项使用其他方式进行加密。

查看被 BCC 模式加密的函数
=========================

启用跟踪模式可以在跟踪日志文件 ``.pyarmor/pyarmor.trace.log`` 中记录那些函数被转换成为了 :term:`C` 函数。例如::

    $ pyarmor cfg enable_trace=1
    $ pyarmor gen --enable-bcc foo.py

查看跟踪日志中使用 ``trace.bcc`` 记录的内容::

    $ ls .pyarmor/pyarmor.trace.log
    $ grep trace.bcc .pyarmor/pyarmor.trace.log

    trace.bcc            foo:5:hello
    trace.bcc            foo:9:sum2
    trace.bcc            foo:12:main

第一条日志记录的是 ``foo.py`` 第5行的函数 ``hello`` 被转换成为 C 函数
第二条日志记录的是 ``foo.py`` 第9行的函数 ``sum2`` 被转换成为 C 函数

不转换特定的模块和函数
======================

使用 BCC 模式加密脚本并不是完全和原来的脚本兼容，如果运行 BCC 模式加密脚本出现了兼容性问题，那么解决方案就是不要转换特定的模块，或者模块中特定的函数。

为了避免影响其他脚本，这里使用 :term:`模块私有配置` 来修改单独修改模块的设置。

让 BCC 模式忽略一个模块 ``pkgname.modname`` 使用下面的命令::

    $ pyarmor cfg -p pkgname.modname bcc:disabled=1

忽略模块中一个函数使用下面的命令::

    $ pyarmor cfg -p pkgname.modname bcc:excludes + "function name"

忽略更多的函数::

    $ pyarmor cfg -p foo bcc:excludes + "hello foo2"

我们可以启用跟踪日志，查看这些语句的效果，看看这些模块和函数是否没有被 BCC 模式处理。例如::

    $ pyarmor cfg enable_trace 1
    $ pyarmor gen --enable-bcc foo.py
    $ grep trace.bcc .pyarmor/pyarmor.trace.log

另外一个例子，忽略 ``joker/card.py`` 但是使用 BCC 模式加密包 ``joker`` 的其他模块::

    $ pyarmor cfg -p joker.card bcc:disabled=1
    $ pyarmor gen --enable-bcc /path/to/pkg/joker

使用两个配置项 ``bcc:excludes`` 和 ``bcc:disabled`` 可以最小限度的排除不被 BCC 模式支持的代码，从而确保其他脚本能正常使用 BCC 模式加密运行。

改变的脚本特性
==============

使用 BCC 模式加密后脚本和原来的脚本存在一些额外的不同

* 部分异常的提示信息和原来不一样

* 如果不是在异常处理的过程中，直接调用没有参数的 `raise` 抛出的异常类型不同

  没有加密前

  .. code-block:: python

      >>> raise
      RuntimeError: No active exception to reraise

  在 BCC 模式加密的脚本中

  .. code-block:: python

      >>> raise
      UnboundlocalError: local variable referenced before assignment

* 函数对象的属性，尤其是以双下划线开始的属性，例如 ``__qualname__`` 等等，在转换成为 C 函数之后都不存在，使用这些属性的函数加密后无法正常工作

不支持的特性
============

使用了下列特性的函数无法转换成为 :term:`C` 函数

.. code-block:: python

    unsupport_nodes = (
        ast.ExtSlice,

        ast.AsyncFunctionDef, ast.AsyncFor, ast.AsyncWith,
        ast.Await, ast.Yield, ast.YieldFrom, ast.GeneratorExp,

        ast.NamedExpr,

        ast.MatchValue, ast.MatchSingleton, ast.MatchSequence,
        ast.MatchMapping, ast.MatchClass, ast.MatchStar,
        ast.MatchAs, ast.MatchOr
    )

如果调用了下列任意一个内置函数，那么该函数也无法转换成为 :term:`C` 函数:

* exec,
* eval
* super
* locals
* sys._getframe
* sys.exc_info

例如，下面这些函数都不会使用终极模式加密，因为它们或者使用了不支持的特性，或者调
用了不支持的函数:

.. code-block:: python

   async def nested():
       return 42

   def foo1():
       for n range(10):
           yield n

   def foo2():
      frame = sys._getframe(2)
      print('parent frame is', frame)

.. include:: ../_common_definitions.txt
