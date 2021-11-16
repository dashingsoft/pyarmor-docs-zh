.. _加密模式:

.. _the modes of obfuscated scripts:

加密模式
========

PyArmor 提供多种加密模式，以满足安全和性能方面的平衡。通常情况下，默认
的加密模式能够满足绝大多数的需要，一般情况下也无不需要对加密模式有详细
的了解。仅当对性能有特别的要求或者默认加密模式无法满足需求的时候，才需
要改变加密模式，这就需要理解 PyArmor 的不同加密模式。

.. _super mode:

.. _超级模式:

超级模式
--------

超级模式是从 PyArmor 6.2.0 开始增加的新特征。在这种模式下，加密脚本中的代码块结
构会被改变，并且会对操作码进行映射，是目前安全级别最高的一种模式。在超级模式下面，
运行辅助文件只需要一个扩展模块 ``pytransform`` ，而不在需要其他任何运行辅助文件，
加密后的脚本统一为如下格式::

    from pytransform import pyarmor
    pyarmor(__name__, __file__, b'\x0a\x02...', 1)

但是超级模式不是所有的 Python 版本都支持的，只有最新的版本被支持：

* Python 2.7
* Python 3.7
* Python 3.8
* Python 3.9

使用下面的命令可以启用超级模式加密脚本::

    pyarmor obfuscate --advanced 2 foo.py

更多使用方法参考 :ref:`使用超级模式加密脚本` 。

.. note::

   超级模式和非超级模式加密的脚本不能混合使用

.. _终极模式:

终极模式
--------

终极模式是基于超级模式的一种加强版，会把部分函数直接转换成为二进制代码。这个功能
是在 PyArmor 7.0.1 中增加的，目前只支持 X86_64 架构和 Python 3.7，3.8，3.9

使用之前需要配置 `c` 编译器，对于 `Linux` 和 `Darwin` 来说，一般不需要进行配置，
只要默认的 ``gcc`` 和 ``clang`` 能工作就可以。在 `Windows` 环境下面，可以使用下
面任意一种方式配置 ``clang.exe`` ，目前其它编译器还不支持

* 如果已经有 ``clang.exe`` ，只要在其它路径直接运行 ``clang.exe`` 不出错就可以。
  如果文件存在，但是无法在任意路径直接运行，可以配置环境变量 ``PYARMOR_CC`` 来指
  定这个文件，例如::
    set PYARMOR_CC=C:\path\to\clang.exe

* 从 `LLVM 官网 <https://releases.llvm.org>`_ 下载并安装预编译版本
* 从 PyArmor 官网下载 `clang-9.0.zip
  <https://pyarmor.dashingsoft.com/downloads/tools/clang-9.0.zip>`_ ，压缩包大小
  约为 26M 左右，里面只有一个可执行文件，解压后存放在 ``$HOME/.pyarmor`` 下面即可

配置好之后使用 ``--advanced 5`` 来进行加密，例如::

  pyarmor obfuscate --advanced 5 foo.py

不是模块内的所有函数都会使用终极模式加密，只有部分满足条件的函数才使用终极模式进
行加密，其它的所有代码依旧使用超级模式进行加密。任何函数如果使用了不被终极模式支
持的特性，都将自动使用超级模式加密。万一某一个模块使用终极模式加密之后出现问题，
可以在模块头部指定选项来忽略整个模块::

  # pyarmor options: no-spp-mode

终极模式会从文件头部开始扫描，依次解析以 ``#`` 开头的行，遇到其它非空行则结束扫
描。如果扫描到包含 ``pyarmor options`` 开头的行，会读取后面的选项并进行处理。

这个选项也可以用在函数和类的文档中来忽略一个特定的函数和类。例如:

.. code-block:: python

    def foo(a, b):
        '''pyarmor options: no-spp-mode'''
        pass

终极模式和原来的脚本存在一些不同:

* 在正常执行的函数调用没有参数的 `raise` 抛出的异常类型不同

.. code-block:: python

    >>> raise
    RuntimeError: No active exception to reraise

    # 在终极模式中
    >>> raise
    UnboundlocalError: local variable referenced before assignment

.. note::

   终极模式在试用版中不可用。

.. _高级模式:

.. _advanced mode:

高级模式
--------

高级模式是从 PyArmor 5.5.0 开始增加的新功能，在这种模式下，加密脚本中
代码块的 `PyCode_Type` 的结构会被修改，同时在加密脚本运行的时候，会在
Python 动态库中注入一个钩子函数，来处理这种非正常的代码块。在注入钩子
函数的同时，也会检查 Python 解释器是否被修改，如果解释器行为不正常，加
密脚本就不会在继续执行。这个特性需要分析汇编指令，目前只在 X86/X64 系
列的 CPU 上实现，并且还依赖于编译器。一些使用低版本 GCC 编译的 Python
解释器可能无法被 PyArmor 正确识别，所以目前高级模式默认是没有启用的。

使用下面的命令可以启用高级模式加密脚本::

    pyarmor obfuscate --advanced 1 foo.py

对于使用老版本的用户来说，升级之前请确保生产环境的 Python 解释器支持高
级模式，参考下面文档来评估 Python 解释器

https://github.com/dashingsoft/pyarmor-core/tree/v5.3.0/tests/advanced_mode/README.md

建议已经在生产环境中使用加密脚本的用户在下一个主版本高级模式稳定之后在升级。

.. note::

   高级模式在试用版本中的限制是每一个模块中的函数（方法）等代码块总数
   不能超过大约 30 个左右，超过这个限制将无法被加密（但是依旧可以使用
   普通模式进行加密）。

.. important::

   从 Python3.9 不在支持高级模式，对于所有支持超级模式的 Python 版本，推荐使用超
   级模式。

.. _vm mode:

.. _虚拟模式:

虚拟模式
--------

虚拟模式是 6.3.3 才新增加的特性，主要是核心算法使用虚拟代码来实现从而提高安全性，
目前只支持 Windows 平台。虚拟模式不能单独使用，它是高级模式和超级模式的增强版，
分别用于增强两种模式下动态库的安全性。

使用下面的命令同时启用高级模式和虚拟模式::

    pyarmor obfuscate --advanced 3 foo.py

使用下面的命令可以同时启用超级模式和虚拟模式::

    pyarmor obfuscate --advanced 4 foo.py

虽然虚拟模式能显著的增强安全性，但是动态库变胖并且性能有所下降。原来的动态库的大
小大约为 600K~800K 左右，虚拟模式的动态库则一般在 4M 左右。关于对性能的影响，可
以参考 :ref:`加密脚本的性能` 进行实际的测试。

.. _代码加密模式:

.. _obfuscating code mode:

代码加密模式
------------

一个 Python 文件中，通常有很多个函数（代码块）

* obf_code == 0

不会加密函数对应的代码块

* obf_code == 1 （默认值）

在这种情况下，会加密每一个函数对应的代码块，根据 :ref:`代码包裹模式`
的设置，使用不同的加密方式

* obf_code == 2

和 obf_mode 1 类似，但是使用更为复杂的算法来加密代码块（bytecode)，所
以比前者要慢一些。

.. _代码包裹模式:

.. _wrap mode:

代码包裹模式
------------

.. note::

    代码包裹模式只适用于非超级模式，超级模式总是要启用包裹模式的。

* wrap_mode == 0

当包裹模式关闭，代码块使用下面的方式进行加密::

    0   JUMP_ABSOLUTE            n = 3 + len(bytecode)

    3    ...
         ... 这里是加密后的代码块
         ...

    n   LOAD_GLOBAL              ? (__armor__)
    n+3 CALL_FUNCTION            0
    n+6 POP_TOP
    n+7 JUMP_ABSOLUTE            0

在开始插入了一条绝对跳转指令，当执行加密后的代码块时

1. 首先执行 JUMP_ABSOLUTE, 直接跳转到偏移 n

2. 偏移 n 处是调用一个 PyCFunction `__armor__` 。这个函数的功能会恢复
   上面加密后的代码块，并且把代码块移动到最开始（向前移动3个字节）

3. 执行完函数之后，跳转到偏移 0，开始执行原来的函数。

这种模式下，除了函数的第一次调用需要额外的恢复之外，随后的函数调用就和
原来的代码完全一样。

* wrap_mode == 1 （默认值）

当打开包裹模式之后，代码块会使用 `try...finally` 语句包裹起来::

    LOAD_GLOBALS    N (__armor_enter__)     N = co_consts 的长度
    CALL_FUNCTION   0
    POP_TOP
    SETUP_FINALLY   X (jump to wrap footer) X = 原来代码块的长度

    这里是加密的后的代码块

    LOAD_GLOBALS    N + 1 (__armor_exit__)
    CALL_FUNCTION   0
    POP_TOP
    END_FINALLY

这样，当被加密的函数开始执行的时候

1. 首先会调用 `__armor_enter__` 恢复加密后的代码块

2. 然后执行真正的函数代码

3. 在函数执行完成之后，进入 `final` 块，调用 `__armor_exit__` 重新加密
   函数对应的代码块

在这种模式下，函数的每一次执行完成都会重新加密，每一次执行都需要恢复。

.. _模块加密模式:

.. _obfuscating module mode:

模块加密模式
------------

* obf_mod == 1

在这种模式下，最终生成的加密脚本如下::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 1)

其中第三个参数是代码块转换而成的字符串，它通过下面的伪代码生成::

    PyObject *co = Py_CompileString( source, filename, Py_file_input );
    obfuscate_each_function_in_module( co, obf_mode );
    char *original_code = marshal.dumps( co );
    char *obfuscated_code = obfuscate_algorithm( original_code  );
    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 1)", obfuscated_code );

* obf_mod == 2 （默认值）

和上一中方式类似，只是使用不同的加密算法，但是安全性更高，性能更好，是从 6.3.0
新增加的。

* obf_mod == 0

在这种模式下，最终生成的代码如下（最后一个参数为 0）::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 0)

第三个参数的生成方式和上面的基本相同，除了最后一条语句替换为::

    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 0)", original_code );

默认情况下以上三种加密模式都是启用的，如果要改变加密模式，必须使用工程
来加密脚本。具体使用方法请参考工程的 :ref:`使用不同加密模式`

.. _约束模式:

.. _restrict mode:

约束模式
--------

约束模式是和具体的脚本相关的，用来限制脚本的使用方式。当加密模块被导入的时候，或
者开始使用加密模块中的函数和属性的时候，首先会检查约束模式，如果违反了约束，那么
就会抛出保护异常。

约束模式共用五种类型，其中模式 2 和 3 主要用于保护可以单独运行的脚本，而模式 4
主要用于保护加密的 Python 包。模式 5 安全性最高，两者都适用。在同一个项目中，可
以根据需要为不同的脚本设置不同的约束模式。

* 模式 1

使用这个模式加密的脚本不允许别人修改，在加密模块加载的时候会检查代码，如果被修改
过，那么就会抛出保护异常。

例如，插入一条 `print` 语句在加密脚本 `foo.py`::

    __pyarmor__(__name__, __file__, b'...', 1)
    print('This is obfuscated module')

这个脚本就无法被运行或者导入。

* 模式 2

使用这个模式加密的脚本不允许别人使用 `import` 导入，只能使用 Python 解释器直接运
行，或者被其他加密脚本导入，并且主脚本也必须加密脚本，否则使用这个模式加密的脚本
就无法被导入。

例如，使用模式 2 加密的脚本 `foo2.py` ，可以使用下面的方式运行::

    python foo2.py

但是不论是通过脚本，还是直接从命令行导入这个加密脚本，都会报错。

例如，下面的命令会抛出保护异常::

    python -c'import foo2'

* 模式 3

模式 3 是模式 2 的增强版，除了不允许被其他用户导入之外，这个模块中的属性和函数调
用也会受到保护。访问模块的属性或者调用其中的函数的时候，会对调用者进行检查。如果
调用者是被加密的兄弟脚本，那么允许调用，否则也会抛出保护异常。

* 模式 4

和模式 3 类似，会对模块的属性和函数调用进行保护。唯一的区别是允许主脚本是没有加
密的脚本。一般用于加密 Python 包的部分脚本，以提高加密脚本安全性。

典型的应用是使用约束模式 1 加密 Python 包中 `__init__.py` 以供外部使用，而使用约
束模式 4 来加密其他的那些仅在包内部使用的模块。

例如，有一个包 `mypkg`::

    mypkg/
        __init__.py
        private_a.py
        private_b.py

在脚本 ``__init__.py`` 中定义公开的函数和属性:

.. code:: python

    from . import private_a as ma
    from . import private_b as mb

    public_data = 'welcome'

    def proxy_hello():
        print('Call private hello')
        ma.hello()

    def public_hello():
        print('This is public hello')

而在脚本 ``private_a.py`` 中定义私有的方法和属性:

.. code:: python

    import sys

    password = 'xxxxxx'

    def hello():
        print('password is: %s' % password)

然后使用模式 1 加密 ``__init__.py`` ，使用模式 4 加密其他模块，保存到 `dist`::

    dist/
        __init__.py
        private_a.py
        private_b.py

现在直接使用 Python 解释器进行一些测试:

.. code:: python

    import dist as mypkg

    # 下面这些应该能够正常工作
    mypkg.public_hello()
    mypkg.proxy_hello()
    print(mypkg.public_data)
    print(mypkg.ma)

    # 下面这些应该抛出保护异常
    mypkg.ma.hello()
    print(mypkg.ma.password)

* 模式 5 (v6.4.0 新增)

模式 5 是模式 4 的增强版，会对运行堆栈中全局变量进行保护。当运行模式 5 中任意函
数的时候，外部脚本就无法访问函数的全局变量。这是安全性最高的一种约束模式，即可用
于保护独立脚本，也可用于保护包。但是因为要对全局变量进行约束，对性能可能会有影响。

* 模式 100+ (v6.7.4 新增)

这个模式是模式 1-5 的增强版，主要是增加了一个新的约束： **模块字典约束**

模式 101 等价于模式 1 和这个新约束，模式 102 等价于模式 2 和这个新约束等等

这个约束是的模块字典的行为就像是一个空的字典一样，即使是在加密脚本中，直接访问模
块字典会返回空的结果。但是也存在一些缺点

* 模块被清理的时候， `module.__dict__` 里面所有的对象不会被释放
* 模块被导入之后，就再也不能显示或者隐式的插入或者删除 `module.__dict__` 里面的键
* 函数 `dirs` ， `vars` 等也会返回空

例如:

.. code:: python

    # 这是模块文件 foo6.py ，使用模式 105 进行加密

    # 下面语句都可以正常使用
    global var_a
    var_a = 'This is global variable a'
    var_b = 'This is global variable b'
    del var_a

    def fabico():
        global var_b

        # 错误，删除模块属性将隐式修改模块字典，只能在模块级别删除全局变量
        del var_b

    # 这是脚本 foo.py ，使用模式 101 进行加密

    import foo6

    # 输出结果是 {}
    foo6.__dict__

    # 输出结果是 {}
    vars(foo6)

    # 输出结果是 []
    dirs(foo6)

    # 正确，可以访问修改已经存在的属性
    foo6.var_a = 'Changed by foo'

    # 错误，新增属性到模块字典，可能导致未知结果
    foo6.__dict__['var_c'] = 1
    foo6.var_d = 2

并且该约束只适用于 Python 3.7 以及其后的版本，对之前的 Python 版本没有效果。

.. important::

   在 v6.3.7 中有一个重要的改进，使用约束模式 3 或者 4 进行加密模块，其数据属性
   也会受到保护，没有加密的脚本将无法访问模块中的任何属性。在前面的版本里面，主
   要是对约束模块中的函数能严格进行限制，对于数据属性没有进行约束。

   不要在公开模块 ``__init__.py`` 中导入私有模块的函数或者类，因为只有模块的属性
   是被保护的，其他类型的属性是没有被保护的::

       # 正确，导入模块名称
       from . import private_a as ma

       # 错误，函数 `hello` 的所有属性可以被外部脚本访问
       from .private_a import hello

.. note::

   约束模式 2 和 3 不能用于加密 Python 包，否则加密后的包是无法被非加密的脚本导
   入的。

.. note::

   约束模式是针对单个脚本的，不同的脚本可以有不同的约束模式。

从 PyArmor 5.2 开始, 约束模式 1 是默认设置。

如果需要使用其他约束模式加密脚本，通过选项 ``--restrict`` 指定。例如::

    pyarmor obfuscate --restrict=2 foo.py
    pyarmor obfuscate --restrict=4 foo.py

    # For project
    pyarmor config --restrict=2
    pyarmor build -B

如果需要禁用上面全部的约束, 那么使用下面的命令加密脚本::

    pyarmor obfuscate --restrict=0 foo.py

    # For project
    pyarmor config --restrict=0
    pyarmor build -B

如果使用了命令 :ref:`licenses` 创建了自定义的许可证，那么禁用各种约束
需要在生成许可证的时候使用选项 ``--disable-restrict-mode`` ，例如::

    pyarmor licenses --disable-restrict-mode r001

    pyarmor obfuscate --with-license=licenses/r001/license.lic foo.py

    # For project
    pyarmor config --with-license=licenses/r001/license.lic
    pyarmor build -B

详细示例请参考 :ref:`使用约束模式增加加密脚本安全性`

从 PyArmor 5.7.0 开始，还有另外一个约束， :ref:`引导代码` 必须在加密脚本中，并且
加密脚本还必须是主脚本。例如，同一个目录下有两个文件 `foo.py` 和 `test.py` ，使
用下面的命令加密::

    pyarmor obfuscate foo.py

如果直接往加密后的脚本 `dist/test.py` 中插入 `引导代码` ，这个加密脚本就无法使用，
因为这个脚本加密的时候没有被指定为主脚本。只能使用下面的命令来插入 `引导代码`::

    pyarmor obfuscate --no-runtime --exact test.py

如果需要在没有加密的脚本中运行 :ref:`引导代码` ，可以使用一种变通方式。首先加密
一个空脚本::

    echo "" > pytransform_bootstrap.py
    pyarmor obfuscate --no-runtime --exact pytransform_bootstrap.py

然后在导入这个加密后的空脚本 `import pytransform_bootstrap` 。

.. include:: _common_definitions.txt
