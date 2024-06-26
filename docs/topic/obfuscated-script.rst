================
深入了解加密脚本
================

.. highlight:: console

.. program:: pyarmor gen

阅读本文档需要一定的 Python 基础，了解 Shell 脚本，环境变量等相关知识。

加密脚本就是普通 Python 文件
============================

Pyarmor 加密后的脚本输出的是同名的 ``.py`` 文件和一个 :term:`运行辅助包` 。它们和普通 Python 模块一样，可以被 Python 解释器调用执行，这也是 Pyarmor 的一个加密特点，可以使用加密后的脚本无缝替换原来的脚本。

使用加密脚本完全和使用普通的 Python 脚本一样，例如，使用解释器直接运行::

    python dist/foo.py

使用文本编辑器打开这个脚本，它的内容一般如下::

    from pyarmor_runtime import __pyarmor__
    __pyarmor__(__name__, __file__, b'\x28\x83....')

可以看到第一条语句就是导入扩展模块 ``pyarmor_runtime`` ，它就是和加密脚本在相同目录下的文件 ``pyarmor_runtime.so`` 。扩展模块 ``pyarmor_runtime`` 不是必须和加密脚本放在一起，只要它存在于 Python 搜索模块的任何路径，能被 Python 导入进来，加密脚本就可以正常使用。

如果运行加密脚本的时候提示模块 ``pyarmor_runtime`` 无法找到，首先要能找到扩展模块文件，然后在扩展模块文件所在的目录直接使用 Python 解释器导入这个模块::

    cd dist/
    python
    >>> import pyarmor_runtime
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    RuntimeError: the format of obfuscated script is incorrect

如果不是抛出这个异常，说明这个扩展模块文件不适用于当前平台和 Python 版本。

扩展模块是二进制的动态库，有些没有加密的时候，脚本可以正常执行，加密之后无法运行。就是因为运行环境无法装载扩展模块而造成的。这种情况只要了解扩展模块的相关知识，设置运行环境允许加载扩展模块，都可以正常运行加密脚本。判断运行环境是否允许加载扩展模块的方法是把任何一个系统扩展模块拷贝到当前目录，看看能否导入。

在导入扩展模块 ``pyarmor_runtime`` 之前，所有的事情都是 Python 的自身功能，和 Pyarmor 和脚本是否加密都没有关系。解决这里出现的问题需要的就是学习 Python 相关的知识，特别是 Python 是如何根据模块名称去搜索和装载模块和扩展模块的。

扩展模块 ``pyarmor_runtime`` 第一次被导入的时候，会进行一些初始化工作，包括检查 :term:`运行密钥` 等，如果初始化失败，那么抛出异常退出。

**装载加密模块**

如果初始化正常完成，那么执行加密脚本的第二行语句，调用从扩展模块中导入的函数 ``__pyarmor__`` 来完成对加密模块的装载工作。

把模块加载完成之后，就又把控制器交给 Python 解释器，执行解密后的模块。

**装载加密函数**

当 Python 解释器调用加密函数的时候，控制权交给扩展模块 ``pyarmor_runtime`` 进行解密，解密完成之后返回 Python 解释器继续执行。

函数调用返回之前，控制权重新交给扩展模块 ``pyarmor_runtime`` 进行加密和做一些保护清理工作，最后在返回给 Python 解释器继续执行。

运行辅助包
==========

上面对加密脚本的说明中进行了简化，实际加密之后扩展模块 ``pyarmor_runtime`` 是在一个 :term:`运行辅助包` 里面，让我们查看一下加密后的目录就一目了然::

    $ pyarmor gen foo.py

    $ ls dist/

    foo.py    pyarmor_runtime_000000

    $ ls dist/pyarmor_runtime_000000
    ...    __init__.py
    ...    pyarmor_runtime.so

这里 ``dist/pyarmor_runtime_000000`` 就是运行辅助包，使用运行辅助包主要是为了能够支持在多平台运行的加密脚本，我们可以把预编译的不同平台扩展模块 ``pyarmor_runtime`` 都存放到包目录下面，然后在 ``__init__.py`` 里面根据不同的平台，导入相应平台的扩展模块，这样就可以让加密脚本运行在多平台下面。

运行辅助包也是一个正常的 :term:`Python 包` ，它不是必须和加密脚本在一起，只要满足 Python 模块导入机制的要求，它可以存放在任何地方。请查看选项 :option:`--use-runtime` 和命令 :ref:`pyarmor gen runtime` 了解更多使用方法。

运行辅助包可以使用相对导入的方式，也可以使用绝对导入的方式。Pyarmor 提供了相关选项 :option:`-i` 和 :option:`--prefix` 来帮助加密脚本生成正确的导入语句。如果还不能满足需求，可以自己编写 :term:`加密插件` 来修改加密脚本的导入语句，从而确保能导入运行辅助包。

运行辅助包的名称可以被配置成为其他名称，请参考 :doc:`../tutorial/customization` 中的 `设置运行辅助包的名称`

.. seealso:: :doc:`../tutorial/advanced` 中的 `使用公共的运行辅助包`

运行密钥
========

运行密钥保存对加密脚本的约束信息以及相关的一些运行设置。

运行密钥一般嵌入到扩展模块中，但是也可以使用外部密钥文件。

扩展模块在初始化的时候要验证运行密钥，验证失败就直接报错退出，只要验证成功之后才会继续执行。

运行密钥的验证模式只是在扩展模块初始化的时候进行验证，但是可以配置成为每一次导入加密模块都进行验证，也可以配置成为定时进行验证。

如果使用外部密钥文件，可以在其头部插入任何可读文本作为注释，这样通过这些注释，不需要在加密脚本内部就可以读取运行密钥的相关信息，从而做一些额外的处理。

用户可以在运行密钥中绑定任何私有数据（但是长度有一定限制，不能超过 4K)，然后使用 :term:`脚本补丁` 在加密脚本自己去验证这些数据，从而实现对加密脚本的限制和约束。这里所有的数据格式和业务逻辑全部由用户自己控制，Pyarmor 只是提供了这种扩展机制。

.. seealso:: :ref:`pyarmor gen key`

.. _restrict modes:

约束模式
========

约束模式用来对加密脚本进行一定的约束和限制。

默认约束模式是不允许对加密后的脚本进行修改。

使用 :option:`--private` 之后不允许外部脚本访问加密脚本的属性和方法

使用 :option:`--restrict` 之后不允许外部脚本导入加密脚本，也不允许外部脚本访问加密脚本的属性和方法

如果需要禁用全部的约束，使用下面的命令::

    $ pyarmor cfg restrict_module 0

一般情况下，是对某一个特殊脚本禁用约束，而其他脚本的约束不变，这样在配置的时候需要指定相应的模块::

    $ pyarmor cfg -p NAME restrict_module 0

.. _the differences of obfuscated scripts:

加密脚本和原来脚本的区别
========================

加密脚本和原来的脚本相比，存在下列一些的不同:

* 加密脚本是和 Python 版本绑定的，例如使用 Python 3.5 加密的脚本，只能使用 Python 3.5 去运行，而无法使用 Python 3.6 去运行，但是可以使用不同补丁的 3.5 版本。

* 加密脚本是平台相关，因为使用到了动态库，不同的平台需要相应平台的动态库。平台根据操作系统，CPU 架构来进行区别，例如 32 位 X86 Windows，Linux Aarch64。

* 执行加密角本的 Python 不能是调试版，准确的说，不能是设置了 Py_TRACE_REFS 或者 Py_DEBUG 生成的 Python

* 使用 ``sys.settrace``, ``sys.setprofile``, ``threading.settrace`` 和 ``threading.setprofile`` 设置的回调函数在加密脚本中将被忽略，所以任何使用这些函数的工具无法正常工作。

* 模块 ``inspect`` 和其他任何第三方包如果试图访问加密脚本的 Byte Code 或者直接访问代码对象的某些属性，也会崩溃，失败或者得到错误的数据

* 使用 ``cPickle`` 或者其他序列化工具传递加密代码对象，传递之后的代码对象可能无法正常运行。

* ``sys._getframe([n])`` 可能得到的不是期望的运行框架，因为加密脚本可能增加了额外的运行框架。

* 加密脚本抛出异常中的行号和原来的脚本在个别情况下会不一样

* 代码块的属性 ``__file__`` 在加密脚本是 ``<frozen name>`` ，而不是文件名称，在异常信息中会看到文件名的显示是 ``<frozen name>``

  需要注意的是模块的属性 ``__file__`` 还和原来的一样，还是文件名称。加密下面的脚本并运行，就可以看到输出结果的不同:

  .. code-block:: python

      def hello(msg):
          print(msg)

      # The output will be 'foo.py'
      print(__file__)

      # The output will be '<frozen foo>'
      print(hello.__file__)

有些选项也会影响到脚本的内部结构:

* ``pyarmor cfg mix_argname=1`` 会导致 annotations 无法使用

* 直接使用 `importlib.util.spec_from_file_location` 导入加密模块需要额外处理，参考 `第846个问题报告`__

.. seealso::

   :doc:`../how-to/third-party`

   :doc:`../tutorial/advanced` 中的 `生成跨平台加密脚本` 以及 `支持多个 Python 版本的加密脚本`

__ https://github.com/dashingsoft/pyarmor/issues/846

第三方解释器的支持
------------------

对于第三方的解释器（例如 Jython 等）以及通过嵌入 Python C/C++ 代码调用加密脚本，只要第三方解释器能够和 CPython :term:`扩展模块` 兼容，就可以使用加密脚本。请自行查看第三方解释器的文档，确认它是否支持 CPython 的扩展模块。

已知的一些问题

* `PyPy` 无法运行加密脚本，因为它完全不同于 `CPython` 。

* 在 Linux 下面 装载 Python 动态库 `libpythonXY.so` 的时候 `dlopen` 必须设置 `RTLD_GLOBAL` ，否则加密脚本无法运行。

* Boost::python，默认装载 Python 动态库是没有设置 `RTLD_GLOAL` 的，运行加密脚本的时候会报错 "No PyCode_Type found" 。解决方法就是在初始化的调用方法 `sys.setdlopenflags(os.RTLD_GLOBAL)` ，这样就可以共享动态库输出的函数和变量。

* 模块 `ctypes` 必须存在并且 `ctypes.pythonapi._handle` 必须被设置为 Python 动态库的句柄，PyArmor 会通过该句柄获取 Python C API 的地址。

* WASM 目前不支持，因为这需要把运行库的代码也编译成为 WASM，但是 WASM 是很容易就被反编译成为原来的 C 代码，为了安全性，所以目前没有支持 WASM 的计划。如果有更多的用户提出这个需求，会考虑实现一个轻量级的运行库，只支持能够运行 RFT 模式的加密脚本，但是目前还没有开发计划。

.. include:: ../_common_definitions.txt
