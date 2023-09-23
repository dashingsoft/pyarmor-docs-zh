============
 使用许可证
============

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor reg

前提条件
========

在注册许可证之前，需要满足下列条件

1. 一个许可证的 :term:`激活文件` ，名称一般为 :file:`pyarmor-regcode-xxxx.txt` ，参考 :doc:`../licenses` 购买合适的许可证
2. 安装 Pyarmor 8.2 以上的版本的设备，
3. 当前设备需要能够访问互联网
4. 确定许可证需要绑定的产品名称，如果是在非商业化产品中使用，产品名称使用 ``non-profits``
5. 如果有防火墙，那么在 Windows 下面，防火墙要允许动态库 ``pytransform3.pyd`` 访问 `pyarmor.dashingsoft.com` 的端口 ``80`` ，在其他系统，防火墙要允许 ``pytransform3.so`` 访问 `pyarmor.dashingsoft.com` 的端口 ``80``，具体防火墙的规则设置请参阅防火墙的文档。

使用基础版和专家版许可证
========================

基本使用步骤：

1. 使用 :term:`激活文件` 进行初始登记，设定许可证绑定的产品名称
2. 初始登记完成之后会生成相应的 :term:`注册文件` ，后续相关的注册都将使用这个文件
3. 在其他设备使用 :term:`注册文件` 进行注册

初始登记
--------

在第一次注册的时候，需要使用 :option:`-p` 指定许可证绑定的产品名称，如果在非商业化产品中的使用 Pyarmor，那么产品名称设定为 ``non-profits`` ，该许可证不能被应用于商用产品中。

假设许可证绑定的产品名称为 ``XXX`` ，那么使用下面的命令进行初始登记注册::

    $ pyarmor reg -p "XXX" pyarmor-regcode-xxxx.txt

Pyarmor 会首先显示注册信息并请求确认，如果确认无误，输入 :kbd:`yes` 并 :kbd:`Enter` 继续下面的注册步骤，其他任何输入都会终止注册过程。

登记成功之后会同时生成一个相应的 :term:`注册文件` ``pyarmor-regfile-xxxx.zip`` ，这个文件被用来在其他设备上进行注册。

请不要再次使用 :term:`激活文件` 进行后续的注册，在初始登记之后这个文件失效，后续在任何设备上注册请使用 :term:`注册文件` 。

请妥善保管并备份注册文件，一旦丢失，Pyarmor 并不提供找回服务。

一旦登记成功，许可证绑定的产品名称就不能在进行修改。

绑定许可证到正在开发的产品
--------------------------

如果产品还在开发中，没有正式开始销售，允许初始化登记的时候设定产品的名称为 ``TBD`` 。例如::

    $ pyarmor reg -p "TBD" pyarmor-regcode-xxxx.txt

绑定的产品名称为 ``TBD`` 的许可证，必须在六个月之内，使用下面的命令设置为正确的产品名称::

    $ pyarmor reg -p "XXX" pyarmor-regcode-xxxx.txt

如果六个月之内还没有进行修改，那么产品名称会被自动设定为 ``non-profits`` ，并且不能在被修改。

在其他设备上面注册
------------------

拷贝 :term:`注册文件` ``pyarmor-regfile-xxxx.zip`` 到其他需要注册的设备，然后运行下面的命令进行注册::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

查看注册信息::

    $ pyarmor -v

注册成功之后所有的加密操作自动应用当前许可证，每一次加密操作需要联网验证许可证。

在 Docker 或者 CI pipeline 中注册
---------------------------------

在 Docker 或者 CI pipeline 中注册 Pyarmor 的基本方法同上，但是同时运行的 Pyarmor 的 Docker 数量有限制，不能超过 100 个。如果需要同时运行超过 100 个的 Docker 容器，请使用集团版许可证。

**只允许在开发设备上安装和注册 Pyarmor。如果 Docker 镜像需要发送给客户，那么不允许在上面安装和注册 Pyarmor**

.. _using group license:

使用集团版许可证
================

.. versionadded:: 8.2

每一个集团版许可证称为一个组，一个组最多可以有 100 个离线设备，每一个设备有一个编号，依次从 1 到 100。

必须依次为离线设备进行编号，例如第一台设备编号为 1 ，第二台设备为 2 等等。

基本使用步骤：

1. 使用 :term:`激活文件` 进行初始登记，设定许可证绑定的产品名称，初始登记完成之后会生成相应的 :term:`注册文件`
2. 在离线设备上生成相应的组设备文件
3. 在联网设备上面使用 :term:`注册文件` 和组设备文件生成离线注册文件
4. 在离线设备上使用离线注册文件进行注册，离线注册文件只能在绑定的设备上使用

初始登记
--------

购买集团版许可证之后，一个 :file:`pyarmor-regcode-xxxx.txt` 的 :term:`激活文件` 会发送到注册邮箱中，这个文件用于第一次的注册登记。

集团版许可证的初始登记也需要在有网络的设备上进行，初始登记必须指定许可证绑定的产品名称，这里不允许使用 ``TBD`` 。假设产品名称是 ``XXX`` ，运行下面的命令进行初始登记::

    $ pyarmor reg -p XXX pyarmor-regcode-xxxx.txt

登记成功之后会生成相应的 :term:`注册文件` ``pyarmor-regfile-xxxx.zip`` ，这个注册文件可用于后续的注册命令。

生成组设备文件
--------------

为了在离线设备上注册 Pyarmor，首先需要生成设备对应的组设备文件 ``pyarmor-group-device.X`` ，其中 ``X`` 是设备编号。

在每一台离线设备上安装 Pyarmor ，使用选项 :option:`-g` 指定设备编号，生成相应的设备文件。

例如，在第一台离线设备上，运行下面的命令生成组设备文件 ``pyarmor-group-device.1``::

    $ pyarmor reg -g 1

生成离线设备注册文件
--------------------

生成离线设备注册文件需要

- 安装 Pyarmor 8.2+ 并且可以访问互联网的设备
- 完成初始登记，并且已经生成 :term:`注册文件` ``pyarmor-regfile-xxxx.zip``
- 离线设备的组设备文件

把离线设备的组设备文件拷贝到进行初始登记的机器上（或者任何有互联网连接的设备），并存放在当前目录下面的子目录 ``.pyarmor/group/`` ，然后使用下面的命令生成离线注册文件 ``pyarmor-device-regfile-xxxx.1.zip``::

    $ mkdir -p .pyarmor/group
    $ cp pyarmor-group-file.1 .pyarmor/group/

    $ pyarmor reg -g 1 /path/to/pyarmor-regfile-xxxx.zip

这条命令生成的离线注册文件只可以在第一台设备上使用。

离线设备注册
------------

一旦生成离线设备注册文件之后，就可以拷贝到离线设备上面进行注册。例如，在第一台离线设备上，运行下面的命令::

    $ pyarmor reg pyarmor-device-regfile-xxxx.1.zip

查看注册信息::

    $ pyarmor -v

注册成功之后所有的加密操作自动应用集团版许可证，注册和加密都不需要联网验证。

运行不受限制的 Docker 容器
--------------------------

.. versionadded:: 8.3

集团版许可证支持运行不受限制的 Docker 容器，每一个 Docker 容器都使用当前离线设备的注册文件。这些容器需要使用默认的 Bridge 网络接口，并且没有进行高度定制而导致 Pyarmor 无法识别。

为了运行不受限制的容器，Docker 主机需要

- 离线设备注册文件 ``pyarmor-device-regfile-xxxx.1.zip`` ，参考上面的方法生成
- 安装 Pyarmor 8.3.9+

下面是使用集团版许可证的实践

- Docker 主机, Ubuntu x86_64, Python 3.8
- Docker 容器, Ubuntu x86_64, Python 3.11

首先拷贝下面的文件到 Docker 主机:

- pyarmor-8.3.9.tar.gz
- pyarmor.cli.core-4.3.5-cp38-none-manylinux1_x86_64.whl
- pyarmor.cli.core-4.3.5-cp311-none-manylinux1_x86_64.whl
- pyarmor-device-regfile-6000.1.zip

然后在 Docker 主机上运行下面的命令::

    $ python3 --version
    Python 3.8.10

    $ pip install pyarmor.cli.core-4.3.5-cp38-none-manylinux1_x86_64.whl
    $ pip install pyarmor-8.3.9.tar.bgz

接着启动 ``pyarmor-auth`` 来侦听来自 Docker 容器的认证请求::

    $ pyarmor-auth pyarmor-device-regfile-6000.1.zip

    2023-06-24 09:43:14,939: work path: /root/.pyarmor/docker
    2023-06-24 09:43:14,940: register "pyarmor-device-regfile-6000.1.zip"
    2023-06-24 09:43:15,016: listen container auth request on 0.0.0.0:29092

不要关闭这个命令窗口，另外打开一个命令窗口运行容器。

运行 Linux 系统的容器时候需要使用额外参数 ``--add-host=host.docker.internal:host-gateway`` （运行 Windows 和 Darwin 容器不需要）::

    $ docker run -it --add-host=host.docker.internal:host-gateway python bash

    root@86b180b28a50:/# python --version
    Python 3.11.4
    root@86b180b28a50:/#

在 Docker 主机上打开第三个控制台拷贝文件到容器::

    $ docker cp pyarmor-8.3.9.tar.gz 86b180b28a50:/
    $ docker cp pyarmor.cli.core-4.3.5-cp311-none-manylinux1_x86_64.whl 86b180b28a50:/
    $ docker cp pyarmor-device-regfile-6000.1.zip 86b180b28a50:/

最后在 Docker 容器中，注册 Pyarmor 并进行加密脚本::

    root@86b180b28a50:/# pip install pyarmor.cli.core-4.3.5-cp311-none-manylinux1_x86_64.whl
    root@86b180b28a50:/# pip install pyarmor-8.3.9.tar.gz
    root@86b180b28a50:/# pyarmor reg pyarmor-device-regfile-6000.1.zip
    root@86b180b28a50:/# echo "print('hello world')" > foo.py
    root@86b180b28a50:/# pyarmor gen --enable-rft foo.py

当需要验证许可证的时候，Docker 容器会发送请求到主机。

使用集团版许可证在 CI 服务器
----------------------------

集团版许可证还不支持直接应用在 CI 服务器的默认 Runner，但是类似 Github 上面的 `self-host runner`__ 应该可以工作，具体使用请参考相应的 CI 服务器文档。

另外一种变通的方法是在一台离线设备或者云服务器（例如阿里云的ECS）上先运行 相应 docker 容器进行加密脚本，然后把加密后脚本保存到源代码库的另外一个分支。

CI 服务器可以从这个分支中读取加密脚本，然后和操作正常脚本一样进行下面的工作流。

__ https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners

支持多设备的离线注册文件
------------------------

.. versionadded:: 8.x
                  该功能尚未实现

如果需要为所有设备生成一个集成的离线注册文件，首先把所有组设备文件拷贝到指定目录，然后使用特殊组号 ``0`` 生成所有设备可用的离线注册文件 ``pyarmor-device-regfile-xxxx.zip``::

    $ cp pyarmor-group-file.1 pyarmor-group-file.2 pyarmor-group-file.3 .pyarmor/group/

    $ pyarmor reg -g 0 /path/to/pyarmor-regfile-xxxx.zip

然后拷贝到每一台离线设备，使用 :option:`-g` 指定设备编号进行注册::

    $ pyarmor reg -g 1 pyarmor-device-regfile-xxxx.zip

升级老版本许可证
================

升级老版本的许可证请参考 :ref:`upgrading old license`

.. include:: ../_common_definitions.txt
