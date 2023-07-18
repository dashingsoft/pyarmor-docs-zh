=================
深入了解 RFT 模式
=================

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 的 RFT 模式能够对模块中的函数，类，方法，属性，变量等进行重命名，相当于把源代码重新写了一下，所以这种加密方式不可逆的。

启用 RFT 模式之后，模块中的所有定义的名称都会发生变化，外部脚本将无法使用原来的名称进行导入。如果需要输出某些名称被外部脚本使用，需要在模块中定义 ``__all__`` ，这里面列出的名称会被保留。

RFT 模式会修改下列名称

* 函数
* 类
* 方法
* 全局变量
* 局部变量
* 内置名称
* 导入到名称

RFT 模式不会修改的名称

* 参数名称
* 调用函数的时候使用的关键字名称
* 使用模块属性 ``__all__`` 输出的所有名称
* 所有以 ``__`` 开头的名称

使用 RFT 模式的时候，必须把所有使用的脚本和包在同一条命令进行加密。Pyarmor 会分析脚本之间的调用关系，对导入的名称也进行正确的重命名。例如::

    src/
        foo.py
        joker/
            __init__.py
            card.py

使用 RFT 模式加密脚本 foo.py 以及包 joker 的命令如下::

    pyarmor gen --enable-rft foo.py src/joker/

Pyarmor 使用内置的自动规则和人工配置的规则来分析脚本，并进行重命名。虽然 RFT 模式的自动规则可以处理大多数的情况，但是对于复杂的脚本和包，肯定会存在一些名称没有被正确处理。因为 Python 语言自身的特点，Pyarmor 并不认为可以找到算法自动处理所有的情况，所以对特殊情况，必须要进行人工配置规则。RFT 模式的目标是不断更新和完善自动规则，对于不能自动处理的部分，能够自动生成相关的参考配置，用户通过这些参考配置生成人工规则。

对于复杂的脚本，不要期望 Pyarmor 会自动进行处理所有情况。例如，

.. code-block:: python

    foo().stack[2].count = 3
    (a+b).tostr().get()

是否对属性 ``stack`` ， ``count`` ， ``tostr`` 和 ``get`` 进行重命名呢？在有些情况下，使用静态的语法分析根本无法判断属性所在的变量类型，因为有些表达式的类型是动态执行的时候确定的，甚至同一个表达式会在不同的情况下返回不同的类型。

为了处理这种情况，RFT 模式提供了两种方法:

- **rft-auto-exclude**

  自动排除未知属性，这是默认方法。

  这种方法是搜索所有的脚本中所有的属性链，如果属性所在的类型名称无法确定，那么把这个属性名称增加到排除属性表，所有被排除的属性在其他任何脚本中都不会被重命名。

  排除属性表存放的文件名称是 ``.pyarmor/rft_exclude_table``

  第一次运行 RFT 模式加密脚本的时候，这个表是空的。RFT 模式依次扫描各个脚本，发现无法处理的属性，就把它添加到这个表中。当所有的脚本都加密完成之后，这个表里面的名称就是所有无法处理的属性名称。

  这时候再次使用 RFT 模式使用相同的参数加密脚本，就可以保证所有的未知属性没有命名，基本不会出现名称绑定的错误。

  RFT 模式不会自动删除这个表 ``.pyarmor/rft_exclude_table`` ，而是不断增加新的未知属性到里面。如果有需要的话，可以把这个表删除掉，然后重新开始一个全新的重命名过程。

  这种方法使用比较简单，但是可能会排除很多名称，查看排除属性表可以知道那些属性名称被保留下来了。

- **rft-auto-include**

  自动命名所有的类和方法。

  这种方法是首先扫描全部脚本，把脚本中函数，类和方法全部都进行重命名，确保这些名称能被重命名。

  对于那些属性链中无法处理的属性名称，保留下来不进行处理。

  用户需要运行加密脚本，如果出现名称错误，那么把这些名称人工进行排除，或者使用人工规则进行排除。

  不断重复上述步骤，直到没有名称错误出现。

  这种方法能够重命名大部分的名称，但是需要更多额外的工作。

启用 RFT 模式
=============

使用下面的命令启用 RFT mode::

    $ pyarmor gen --enable-rft foo.py

也可以使用 :command:`pyarmor cfg` 通过配置文件启用::

    $ pyarmor cfg enable_rft=1
    $ pyarmor gen foo.py

启用 **rft-auto-include** 方法通过禁用 ``rft_auto_exclude``::

    $ pyarmor cfg rft_auto_exclude=0

重新启用 **rft-auto-exclude** 方法::

    $ pyarmor cfg rft_auto_exclude=1

查看重命名之后的完整脚本
========================

使用 RFT 模式修改后的脚本究竟能否满足安全需要？可能需要先看一下。

在 Python 3.9+ 启用 RFT 跟踪模式，可以输出相应的脚本::

    $ pyarmor cfg trace_rft 1
    $ pyarmor gen --enable-rft foo.py
    $ ls .pyarmor/rft

``foo.py`` 转换后对应的脚本是 ``.pyarmor/rft/foo.py``::

    $ cat .pyarmor/rft/foo.py

转换后的脚本表示的只是 RFT 模式所进行的修改，并不完全等价于加密后的脚本，它还会按照加密选项的设置进行下面的处理。例如 docstring 现在还保留着，但是如果使用了下面的配置项::

    $ pyarmor cfg optimize 2

加密脚本最终会把所有 DocString 删除。

.. note::

   Python 3.8 以及之前的版本不支持该功能。

查看重命名的名称
================

如果需要跟踪那些名称被重命名，可以同时启用日志跟踪和RFT 跟踪选项，这时候会生成跟踪日志 ``.pyarmor/pyarmor.trace.log`` ，里面有所有的重命名日志::

    $ pyarmor cfg enable_trace=1 trace_rft=1
    $ pyarmor gen --enable-rft foo.py
    $ grep trace.rft .pyarmor/pyarmor.trace.log

    trace.rft            foo:1 (import sys as pyarmor__1)
    trace.rft            foo:12 (self.wScan->self.pyarmor__4)

第一个日志记录了 ``sys`` 被重命名为 ``pyarmor__1``

第二条日志记录了属性 ``wScan`` 被重命名为 ``pyarmor__4``

人工排除属性名称
================

如果加密后的脚本出现名称错误，最简单的解决方式就是把这个名称人工排除，也就是说，在所有的脚本中保留这个名称（属性）而不进行重命名。下面的命令把名称 ``mouse_keybd`` 保留下来::

    $ pyarmor cfg rft_excludes + "mouse_keybd"
    $ pyarmor gen --enable-rft foo.py

如果错误显示的名称是像 ``pyarmor__22`` 这样的格式，那么首先通过跟踪日志反向查找出原来的名称::

    $ grep pyarmor__22 .pyarmor/pyarmor.trace.log

    trace.rft            foo:65 (self.height->self.pyarmor__22)
    trace.rft            foo:81 (self.height->self.pyarmor__22)

例如，上例中 ``height`` 是 ``pyarmor__22`` 原来的名称，那么使用下面的命令保留这个名称::

    $ pyarmor cfg rft_excludes + "height"
    $ pyarmor gen --enable-rft foo.py
    $ python dist/foo.py

如果需要把这些人工排除的名称清除，使用下面的命令::

    $ pyarmor cfg rft_excludes ""

处理模块属性 ``__all__``
========================

模块属性 ``__all__`` 中定义的名称会被保留下来，可以输出名称供外部脚本使用。例如，下面的函数 ``foo`` 会被保留下来不进行重命名

.. code-block:: python

    __all__ = ['foo']

    def foo(msg):
        print(msg)

    def _private_foo(msg):
        print(msg)

如果输出的是一个类，那么类中的方法和属性也会被保留下来。

有时候可能需要重命名所有的类和方法，可以使用下面的选项让 RFT 模式忽略 ``__all__`` 中定义的名称::

    $ pyarmor cfg rft_export__all__ 0

处理特殊的导入语句
==================

导入全部名称的语句 — ``from module import *`` — 会进行特殊的处理。

如果是从一个加密模块里面导入所有名称，那么 RFT 模式会直接解析源代码并直到属性 ``__all__`` 的定义。

如果是从一个外部模块中导入所有名称，那么 RFT 模式会尝试直接导入这个模块，并且尝试得到模式属性 ``__all__`` 。如果导入失败，会导致 RFT 模式报错退出。在这种情况下，需要设置环境变量 :envvar:`PYTHONPATH` 或者其他任何方式让 Python 可以导入这个模块。

如果模块属性 ``__all__`` 没有定义，那么模块的所有属性中不是以 ``_`` 开头的名称都会被导入进来。

自定义重命名规则
================

自定义规则可用于重命名无法自动识别的类型的属性名称，自定义规则仅适用于 **rft-auto-include**::

    $ pyarmor cfg rft_auto_exclude=0

定义全局重命名规则::

  pyarmor cfg rft_rulers ^ "self.task.x self.task.?"

定义模块私有重命名规则::

  pyarmor cfg -p joker.card rft_rulers "self.task.x self.task.?"

重命名规则格式::

  属性链模式 对应属性重命名方式

属性链模式是以 ``.`` 分开的 ``fnmatch`` 的模式字符串，然后使用此模式和属性链进行匹配

满足匹配的属性链按照后面的重命名方式进行处理

- ``?``: 总是进行重命名
- 其他值: 保留原来的名称

人工规则只用来修改属性名称，而忽略第一个名称

例如，下面的规则::

    self.task.x self.task.?

应用于下面的脚本

.. code-block:: python
    :linenos:
    :emphasize-lines: 8,9

    class Sdipmk:

        def __init__(self):
            self.width = 100
            self.height = 200

        def move(self, x, y, absolute=False):
            self.task.x = int(abs(x*65536/self.width)) if absolute else int(x)
            self.task.y = int(abs(y*65536/self.height)) if absolute else int(y)
            return Mouse(MS_MOVE, x, y)

使用下面的命令配置重命名规则::

    $ pyarmor cfg rft_rulers "self.task.x self.task.?"

然后检查结果::

    $ pyarmor gen --enable-rft foo.py
    $ grep trace.rft .pyarmor/pyarmor.trace.log

    trace.rft            foo:8 (self.task.x->self.task.pyarmor__2)

第 8 行的 ``self.task.x`` 被重命名为 ``self.task.pyarmor__2``

让我们修改一下重命名规则，然后在看看结果::

    $ pyarmor cfg rft_rulers "self.task.x self.?.?"
    $ grep trace.rft .pyarmor/pyarmor.trace.log

    trace.rft            foo:8 (self.task.x->self.pyarmor__1.pyarmor__2)

接下来增加一条新规则重命名 ``self.task.y`` ，注意使用 ``^`` 来增加规则::

    $ pyarmor cfg rft_rulers ^"self.task.y self.?.?"
    $ grep trace.rft .pyarmor/pyarmor.trace.log

    trace.rft            foo:8 (self.task.x->self.pyarmor__1.pyarmor__2)
    trace.rft            foo:9 (self.task.y->self.pyarmor__1.pyarmor__3)

这两条规则可以合并成为一条，这里使用 ``=`` 进行配置，会自动删除原来的所有规则::

    $ pyarmor cfg rft_rulers = "self.task.* self.?.?"
    $ grep trace.rft .pyarmor/pyarmor.trace.log

    trace.rft            foo:8 (self.task.x->self.pyarmor__1.pyarmor__2)
    trace.rft            foo:9 (self.task.y->self.pyarmor__1.pyarmor__3)

..
  类型字典和变量类型
  ==================

  模块，类在类型字典中定义，存放在 ``module_types``::
    mod
    pkg
    pkg.mod
    pkg.mod.Cls
    pkg.mod[.Cls|.func].Cls

  函数，变量和参数在变量字典中定义，存放在 ``variable_types``::

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

.. include:: ../_common_definitions.txt
