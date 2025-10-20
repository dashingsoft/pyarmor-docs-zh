.. highlight:: console

=============================
 在 CI/CD 管线中使用 Pyarmor
=============================

:ref:`直接使用方式` 基本不改变原来 CI/CD 的管线流程，适用于

- 试用版
- :term:`Pyarmor 基础版`
- :term:`Pyarmor 管线版`

:ref:`间接使用方式` 需要改变原来的 CI/CD 管线流程，任何许可证都可以使用

直接使用方式
============

**试用版** 可以直接在 CI/CD 管线中使用，只需要增加一个步骤::

      pip install pyarmor

:term:`Pyarmor 基础版` 和 :term:`Pyarmor 管线版` 许可证

- 首先完成 :ref:`initial registration` ，得到许可证 :term:`注册文件` ``pyarmor-regfile-xxxx.zip``
- 其次在本地联网设备上申请 :term:`管线注册文件`::

      $ pyarmor reg -C pyarmor-regfile-xxxx.zip

  申请一次即可，该命令执行成功之后会在当前目录创建 :term:`管线注册文件` ``pyarmor-ci-xxxx.zip``

  在本地设备上面查看管线许可证的相关信息::

      $ pyarmor --home temp reg pyarmor-ci-xxxx.zip

- 在管线中增加如下命令::

      # 请替换 9.X.Y 为当前 Pyarmor 的版本
      pip install pyarmor==9.X.Y
      parmor reg pyarmor-ci-xxxx.zip

  在管线中查看注册信息::

      pyarmor -v

注意事项

- 管线注册文件有效期为一年，可以用于在 CI/CD 中注册 Pyarmor
- 管线注册文件在当前 Pyarmor 的版本有效，但是不一定在升级之后的 Pyarmor 中有效
- 每一个许可证最多可以申请 100 个管线注册文件，超过申请次数之后就无法在申请
- 请勿在 CI/CD 管线中申请管线注册文件，因为请求次数有一定限制

.. important::

   每次管线运行都会向 Pyarmor 许可证发送验证请求，过多的请求或者超过正常使用的请求可能会被拒绝，目前的限制是

   - 每秒至多 1 次注册
   - 每小时至多 100 次注册
   - 每天至多 1,000 次注册
   - 每月至多 10,000 次注册

   只允许在开发设备上安装和注册 Pyarmor，不允许在需要发送给客户的 Docker 镜像中安装和注册 Pyarmor

.. note::

   在 GitHub Action 中需要额外的一个操作步骤，否则会报错 `CI license only works in CI/CD pipeline`

   1. 对于 ubuntu ，增加操作步骤::

        - run: sudo mv /dev/disk /dev/disk-none

   2. 对于 darwin ，增加操作步骤::

        - run: sudo mv /dev/rdisk0 /dev/rdisk0-none

   请参考这里 `Error when using CI license in CI pipeline <https://github.com/dashingsoft/pyarmor/discussions/2004>`_

间接使用方式
============

因为加密后的脚本等价于原来的脚本外加运行辅助包，所以可以首先在本地设备对脚本进行加密，然后把加密后的脚本保存到一个单独的分支，CI/CD 管线流程直接从加密分支中获取数据，这种方式适用于任何许可证

下面我们用一个示例来进行简单说明

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

可以首先在本地加密脚本，并保存到分支 `master-obf` ，下面是示例脚本:

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

对于 CI/CD 管线来说，只需要使用分支 `master-obf` 就基本可以和以前的方式一样不受限制的运行

.. include:: ../_common_definitions.txt
