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

但是超级模式不是所有的 Python 版本都支持的，目前只有最新的版本被支持：

* Python 2.7
* Python 3.7
* Python 3.8

Python 3.5，3.6 有可能随后会被支持，但是 Python 3.0 ~ 3.4 不会被支持。

使用下面的命令可以启用超级模式加密脚本::

    pyarmor obfuscate --advanced 2 foo.py

更多使用方法参考 :ref:`使用超级模式加密脚本` 。

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

.. note::

   如果在使用过程中出现正常的 Python 解释器无法在高级模式下运行，欢迎
   报告问题到 https://github.com/dashingsoft/pyarmor/issues 或者发送邮
   件到 jondy.zhao@gmail.com

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
者开始运行一个加密模块中的函数的时候，首先会检查约束模式，如果违反了约束，那么就
会抛出保护异常。

约束模式共用四种类型，其中模式 2 和 3 主要用于保护可以单独运行的脚本，而模式 4
主要用于保护加密的 Python 包。在同一个项目中，可以根据需要为不同的脚本设置不同的
约束模式。

* 模式 1

使用这个模式加密的脚本不允许别人修改，在加密模块加载的时候会检查代码，如果被修改
过，那么就会抛出保护异常。

例如，插入一条 `print` 语句在加密脚本 `foo.py`::

    __pyarmor__(__name__, __file__, b'...', 1)
    print('This is obfuscated module')

这个脚本就无法被运行或者导入。

* 模式 2

使用这个模式加密的脚本不允许别人使用 `import` 导入，只能使用 Python 解释器直接运
行，其主要作用是保护主脚本不能被其他用户随便导入使用。

例如，使用模式 2 加密的脚本 `foo2.py` ，可以使用下面的方式运行::

    python foo2.py

但是不论是通过脚本，还是直接从命令行导入这个加密脚本，都会报错。

例如，下面的命令会抛出保护异常::

    python -c'import foo2'

* 模式 3

模式 3 是模式 2 的增强版，除了不允许被其他用户导入之外，这个模块中的任意一个函数
被调用的时候，也会对调用者进行检查。如果调用者是被加密的兄弟脚本，那么允许调用，
否则也会抛出保护异常。

例如，使用模式 3 加密的两个脚本 `foo3.py` ， `mod3.py` 。前者是主脚本，会调用后
者里面定义的函数。如果尝试使用下面的脚本 `test.py` 来调用 `mod3`::

    import mod3
    mod3.hello()

运行 `python test.py` ，当执行第一行 `import` 语句的时候，PyArmor 会对主脚本
`hello.py` 进行检查，发现其不是加密脚本，会抛出保护异常。

需要注意的如果 `mod3` 之前已经被导入到系统中，那么执行 `import` 语句的时候，
PyArmor 不会对主脚本进行检查。

而第二行调用 `mod3.hello` 的时候，PyArmor 会再次检查调用者 `test.py` ，如果发现
其不是加密脚本，那么就会抛出保护异常。

* 模式 4

和模式 3 类似，这个模块中的任意一个函数被调用的时候，会对调用者进行检查。但是这
种模式加密的模块在导入的时候不会对主脚本进行检查，允许主脚本是没有加密的脚本。

一般用于加密 Python 包的部分脚本，以提高加密脚本安全性。

典型的应用是使用约束模式 1 加密 Python 包中 `__init__.py` 和其他需要被
外部使用的脚本，而使用约束模式 4 来加密那些只是在包内部使用的脚本。

.. note::

   约束模式 2 和 3 不能用于加密 Python 包，否则加密后的包是无法被非加
   密的脚本导入的。

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
