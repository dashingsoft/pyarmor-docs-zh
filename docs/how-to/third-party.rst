========================================
 如何解决第三方库调用加密脚本存在的问题
========================================

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 提供了丰富的选项就是为了解决不同情况下遇到的问题，当使用一个复杂的第三方包调用加密脚本出现问题的时候，请首先花费一点时间看看 :doc:`../reference/man` ，去了解所有的选项和作用，有一些选项就是专门针对不同的应用情况设计的。 **Pyarmor 开发组不会去告诉应该使用那个选项就可以解决你遇到的问题** ，而是你需要去学习和了解 Pyarmor，并找到适合自己需要的选项。

Python 在各个领域得到广泛的应用，有很多包我甚至从来都没有用过。对于我来说，不会去学习每一个包，然后确保其能够和 Pyarmor 兼容。通常的处理方式是用户报告相关的异常，根据异常的行号给出附近的源代码，我可以帮助分析哪些地方可能和 Pyarmor 发生冲突并给出相应的解决方案。

对于属于 Pyarmor 的问题，Pyarmor 开发组会尽快解决，但是有一些问题是 Pyarmor 自身无法解决的。

通常情况下，使用第三方包主要导致的问题来源有:

* 使用 ``sys._getframe`` 去访问函数的局部变量，或者其他运行框架的信息，但是加密脚本的运行框架和普通脚本是不一样的
* 使用 :mod:`inspect` 或者其他方式直接去访问函数的源代码（Byte code）等相关属性，它们所访问的属性正好就是 Pyarmor 所保护的
* 使用 :mod:`pickle` 或者类似功能序列号加密函数，然后把加密函数传递给其他进程或者线程执行，但是加密函数是无法使用普通方法序列化的

这些问题都来源于第三方包使用到了加密脚本修改过的底层对象，更多的不同之处请参考 :ref:`the differences of obfuscated scripts` 。如果使用被修改的特性，就会出现不兼容的问题。特别是使用 :term:`BCC 模式` 进行加密的脚本，改变的内容更多。

对于这些问题，有一些常用的解决方案，可以首先尝试下面的方式

- 使用 :term:`RFT 模式` 和选项 :option:`--obf-code` ``0``

  :term:`RFT 模式` 几乎不改变原来脚本的内部结构，所以一般不会导致不兼容性。使用 :option:`--obf-code` 禁用代码对象的加密也是为了保持内部对象的结构不改变。这是一种推荐的解决方案::

    $ pyarmor gen --enable-rft --obf-code 0 /path/to/myapp

  首先使用最少的选项确保其能工作，然后在尝试更多的选项去增加安全性。例如::

    $ pyarmor gen --enable-rft --obf-code 0 --mix-str /path/to/myapp
    $ pyarmor gen --enable-rft --obf-code 0 --mix-str --assert-call /path/to/myapp

- 忽略有问题的模块

  在一个复杂的应用中，如果只是个别模块存在问题，可以不加密这些模块。例如，如果只有模块 ``config.py`` 不能正常工作，使用模块私有配置的方式把这个模块忽略掉::

    $ pyarmor cfg -p myapp.config obf_code=0
    $ pyarmor gen [other options] /path/to/myapp

  也可以按照原来的方法加密，只是加密之后使用原来的脚本直接把加密脚本替换::

    $ pyarmor gen [other options] /path/to/myapp
    $ cp /path/to/myapp/config.py dist/myapp/config.py

- 修改第三方库

  这是一个实例，如果使用别名 "myapi" 去调用，抛出 404 错误，但是不使用别名工作正常。

  .. code-block:: python

      @cherrypy.expose(alias='myapi')
         @cherrypy.tools.json_out()
         # pylint: disable=no-member
         @cherrypy.tools.authenticate()
         @cherrypy.tools.validateOptOut()
         @cherrypy.tools.validateHttpVerbs(allowedVerbs=['POST'])
         # pylint: enable=no-member
         def abc_xyz(self, arg1, arg2):
             """
             This is the doc string
             """

  导致问题出现的原因在于 ``cherrypy.expose`` 使用下面的语句

  .. code-block:: python

      parents = sys._getframe(1).f_locals

  因为 ``sys._getframe(1)`` 在加密脚本返回的不是期望的执行框架，把它修改成为下面的语句就可以在加密脚本正常使用:

  .. code-block:: python

      parents = sys._getframe(2).f_locals

  .. note::

      如果第三方包 :mod:`cheerypy` 也被其他程序使用，请为加密脚本创建一个私有的包

常用的第三方库
==============

这里列出了一些常用到的第三方库以及可能的解决方案，欢迎大家分享自己的使用经验，提交 Pull request 增加新的第三方库。

.. list-table:: 表-1. 常用第三方库列表
   :header-rows: 1

   * - 库
     - 状态
     - 备注
   * - cherrypy
     - 打补丁 [#patch]_
     - 问题原因是使用了 sys._getframe
   * - `pandas`_
     - 打补丁 [#patch]_
     - 问题原因是使用了 sys._getframe
   * - playwright
     - 打补丁应该可以工作 [#RFT]_
     - 尚未验证
   * - `nuitka`_
     - 使用 restrict_module = 0 之后应该可以工作
     - 尚未验证

.. rubric:: 说明

.. [#patch] 通过打补丁的方式可以使用加密脚本
.. [#RFT] 可以使用 :term:`RFT 模式` 加密的脚本
.. [#obfcode0] 只有使用 ``--obf-code 0`` 加密的脚本可以工作
.. [#not] 任何方式都无法使用加密脚本

pandas
------

另外一个实例是 :mod:`pandas`

.. code-block:: python

    import pandas as pd

    class Sample:
        def __init__(self):
            self.df = pd.DataFrame(
                data={'name': ['Alice', 'Bob', 'Dave'],
                'age': [11, 15, 8],
                'point': [0.9, 0.1, 0.4]}
            )

        def func(self, val: float = 0.5) -> None:
            print(self.df.query('point > @val'))

    sampler = Sample()
    sampler.func(0.3)

加密之后运行报错::

    pandas.core.computation.ops.UndefinedVariableError: local variable 'val' is not defined

同样需要对其打补丁，把 pandas ``scope.py`` 里面的 ``sys._getframe(self.level)`` 修改成为 ``sys._getframe(self.level+1)`` ， ``sys._getframe(self.level+2)`` 或者 ``sys._getframe(self.level+3)``

nuitka
------

把加密脚本作为普通脚本一样使用 Nuitka 进行处理，但是需要禁用约束模式::

    $ pyarmor cfg restrict_module=0

如果不禁用约束模式，会导致校验错误 ``RuntimeError: unauthorized use of script``

使用默认选项进行加密::

    $ pyarmor gen foo.py

然后在尝试使用更多选项，但是约束相关的这些选项，例如 :option:`--private` ， :option:`--restrict` ， :option:`--assert-call` ， :option:`--assert-import` 可能无法使用。

.. note::

  在 v9.1.0 之前的版本中，扩展模块 `pyarmor_runtime.so` 必须在包目录下面，例如::

      $ ls dist/pyarmor_runtime_000000
      ...    __init__.py
      ...    pyarmor_runtime.so

  如果 Nuitka 把 `__init__.py` 转换成为 `pyarmor_runtime_000000_init_.py` ，并且拷贝 `pyarmor_runtime.so` 到相同目录下面，运行时候通用会报错 ``RuntimeError: unauthorized use of script``

streamlit
---------

需要禁用下列选项，然后在进行加密::

    $ pyarmor cfg restrict_module=0
    $ pyarmor cfg clear_module_co=0

    $ pyarmor gen foo.py

不禁用第一项可能会报错 `RuntimeError: unauthorized use of script (1:1102)`

不禁用第二项可能会报错 `RuntimeError: the format of obfuscated script is incorrect (1:1082)`

**不过 Streamlit 依旧可能无法直接使用加密脚本，因为它是直接访问甚至修改 code object**

.. include:: ../_common_definitions.txt
