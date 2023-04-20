=================
深入了解 RFT 模式
=================

Pyarmor 的 RFT 模式能够对模块中的函数，类，方法，属性，变量等进行重命名，相当于把源代码重新写了一下，所以这种加密方式不可逆的。

启用 RFT 模式之后，模块中的所有定义的名称都会发生变化，外部脚本将无法使用原来的名称进行导入。如果需要输出某些名称被外部脚本使用，需要在模块中定义 ``__all__`` ，这里面列出的名称会被保留。

使用 RFT 模式的时候，必须把所有使用的脚本和包在同一条命令进行加密。Pyarmor 会分析脚本之间的调用关系，对导入的名称也进行正确的重命名。

Pyarmor 使用内置的自动规则和人工配置的规则来分析脚本，并进行重命名。虽然 RFT 模式的自动规则可以处理大多数的情况，但是对于复杂的脚本和包，肯定会存在一些名称没有被正确处理。因为 Python 语言自身的特点，Pyarmor 并不认为可以找到算法自动处理所有的情况，所以对特殊情况，必须要进行人工配置规则。RFT 模式的目标是不断更新和完善自动规则，对于不能自动处理的部分，能够自动生成相关的参考配置，用户通过这些参考配置生成人工规则。

对于复杂的脚本，不要期望 Pyarmor 会自动进行处理所有情况。

我们以下面的例子来说明 RFT 模式的各个功能特点::

    src/
        foo.py
        joker/
            __init__.py
            card.py

使用 RFT 模式加密脚本 foo.py 以及包 joker 的命令如下::

    pyarmor gen --enable-rft foo.py src/joker/


使用 ``__all__`` 输出名称
=========================

.. code-block:: python

    __all__ = ['foo']

    def foo(msg):
        print(msg)

    def _private_foo(msg):
        print(msg)

对于属性名称，同样存在无法确定是否需要进行重命名的情况。并且，有些变量的类型可能在运行的时候才能确定，所以根本无法在编译的时候决定是否进行重命名

.. code-block:: python

    obj = call_foo()
    obf.name = 'joker'


模块，类在类型字典中名称，存放在 ``module_types``::
  mod
  pkg
  pkg.mod
  pkg.mod.Cls
  pkg.mod[.Cls|.func].Cls

函数，变量和参数在变量字典中的名称，存放在 ``variable_types``::

  pkg.mod[.Cls|.func]*.[arg|var]

特殊的，Lambda 和 Comprehension 不会出现在 ``.func`` 中

``__init__.py`` 中的 ``__init__`` 在类型表和变量表中会被忽略

类型名称 Field.cls 的可选值::

  int/float/str/Exception 等内置类型
  import    外部导入的名称
  module    模块类型
  class     类
  function  函数
  method    ？方法，may be seem as function
  pkg.mod.Cls 自定义类型
  ?         不确定类型，这种类型的属性会加入到自动排除列表
  None      类型不存在，等价于不确定类型

  类型为空表示需要重新确定的类型，只存在预处理的时候，扫描结束必须确定为以上列出的类型之一

如果需要输出名称，必须在模块中定义 ``__all__`` ，这里面列出的名称会被保留不变。需要注意的是这里列出的名称也可能是从别的模块中导入的。

两种不同的算法处理不可识别的变量类型：

1. 排除法
2. 包含法

第一种方法是所有不可识别类型的变量使用的属性，以及下一级的属性，全部设置为自动排除的名称，不进行重命名。

第二种方法是首先重命名每一个模块的类/函数和属性名称，如果不可识别的类型使用到已经重命名的字符串，那么，这些变量会被写入到配置文件，用户可以修改这些配置文件，然后重新运行重命名

第一步是分析所有模块的类型，生成 module_types 表，以及参数/变量类型表 variable_types

每一个模块和类都应该存在 module_types
每一个变量和参数都应该存在 variable_types
没有任何一个类型为空，不确定的类型设置成为 '?'

第二步是重构每一个脚本


相关选项和配置
==============

配置 ``rft_excludes`` 排除名称，列出的名称会被保留: 类，方法，函数，属性
配置 ``rft_rulers`` 人工规则，定义如何重命名属性链: x.y.z:?.%.!

名称引用
--------

使用模块名称和包名称来引用相关的脚本::

  foo -> foo.py

  joker -> joker/__init__.py
  joker.card -> joker/card.py

属性链，用来匹配下列语句::

  self.a.b = 1               ==> self.a.b
  self.a[2].b                ==> self.a.b
  foo().a[2].b.c             ==> foo.a.b.c
  (a+b).tostr()              ==> ?.tostr
  (a+b).tostr().geta()       ==> ?.tostr.geta
  'a'.tolower().replace()    ==> ?.tolower.replace
  b.a[3].k = 5               ==> b.a.k

使用 trace log::

  pyarmor cfg enable_trace=1 trace_rft=1

查看 ``.pyarmor/pyarmor.trace.log``::

  trace.rft            t1090:17 (exclude attrs "wintypes.DWORD")
  trace.rft            t1090:32 (! self.dwFlags)
  trace.rft            t1090:37 (wintypes.DWORD->pyarmor__27.DWORD)

人工规则
--------

定义全局规则::

  pyarmor cfg rft_ruler="a.b.c *.%.*"

定义模块私有规则::

  pyarmor cfg -p joker.card rft_ruler="a.b.c *.%.*"

规则格式::

  属性链模式:对应属性重命名方式

属性链模式是以 ``.`` 分开的支持 fnmatch 的模式，然后使用此模式和属性链进行匹配

满足匹配的属性链按照后面的重命名方式进行处理

``%``: 总是进行重命名
其他值: 保留原来的名称

人工规则只用来修改属性名称，而忽略第一个名称，例如

.. include:: ../_common_definitions.txt
