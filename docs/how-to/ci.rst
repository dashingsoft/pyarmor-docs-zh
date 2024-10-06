.. highlight:: console

==================================
 在 CI/CD Pipeline 中使用 Pyarmor
==================================

.. versionchanged:: 8.6.0
    从 8.6.0 开始，专家版许可证不能直接应用在 CI/CD pipeline，需要使用专门的 CI 许可证

.. important::

   Pyarmor 8.6.0 对不同许可证在 CI/CD pipeline 的使用进行了重大调整

   在之前购买的 Pyarmor 8 的许可证如果不升级到 8.6.0，可以继续安装原来的方式在 CI/CD pipeline 中使用，最大可用的版本是 8.5.12, 使用文档请参考 `Pyarmor Doc 8.5.12`__

   如果需要升级 Pyarmor 到 8.6+，那么需要按照新的方式进行使用，其中最重要的变化是专家版许可证不能在用于 CI/CD pipeline，而是需要使用新的专门 CI 许可证进行代替

   __ https://pyarmor.readthedocs.io/zh/8.5.12/

从 Pyarmor 8.6.0 开始，Pyarmor 专家版和集团版许可证不可以应用于 CI/CD pipeline

Pyarmor 试用版，基础版和专用的 CI 许可证可以直接应用于 CI/CD pipeline

首先生成一个有效期为 3 天的注册文件，然后在 CI/CD pipeline 中使用此文件进行注册

CI/CD pipeline 需要联网验证时间是否过期

在过期之后，需要重新生成注册文件

Pyarmor 也可以使用下面的方式在 CI/CD pipeline 加密脚本:

1. 首先使用少量 Runner 加密脚本，然后保存加密脚本到另外一个分支
2. 然后其他 Runner 都基于这个加密分支，按照原来的方式进行构建工作

虽然这样的目的是为了可以解决老版本许可证使用次数的约束问题，但是同样也可以在新版本中使用

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
