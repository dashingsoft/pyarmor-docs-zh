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

.. _using group license:

使用集团版许可证
================

.. versionadded:: 8.2

每一个集团版许可证称为一个组，一个组最多可以有 100 个离线设备，每一个设备有一个编号，依次从 1 到 100。

必须依次为离线设备进行编号，例如第一台设备编号为 1 ，第二台设备为 2 等等。

离线设备可以为物理机，虚拟机，云服务器等各种形式，只要其 设备ID 每次重新启动能保持一致即可。大多数的物理设备，云服务器以及使用相同磁盘映像的虚拟机（Qemu，VirtualBox，Vmware）可以使用集团版许可证。

一旦设备注册，将永久占用该设备号。如果已经注册的设备重装系统，或者更换硬件系统，那么需要为其重新分配新的设备号。

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

    INFO     Python 3.12.0
    INFO     Pyarmor 8.4.7 (trial), 000000, non-profits
    INFO     Platform darwin.x86_64
    INFO     generating device file ".pyarmor/group/pyarmor-group-device.1"
    INFO     current machine id is "mc92c9f22c732b482fb485aad31d789f1"
    INFO     device file has been generated successfully

日志中的这一行显示的就是这台设备的标识符::

    INFO     current machine id is "mc92c9f22c732b482fb485aad31d789f1"

检查离线设备是否可以使用集团版许可证，可以重新启动系统，然后再次运行该命令，比较一下设备的标识符是否发生改变::

    $ pyarmor reg -g 1

    ...
    INFO     current machine id is "mc92c9f22c732b482fb485aad31d789f1"
    ...

因为集团版许可证是绑定到设备的，所有只用当设备标识符没有改变，那么这台离线设备才可以使用集团版许可证，否则无法使用集团版许可证。

对于虚拟机，WSL（Windows Subsystem Linux）以及其他任何系统，如果其设备标识符重启之后发生变化，请参考相应的文档配置系统网络的 Mac 地址以及硬盘的设备序列号保持不变，然后在检查设备标识符是否能够保持不变。如果设备标识符始终发生变化，那么集团版许可证无法在该系统使用。

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

这条命令生成的离线注册文件只可以在第一台设备上使用，因为离线设备注册文件 ``pyarmor-device-regfile-xxxx.1.zip`` 包含设备标识符的信息。

离线设备注册
------------

一旦生成离线设备注册文件之后，就可以拷贝到离线设备上面进行注册。例如，在第一台离线设备上，运行下面的命令::

    $ pyarmor reg /path/to/pyarmor-device-regfile-xxxx.1.zip

    INFO     Python 3.12.0
    INFO     Pyarmor 8.4.7 (trial), 000000, non-profits
    INFO     Platform darwin.x86_64
    INFO     register "/path/to/pyarmor-device-regfile-xxxx.1.zip"
    INFO     machine id in group license: mc92c9f22c732b482fb485aad31d789f1
    INFO     got machine id: mc92c9f22c732b482fb485aad31d789f1
    INFO     this machine id matchs group license
    INFO     This license registration information:

    License Type    : pyarmor-group
    License No.     : pyarmor-vax-006000
    License To      : Tester
    License Product : btarmor

    BCC Mode        : Yes
    RFT Mode        : Yes

    Notes
    * Offline obfuscation

注意查看这两行日志::

    INFO     machine id in group license: mc92c9f22c732b482fb485aad31d789f1
    INFO     got machine id: mc92c9f22c732b482fb485aad31d789f1

其中第一行显示的是注册文件中的设备标识符，下面显示的当前设备的标识符，两者只有匹配才能注册成功，如果不匹配，那么需要重新为当前设备生成设备标识符文件和离线注册文件。

查看注册信息::

    $ pyarmor -v

注册成功之后所有的加密操作自动应用集团版许可证，注册和加密都不需要联网验证。

.. important::

   如果需要重新为当前设备生成注册文件，必须使用下一个设备编号，原来的设备编号已经被占用而无法使用。例如，应该使用 `pyarmor reg -g 2` ，而不能还依旧使用  `pyarmor reg -g 1`

运行不受限制的 Docker 容器
--------------------------

.. versionadded:: 8.3

集团版许可证支持运行不受限制的 Docker 容器，每一个 Docker 容器都使用当前离线设备的注册文件。这些容器需要使用默认的 Bridge 网络接口，并且没有进行高度定制而导致 Pyarmor 无法识别。

**基本使用方法**

1. 每一个 Docker 主机被当作一个离线设备，并且必须能够注册集团版许可证

2. 然后在 Docker 主机上启动一个认证服务，来接受 Docker 容器的认证请求

3. 在 Docker 容器中运行 Pyarmor 的时候，会发送认证请求到 Docker 主机，并验证返回的结果

4. Docker 主机和 Docker 容器必须在同一个网段，否则 Docker 主机无法接受到 Docker 容器的认证请求。

**Linux Docker Host**

下面是使用集团版许可证的实践

- Docker 主机, Ubuntu x86_64, Python 3.8
- Docker 容器, Ubuntu x86_64, Python 3.11

为了运行不受限制的容器，Docker 主机需要

- 离线设备注册文件 ``pyarmor-device-regfile-xxxx.1.zip`` ，参考上面的方法生成
- 安装 Pyarmor 8.4.1+

首先拷贝下面的文件到 Docker 主机:

- pyarmor-8.4.1.tar.gz
- pyarmor.cli.core-5.4.1-cp38-none-manylinux1_x86_64.whl
- pyarmor.cli.core-5.4.1-cp311-none-manylinux1_x86_64.whl
- pyarmor-device-regfile-6000.1.zip

然后在 Docker 主机上运行下面的命令::

    $ python3 --version
    Python 3.8.10

    $ pip install pyarmor.cli.core-5.4.1-cp38-none-manylinux1_x86_64.whl
    $ pip install pyarmor-8.4.1.tar.bgz

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

    $ docker cp pyarmor-8.4.1.tar.gz 86b180b28a50:/
    $ docker cp pyarmor.cli.core-5.4.1-cp311-none-manylinux1_x86_64.whl 86b180b28a50:/
    $ docker cp pyarmor-device-regfile-6000.1.zip 86b180b28a50:/

最后在 Docker 容器中，注册 Pyarmor 并进行加密脚本::

    root@86b180b28a50:/# pip install pyarmor.cli.core-5.4.1-cp311-none-manylinux1_x86_64.whl
    root@86b180b28a50:/# pip install pyarmor-8.4.1.tar.gz
    root@86b180b28a50:/# pyarmor reg pyarmor-device-regfile-6000.1.zip
    root@86b180b28a50:/# pyarmor -v

如果配置正确，应该显示许可证的相关信息。

下面进行简单的测试::

    root@86b180b28a50:/# echo "print('hello world')" > foo.py
    root@86b180b28a50:/# pyarmor gen --enable-rft foo.py

当需要验证许可证的时候，Docker 容器会发送请求到主机。如果 Docker 主机的 `pyarmor-auth` 对应的控制台没有收到任何请求，请检查 Docker 网络配置，确保主机和容器的 IPv4 地址在同一个网段。

**MacOS Docker Host**

如果 Docker 主机是 MacOS 的时候，和 Linux 主机稍微有一些不同。因为 Docker 容器是运行在 Linux 虚拟机中，而不是直接运行在 MacOS 上面。

一个解决方案是直接在 Linux VM 上运行 `pyarmor-auth` ，在这种情况下，必须把 Linux VM 作为一个离线设备进行注册，而不是把 MacOS 作为离线设备进行注册，然后使用下面的额外参数启动 Docker 容器::

    $ docker run --add-host=host.docker.internal:172.17.0.1 ...

在这种情况下，也许需要对 Linux VM 进行额外的配置以确保其设备标识符重启之后保持不变。

参考 `issue 1542`__ 了解更多信息。

__ https://github.com/dashingsoft/pyarmor/issues/1542

**Windows Docker Host**

对于 Windows Docker 主机，首先检查网络配置::

  C:> ipconfig

  Ethernet adapter vEthernet (WSL):

       Connection-specific DNS Suffix  . :
       Link-local IPv6 Address . . . . . : fe80::8984:457:2335:588e%28
       IPv4 Address. . . . . . . . . . . : 172.22.32.1
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . :

如果它有一个 IPv4 地址，例如 ``172.22.32.1`` 是和 Docker 容器在同一个网段，那么事情就简单了。只需要把 Windows 作为离线设备进行注册，然后在上面运行 `pyarmor-auth` ，并且使用下面的额外选项启动 Docker 容器::

    $ docker run --add-host=host.docker.internal:172.22.32.1 ...

总之， `pyarmor-auth` 侦听的 IPv4 地址必须和 Docker 容器的 IPv4 地址在同一个网段，只要能达到这个目的，怎么进行配置都可以。

如果 Windows 没有任何 IPv4 地址和 Docker 容器在同一个网段，那么另外一种解决方案就是把 WSL (Windows Subsystem Linux）作为离线设备进行注册，然后运行 `pyarmor-auth` 在 WSL 里面。在这种情况下，也需要对 WSL 进行额外的配置以确保其设备标识符保持不变，配置方式请参考 WSL 的文档。

支持多设备的离线注册文件
------------------------

.. versionadded:: 8.x
                  该功能尚未实现

如果需要为所有设备生成一个集成的离线注册文件，首先把所有组设备文件拷贝到指定目录，然后使用特殊组号 ``0`` 生成所有设备可用的离线注册文件 ``pyarmor-device-regfile-xxxx.zip``::

    $ cp pyarmor-group-file.1 pyarmor-group-file.2 pyarmor-group-file.3 .pyarmor/group/

    $ pyarmor reg -g 0 /path/to/pyarmor-regfile-xxxx.zip

然后拷贝到每一台离线设备，使用 :option:`-g` 指定设备编号进行注册::

    $ pyarmor reg -g 1 pyarmor-device-regfile-xxxx.zip

.. _using pyarmor in ci pipeline:

在 CI/CD 管线中使用 Pyarmor
===========================

.. versionchanged:: 9.0

   专家版许可证不可用于 CI/CD 集成构建管线。

对于Pyarmor 试用版，直接使用 `pip install pyarmor` 在 CI/CD 管线中即可。

从 Pyarmor 9.0 开始，专家版和集团版许可证不可以应用于 CI/CD 管线

基础版和专用的 CI 许可证可以使用下面的方式应用于 CI/CD 管线

首先要经过初始登记，得到 :term:`注册文件`

1. 使用注册文件 pyarmor-regfile-xxxx.zip 在联网设备使用下面的命令向服务器发出请求::

    pyarmor reg -C pyarmor-regfile-xxxx.zip

2. 请求成功之后，会生成 CI 注册文件 "pyarmor-ci-xxxx.zip"

3. 使用 CI 注册文件 "pyarmor-ci-xxxx.zip" 在 CI/CD 中注册 Pyarmor::

    pyarmor reg pyarmor-ci-xxxx.zip

4. 使用 CI 注册文件的注意事项

   - 请勿在 CI/CD 管线中申请 CI 注册文件
   - 请求 CI 注册文件的次数最多为 100 次，超过之后将无法申请新的 CI 注册文件
   - CI 注册文件的最长有效期为 1 年，过期之后需要重新申请
   - CI 注册文件可能在升级后的 Pyarmor 中失效，失效之后也需要重新申请

.. important::

   只允许在开发设备上安装和注册 Pyarmor，如果 Docker 镜像需要发送给客户，那么不允许在上面安装和注册 Pyarmor

..
  在 Docker 或者 CI 管线 中注册 Pyarmor 基础版/专家版许可证的基本方法同上，但是在一天之内运行的 Pyarmor 的 Docker 数量有限制，不能超过 100 个。如果需要在一天之内运行超过 100 个的 Docker 容器，请使用集团版许可证。并且同时运行的 Docker 容器或者 Runner 的数量不能超过 3 个，如果超过这个数量，最好每隔半分钟之后启动一个 Docker 容器或者 Runner，同时运行太多的 `pyarmor reg` 命令会导致许可证服务器返回 HTTP 500 的错误并导致注册失败。

同一台设备使用多个许可证
========================

使用默认选项的时候，注册文件都是保存在 Pyarmor 的 :term:`根目录` ，通常情况下，就是 :file:`~/.pyarmor` ，这也意味着

- 不同的 Python 的虚拟环境会共享注册文件
- 如果需要在同一台设备上使用不同的许可证，那么会存在问题

当需要在一台设备上注册多个 :term:`Pyarmor 许可证` 的时候，可以为每一个许可证设置一个 :term:`根目录` 。例如::

    $ pyarmor --home ~/.pyarmor1 reg pyarmor-regfile-2051.zip
    $ pyarmor --home ~/.pyarmor2 reg pyarmor-regfile-2052.zip

    $ pyarmor --home ~/.pyarmor1 gen project1/foo.py
    $ pyarmor --home ~/.pyarmor2 gen project2/foo.py

升级 Pyarmor 之后的许可证兼容性
===============================

一般情况下，升级 Pyarmor 之后不需要重新注册，原来的注册信息依旧有效。

但是下面列出的版本是例外

- **Pyarmor 8.0** 老版本许可证无法在 Pyarmor 8.0 之后的版本中直接使用

  - 部分老版本许可证可以免费升级到基础版许可证，具体升级步骤请参考 :ref:`upgrading old license`
  - 老版本许可证无法升级为专家版或者集团版许可证

- **Pyarmor 9.0**

  主要是对在 CI/CD 管线环境的使用 Pyarmor 许可证进行了大的变更

  - 基础版许可证

    - 如果没有在 CI/CD 管线中使用，可以直接升级到 Pyarmor 9，无需额外操作
    - 如果在 CI/CD 管线中使用，需要为管线生成专门的注册文件，请参考上文中的 :ref:`using pyarmor in ci pipeline`

  - 专家版许可证

    - 如果没有在 CI/CD 管线中使用，可以直接升级到 Pyarmor 9，无需额外操作
    - 如果需要使用在 CI/CD 管线中，那么两种方案

      - 按照原来的协议继续使用任意 Pyarmor 8.x 的版本
      - 升级到 Pyarmor 9，同时需要升级专家版许可证为 CI 许可证

        - CI 许可证是按照年度收费
        - 专家版许可证购买时间超过一年的，需要重新付费升级
        - 专家版许可证购买时间没有超过一年的，免费升级到一年期满
        - 需要升级请联系 pyarmor@163.com

  - 集团版许可证需要重新生成设备注册文件

    使用 Pyarmor 9.0 之前版本生成的设备注册文件在 Pyarmor 9.0+ 版本中无效

    - 在联网设备上面安装升级 Pyarmor 到 9.0+
    - 使用集团版注册文件为设备重新生成注册文件，方法和第一次生成设备注册文件是一样的。例如，为编号为 1 的离线设备重新生成设备注册文件 ``pyarmor-device-regfile-6000.1.zip``::

      pyarmor reg -g 1 /path/to/pyarmor-regfile-6000.zip

    - 使用新生成的设备注册文件替换原来的设备注册文件

.. _check device for group license:

适用集团版许可证的设备
======================

集团版许可证只能用于设备硬件信息保持不变的系统，使用下面的方法检查一个设备是否适用集团版许可证

* 在设备上安装 Pyarmor 8.4.0+ 的试用版本
* 运行下面的命令得到机器标识符::

    $ pyarmor reg -g 1
    ...
    INFO     current machine id is "mc92c9f22c732b482fb485aad31d789f1"
    INFO     device file has been generated successfully

* 重新启动设备，重复上面的命令查看机器标示符
* 如果每一次重启之后机器标识符都保持不变，那么该设备可以使用集团版许可证，否则无法使用基本版许可证

对于 Docker 容器来说，只需要按照上面的方法检查 Docker Host。如果 Docker Host 可以使用集团版许可证，那么就可以在其上运行不受限制的 Docker 容器，具体使用方法请参考 :doc:`how-to/register` 中的 ``运行不受限制的 Docker 容器``

**如果 Docker Host 的机器标识符每次重启都会发生变化，那么集团版许可证无法运行任何 Docker 容器**

大多数的物理设备，云服务器以及使用相同磁盘映像的虚拟机（qemu，virtualbox，vmware）可以使用集团版许可证，集团版许可证不可用于 CI/CD 管线中

.. _upgrading old license:

升级老版本许可证
================

不是所有的老版本的许可证都可以升级为新的许可证。

符合下列条件的老版本许可证可以免费升级到 Pyarmor 基础版许可证：

* 遵循新的 `Pyarmor 最终用户许可协议`_
* 原来的许可证编号是以 ``pyarmor-vax-`` 开头的
* 原来许可证的注册文件 ``pyarmor-regcode-xxxx.txt`` 存在且不能被使用超过 100 次
* 原来的许可证的购买日期在 2023年6月1日之前，原则上在 Pyarmor 8 发布之后依旧购买的老许可证不支持升级。

如果无法免费升级，请购买新的许可证。

老版本的许可证不支持升级到专家版和集团版。

免费升级到基础版
-----------------

首先找到原来的许可证激活文件 ``pyarmor-regcode-xxxx.txt``

然后安装 Pyarmor 8.2+

按照新的 `Pyarmor 最终用户许可协议`_ ，需要为每一个许可证指定产品名称。这也意味着，如果老的许可证是被用于多种产品的话，升级之后就只能用于其中的一个，其他产品还需要购买新的许可证。

假定使用许可证的产品名称是 ``XXX`` ，那么使用下面的命令进行升级::

    $ pyarmor reg -u -p "XXX" pyarmor-regcode-xxxx.txt

升级成功之后会生成新的 :term:`注册文件`

在其他设备直接使用新的 :term:`注册文件` 进行注册::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

运行下面的命令检查升级后的许可证::

    $ pyarmor -v

注册成功之后所有的加密操作自动应用当前许可证，每一次加密操作需要联网验证许可证。

.. include:: ../_common_definitions.txt
