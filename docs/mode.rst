.. _加密模式:

加密模式
========

PyArmor 提供多种加密模式，以满足安全和性能方面的平衡。通常情况下，默认
的加密模式能够满足绝大多数的需要，一般情况下也无不需要对加密模式有详细
的了解。仅当对性能有特别的要求或者默认加密模式无法满足需求的时候，才需
要改变加密模式，这就需要理解 PyArmor 的不同加密模式。

.. _高级模式:

高级模式
--------

高级模式是从 PyArmor 5.5.0 开始增加的新功能，在这种模式下，加密脚本中
代码块的 `PyCode_Type` 的结构会被修改，同时在加密脚本运行的时候，会在
Python 动态库中注入一个钩子函数，来处理这种非正常的代码块。在注入钩子
函数的同时，也会检查 Python 解释器是否被修改，如果解释器行为不正常，加
密脚本就不会在继续执行。这个特性需要分析汇编指令，目前只在 X86/X64 系
列的 CPU 上实现，并且还依赖于编译器。一些使用低版本 GCC 编译的 Python
解释器可能无法被 PyArmor 正确识别，所以目前高级模式默认是没有启用的。
如果在使用过程中出现正常的 Python 解释器无法在高级模式下运行，欢迎报告
问题到 https://github.com/dashingsoft/pyarmor/issues 或者发送邮件到
jondy.zhao@gmail.com

使用下面的命令可以启用高级模式加密脚本::

    pyarmor obfuscate --advanced foo.py

在下一个主版本中高级模式有可能会默认启用。

对于使用老版本的用户来说，升级之前请确保生产环境的 Python 解释器支持高
级模式，参考下面文档来评估 Python 解释器

https://github.com/dashingsoft/pyarmor-core/tree/v5.3.0/tests/advanced_mode/README.md

建议已经在生产环境中使用加密脚本的用户在下一个主版本高级模式稳定之后在升级。

.. note::

   高级模式在试用版本中的限制是每一个模块中的函数（方法）等代码块总数
   不能超过大约 30 个左右，超过这个限制将无法被加密（但是依旧可以使用
   普通模式进行加密）。

.. _代码加密模式:

代码加密模式
------------

一个 Python 文件中，通常有很多个函数（代码块）

* obf_code == 0

不会加密函数对应的代码块

* obf_code == 1 （默认值）

在这种情况下，会加密每一个函数对应的代码块，根据 :ref:`代码包裹模式`
的设置，使用不同的加密方式

.. _代码包裹模式:

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

模块加密模式
------------

* obf_mod == 1 （默认值）

在这种模式下，最终生成的加密脚本如下::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 1)

其中第三个参数是代码块转换而成的字符串，它通过下面的伪代码生成::

    PyObject *co = Py_CompileString( source, filename, Py_file_input );
    obfuscate_each_function_in_module( co, obf_mode );
    char *original_code = marshal.dumps( co );
    char *obfuscated_code = obfuscate_algorithm( original_code  );
    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 1)", obfuscated_code );

* obf_mod == 0

在这种模式下，最终生成的代码如下（最后一个参数为 0）::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 0)

第三个参数的生成方式和上面的基本相同，除了最后一条语句替换为::

    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 0)", original_code );

默认情况下以上三种加密模式都是启用的，如果要改变加密模式，必须使用工程
来加密脚本。具体使用方法请参考工程的 :ref:`使用不同加密模式`

.. _约束模式:

约束模式
--------

在约束模式下，加密脚本必须是下面的形式之一::

    __pyarmor__(__name__, __file__, b'...')

    Or

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')

    Or

    from pytransform import pyarmor_runtime
    pyarmor_runtime('...')
    __pyarmor__(__name__, __file__, b'...')

例如，下面的加密脚本能够运行::

    $ cat a.py

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')

    $ python a.py

而下面的这个就无法运行，因为加密脚本中有一条额外的语句 `print`::

    $ cat b.py
    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')
    print(__name__)

    $ python b.py

从 PyArmor 5.2 开始, 约束模式是默认设置。如果需要禁用约束模式, 那么使
用下面的命令加密脚本::

    pyarmor obfuscate --restrict=0 foo.py

.. include:: _common_definitions.txt