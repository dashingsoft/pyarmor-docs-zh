.. highlight:: console

==================================
 在 CI/CD Pipeline 中使用 Pyarmor
==================================

.. deprecated:: 9.0

                请参考 :ref:`Using Pyarmor in CI Pipeline`

Pyarmor 也可以使用下面的方式在 CI/CD pipeline 加密脚本:

1. 首先使用少量 Runner 加密脚本，然后保存加密脚本到另外一个分支
2. 然后其他 Runner 都基于这个加密分支，按照原来的方式进行构建工作

假设我们的测试项目位于 `https://github.com/dashingsoft/test-project` ，其目录结构如下::

    $ tree test-project

    test-project
    └── src
        ├── main.py
        ├── utils.py
        └── parent
            ├── child
            │   └── __init__.py
            └── __init__.py

可以首先使用一个 Runner 来加密脚本，并保存到分支 `master-obf` ，下面是示例脚本:

.. code-block:: bash

    $ git clone https://github.com/dashingsoft/test-project

    $ pip install pyarmor
    $ pyarmor reg /path/to/pyarmor-regfile-5068.zip

    # 创建新分支
    $ git checkout -B master-obf

    # 创建输出目录 dist ，链接到项目目录
    $ ln -s test-project dist

    # 加密所有的脚本到输出目录 "dist" ，因为它等同于 "test-project"
    # 所以加密后的脚本 "dist/src/main.py" 会覆盖原来的脚本 "test-project/src/main.py"
    $ pyarmor gen -O dist -r --platform windows.x86_64,linux.x86_64,darwin.x86_64 test_project/src

    # 增加运行辅助包到这个分支
    $ git add -f test_project/pyarmor_runtime_5068/*

    # 提交加密结果和运行辅助包
    $ git commit -m'Build obfuscated scripts' .

    # 创建远程分支
    $ git push -u origin master-obf

对于其他 Runner，因为不需要注册 Pyarmor，所以只需要使用分支 `master-obf` 就基本可以和以前的方式一样不受限制的运行

.. include:: ../_common_definitions.txt
