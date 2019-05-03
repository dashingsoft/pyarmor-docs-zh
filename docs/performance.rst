.. _加密脚本的性能:

加密脚本的性能
==============

运行命令 `benchmark` 可以检查加密脚本的性能::

    pyarmor benchmark

下面是输出结果的示例::

    INFO     Start benchmark test ...
    INFO     Obfuscate module mode: 1
    INFO     Obfuscate code mode: 1
    INFO     Obfuscate wrap mode: 1
    INFO     Benchmark bootstrap ...
    INFO     Benchmark bootstrap OK.
    INFO     Run benchmark test ...
    Test script: bfoo.py
    Obfuscated script: obfoo.py
    --------------------------------------

    load_pytransform: 28.429590911694085 ms
    init_pytransform: 10.701080723946758 ms
    verify_license: 0.515428636879825 ms
    total_extra_init_time: 40.34842417122847 ms

    import_no_obfuscated_module: 9.601499631936461 ms
    import_obfuscated_module: 6.858413569322354 ms

    re_import_no_obfuscated_module: 0.007263492985840059 ms
    re_import_obfuscated_module: 0.0058666674116400475 ms

    run_empty_no_obfuscated_code_object: 0.015085716201360122 ms
    run_empty_obfuscated_code_object: 0.0058666674116400475 ms

    run_one_thousand_no_obfuscated_bytecode: 0.003911111607760032 ms
    run_one_thousand_obfuscated_bytecode: 0.005307937181960043 ms

    run_ten_thousand_no_obfuscated_bytecode: 0.003911111607760032 ms
    run_ten_thousand_obfuscated_bytecode: 0.005587302296800045 ms

    --------------------------------------
    INFO     Remove test path: .\.benchtest
    INFO     Finish benchmark test.

其中额外的初始化时间大约是 `40ms` ，这包括装载动态库、初始化动态库和校
验授权文件的总时间。

上面结果中，导入加密模块的时间还少于导入正常模块的时间，这主要是因为加
密脚本已经被编译成为字节码文件，而原始文件需要额外的时间来进行编译。

这里执行加密函数需要的额外时间一般在 `0.002ms` 左右，也就是执行 `1000`
个函数，加密脚本额外消耗的时间大约为 `2ms` 。不同的机器可能结果不同，
需要根据实际环境下运行结果来进行评估。

.. include:: _common_definitions.txt
