.. _加密脚本的性能:

加密脚本的性能
==============

运行命令 :ref:`benchmark` 可以检查加密脚本的性能::

    pyarmor benchmark

下面是输出结果的示例::


    INFO     PyArmor Trial Version 6.3.0
    INFO     Python version: 3.7
    INFO     Start benchmark test ...
    INFO     Obfuscate module mode: 1
    INFO     Obfuscate code mode: 1
    INFO     Obfuscate wrap mode: 1
    INFO     Obfuscate advanced mode: 0
    INFO     Benchmark bootstrap ...
    INFO     Benchmark bootstrap OK.
    INFO     Run benchmark test ...

    Test script: bfoo.py
    Obfuscated script: obfoo.py
    --------------------------------------

    import_first_no_obfuscated_module                 :   6.177000 ms
    import_first_obfuscated_module                    :  15.107000 ms

    re_import_no_obfuscated_module                    :   0.004000 ms
    re_import_obfuscated_module                       :   0.005000 ms

    --- Import 10 modules ---
    import_many_no_obfuscated_modules                 :  58.882000 ms
    import_many_obfuscated_modules                    :  50.592000 ms

    run_empty_no_obfuscated_code_object               :   0.004000 ms
    run_empty_obfuscated_code_object                  :   0.003000 ms

    run_no_obfuscated_1k_bytecode                     :   0.010000 ms
    run_obfuscated_1k_bytecode                        :   0.027000 ms

    run_no_obfuscated_10k_bytecode                    :   0.053000 ms
    run_obfuscated_10k_bytecode                       :   0.119000 ms

    call_1000_no_obfuscated_1k_bytecode               :   2.411000 ms
    call_1000_obfuscated_1k_bytecode                  :   3.735000 ms

    call_1000_no_obfuscated_10k_bytecode              :  32.067000 ms
    call_1000_obfuscated_10k_bytecode                 :  42.164000 ms

    call_10000_no_obfuscated_1k_bytecode              :  22.387000 ms
    call_10000_obfuscated_1k_bytecode                 :  36.666000 ms

    call_10000_no_obfuscated_10k_bytecode             : 307.478000 ms
    call_10000_obfuscated_10k_bytecode                : 407.585000 ms

    --------------------------------------
    INFO     Remove test path: ./.benchtest
    INFO     Finish benchmark test.

PyArmor 使用一个简单脚本 `bfoo.py` 来进行测试，里面有两个函数，

* one_thousand: 它的代码大小大约是 1K
* ten_thousand: 它的代码大小大约是 10K

第一个测试 `import_first_obfuscated_module` 包含了动态库的初始化，以及验证许可证
等其他额外时间，所以比普通脚本时间要长一些。而 `import_many_obfuscated_modules`
，这个测试中把原来的脚本拷贝成为 10 个其他脚本，然后使用新的名称导入，它花费的时
间就明显比没有加密的要快很多，这主要是因为加密脚本已经是编译好的代码，省去了编译
时间，而加密消耗的额外时间小于编译时间。

其他的测试，例如 `call_1000_no_obfuscated_1k_bytecode` ，它的意思是调用 1000 次
函数 `one_thousand` 所消耗的时间，通过比较 `call_1000_obfuscated_1k_bytecode` 的
结果，可以大概了解到加密后脚本性能。需要注意的是测试结果和多个因素有关，包括测试
脚本，加密模式，Python 版本等等，即便是在相同的机器，运行相同的命令，结果也会有
些微差别，所以需要根据实际环境下运行结果来进行评估。

另外也可以测试不同加密模式下的性能，使用下面的选项查看所有支持的模式::

    pyarmor benchmark -h

然后传入不同的参数，测试不同加密模式的性能。例如::

    pyarmor benchmark --wrap-mode 0 --obf-code 2

查看测试命令使用的脚本，使用选项 ``--debug`` 保留生成的中间文件，所有的中间文件
保存在目录 `.benchtest` 下面::

    pyarmor benchmark --debug

.. include:: _common_definitions.txt
