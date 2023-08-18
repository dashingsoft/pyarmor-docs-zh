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

如果在 ``trace.bcc`` 之后的字符是 ``!`` ，那么意味着这个函数被 BCC 模式忽略。例如::

    trace.bcc ! foo:29:Test.new (unsupported function "super")

不转换特定的模块和函数
======================

使用 BCC 模式加密脚本并不是完全和原来的脚本兼容，如果运行 BCC 模式加密脚本出现了兼容性问题，那么解决方案就是不要转换特定的模块，或者模块中特定的函数。

为了避免影响其他脚本，这里使用 :term:`模块私有配置` 来修改单独修改模块的设置。

让 BCC 模式忽略一个模块 ``pkgname.modname`` 使用下面的命令::

    $ pyarmor cfg -p pkgname.modname bcc:disabled=1

忽略模块中的函数或者方法使用下面的命令::

    $ pyarmor cfg -p pkgname.modname bcc:excludes="name"
    $ pyarmor cfg -p pkgname.modname bcc:excludes="name1 name2 name3"

    $ pyarmor cfg -p pkgname.modname bcc:excludes="Class.method_1"
    $ pyarmor cfg -p pkgname.modname bcc:excludes="Class.*"

如果没有选项 ``-p`` 的话，其他模块中同名函数和方法也会被忽略。

下面是一个示例脚本 :file:`foo.py`

.. code-block:: python

    def hello_a():
        pass

    def hello_b():
        pass

    class Test(object):

        def __init__(self):
            pass

        def hello_a():
            pass

使用下面的任意一种方式来忽略其中的函数和方法::

    $ pyarmor cfg -p foo bcc:excludes = "hello_a"
    $ pyarmor cfg -p foo bcc:excludes = "hello_a hello_b"
    $ pyarmor cfg -p foo bcc:excludes = "hello_*"

    $ pyarmor cfg -p foo bcc:excludes = "Test.hello_a"
    $ pyarmor cfg -p foo bcc:excludes = "Test.*"
    $ pyarmor cfg -p foo bcc:excludes = "Test.__*__"

    $ pyarmor cfg -p foo bcc:excludes = "hello_a Test.hello_a"

如果只需要 BCC 模式处理特定的函数，那么使用选项 `bcc:includes`::

    # 恢复默认选项
    $ pyarmor cfg bcc:excludes = ""

    # BCC 模式只处理模块函数 "hello_a"
    $ pyarmor cfg -p foo bcc:includes = "hello_a"

    # BCC 模式还需要处理类方法 "Test.hello_a"
    $ pyarmor cfg -p foo bcc:includes + "Test.hello_a"

    # BCC 模式处理类 "Test" 的所有方法，除了 "__init__"
    $ pyarmor cfg -p foo bcc:includes="Test.*" bcc:excludes="Test.__init__"

我们可以启用跟踪日志，查看这些语句的效果，看看这些模块和函数是否没有被 BCC 模式处理。例如::

    $ pyarmor cfg enable_trace 1
    $ pyarmor gen --enable-bcc foo.py
    $ grep trace.bcc .pyarmor/pyarmor.trace.log

另外一个例子，忽略 ``joker/card.py`` 但是使用 BCC 模式加密包 ``joker`` 的其他模块::

    $ pyarmor cfg -p joker.card bcc:disabled=1
    $ pyarmor gen --enable-bcc /path/to/pkg/joker

选项 `bcc:includes` 和 `bcc:excludes` 只对模块基本的函数和类有效，它们不能处理子函数和子类。例如

.. code-block:: python

    def hello():

        def wrap():
            pass

        class Test:

            def __init__(self):
                pass

下面的命令无法忽略子函数 ``wrap`` 和子类 ``Test``::

    pyarmor cfg bcc:excludes = "wrap hello.wrap Test.__init__ hello.Test.__init__"

忽略子函数和子类的唯一方法是直接忽略其所在的顶层函数 ``hello``::

    pyarmor cfg bcc:excludes = hello

.. versionadded:: 8.3.4

   新增选项 `bcc:includes`.

.. versionchanged:: 8.3.4

   选项 `bcc:excludes` 的使用方法发生了变化。在之前的版本::

       # 忽略模块函数和所有的类方法 "hello_a"
       pyarmor cfg bcc:excludes="hello_a"

       # 在方法前指定类名称的话，无法忽略任何方法
       pyarmor cfg bcc:excludes="Myclass.hello_a"

   在现在的版本中::

       # 需要使用下面的形式来忽略模块函数和所有的类方法 "hello_a"
       pyarmor cfg bcc:excludes="hello_a *.hello_a"

       # 可以指定类名称来忽略单独的类方法
       pyarmor cfg bcc:excludes="Myclass.hello_a"

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
