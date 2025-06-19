==============
 完整安装教程
==============

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

前提条件
========

Pyarmor_ 需要 Python 以及 C 库，缺少它们 Pyarmor 无法正常启动运行。

..
  在 Linux 平台，如有必要请安装 Python 运行动态库。例如，使用下面的命令安装 Python 3.10 的运行动态库::

      $ apt install libpython3.10

  在 Darwin 平台，确保文件 ``@rpath/lib/libpythonX.Y.dylib`` 存在，这里 ``X.Y`` 表示 Python 的版本。例如::

      @rpath/lib/libpython3.10.dylib

  ``@rpath`` 是下列路径之一:

  - @executable_path/..
  - @loader_path/..
  - /System/Library/Frameworks/Python.framework/Versions/3.10
  - /Library/Frameworks/Python.framework/Versions/3.10

  如果没有这个文件，请安装必要的包或者使用必要的选项重新编译 Python ，或者使用 `install_name_tool` 适配当前的 Python 环境，参阅 :doc:`../question` 中如何解决 Apple 下面的奔溃问题。

.. _install-pypi:

从 PyPI 直接安装
================

Pyarmor_ 发布在 PyPI_ 上面，最方便的方式就是使用命令 :command:`pip` 直接安装。

在 Linux 和 MacOS，直接打开命令终端并运行下面的命令::

    $ pip install pyarmor

在 Windows 环境下，需要使用 :kbd:`Win-r` 打开命令输入框，然后输入 :command:`cmd` 打开命令窗口，并运行下面的命令:

.. code-block:: doscon

    C:\> pip install pyarmor

安装完成之后，输入命令 :command:`pyarmor --version` 并回车。如果安装成功，会显示安装的 Pyarmor 的版本信息。

如果需要跨平台发布加密脚本，根据需要安装相应平台的辅助运行包::

    $ pip install pyarmor.cli.core.windows
    $ pip install pyarmor.cli.core.themida
    $ pip install pyarmor.cli.core.linux
    $ pip install pyarmor.cli.core.darwin
    $ pip install pyarmor.cli.core.freebsd
    $ pip install pyarmor.cli.core.android
    $ pip install pyarmor.cli.core.alpine

并不是所有的平台都支持 Pyarmor，所有支持的运行平台请查看 :doc:`../reference/environments`

.. note::

   如果无需使用老版本 Pyarmor 7 的功能，直接安装 :mod:`pyarmor.cli` 而不是安装 :mod:`pyarmor` 可以显著减少下载时间。例如::

       $ pip install pyarmor.cli

.. note::

   如果需要安装老版本的 Pyarmor，请在命令行指定相应的版本。例如::

       $ pip install pyarmor==8.5.12

   更多指定版本的方式请参考 `pip` 的文档

安装的命令
----------

* :program:`pyarmor` 是最重要的一个，所有的工作基本都由它来完成，详细使用方法请参考 :doc:`../reference/man`
* :program:`pyarmor-7` 是为了和老版本兼容的命令，它等价于 Pyarmor 7.x 的修正版本。
* :program:`pyarmor-auth` 是集团版许可证支持运行不受限制 Docker 容器的辅助命令。

使用 Python 解释器直接运行 Pyarmor
----------------------------------

:program:`pyarmor` 等价于下面的命令::

    $ python -m pyarmor.cli

从 Github 上面进行安装
======================

.. deprecated:: 8.2.9

也可以直接从 `Pyarmor Github`__ 安装 Pyarmor。下载库到本地，然后使用 pip 进行安装::

    $ git clone https://github.com/dashingsoft/pyarmor
    $ cd pyarmor
    $ pip install .

你可以直接下载一个库的压缩文件 `tar.gz`__ 或者 `zip`__ ，解压之后在使用 pip 进行安装。

.. note::

   从 8.2.9 开始不推荐使用这种方式进行安装，这种方式可能因为无法正常运行。

__ https://github.com/dashingsoft/pyarmor
__ https://github.com/dashingsoft/pyarmor/archive/master.tar.gz
__ https://github.com/dashingsoft/pyarmor/archive/master.zip

在离线设备上安装
================

所有的 Pyarmor 需要的包发布在 PyPI_ ，需要从上面下载必要的包，然后拷贝到离线设备。

首先安装 :mod:`pyarmor.cli.core`

其次安装 :mod:`pyarmor` 或者 :mod:`pyarmor.cli`

例如，在 64位 Linxu 平台下面为 Python 3.10 安装 Pyarmor::

    $ pip install pyarmor.cli.core-3.2.9-cp310-none-manylinux1_x86_64.whl
    $ pip install pyarmor-8.2.9.zip

在 Android 或者 FreeBSD 系统，因为在包 :mod:`pyarmor.cli.core` 没有预编译的 wheel ，所以需要下载额外的包 :mod:`pyarmor.cli.core.android` 或者 :mod:`pyarmor.cli.core.freebsd` 。例如，在 Android 系统，运行下面的命令离线安装 Pyarmor::

    $ pip install pyarmor.cli.core-3.2.9.zip
    $ pip install pyarmor.cli.core.android-3.2.9-cp310-none-any.whl
    $ pip install pyarmor-8.2.9.zip

在一些特殊架构 `ppc64le`, `mips32el`, `mips64el`, `riscv64`, `loongarch64` 上，也没有 Wheel 直接可用，需要安装 `pyarmor.cli.core.linux` (glibc) 或者 `pyarmor.cli.core.alpine` (musl)。例如::

    $ pip install pyarmor.cli.core-8.5.9.zip
    $ pip install pyarmor.cli.core.linux-6.5.2-cp310-none-any.whl
    $ pip install pyarmor.cli-8.5.9.zip

如果需要跨平台加密，还需要安装相应的包 `pyarmor.cli.core.NAME`

- :mod:`pyarmor.cli.core.freebsd`
- :mod:`pyarmor.cli.core.android`
- :mod:`pyarmor.cli.core.windows`
- :mod:`pyarmor.cli.core.themida`
- :mod:`pyarmor.cli.core.linux`
- :mod:`pyarmor.cli.core.alpine`
- :mod:`pyarmor.cli.core.darwin`

例如，需要使用 Themida 保护的运行包，就需要安装::

    $ pip install pyarmor.cli.themida-3.2.9-cp310-none-any.whl

在 Linux 平台加密运行在 Windows 平台的包，需要安装::

    $ pip install pyarmor.cli.windows-3.2.9-cp310-none-any.whl

如果不需要使用老版本的命令 `pyarmor-7` ，推荐安装 :mod:`pyarmor.cli` 而不是 :mod:`pyarmor` ，前者需要下载的文件显著小于后者。例如::

    $ pip install pyarmor.cli-8.2.9.zip

Termux 平台的额外补丁
=====================

在 Termux 平台，安装完成之后还需要额外对扩展模块打补丁。例如::

    $ patchelf --add-needed libpython3.11.so.0.1 /data/data/com.termux/files/usr/lib/python3.11/site-packages/pyarmor/cli/core/android/aarch64/pytransform3.so
    $ patchelf --add-needed libpython3.11.so.0.1 /data/data/com.termux/files/usr/lib/python3.11/site-packages/pyarmor/cli/core/android/aarch64/pyarmor_runtime.so

有时候可以还需要额外的配置。例如::

    $ patchelf --set-rpath /data/data/com.termux/files/usr/lib /path/to/{pytransform3,pyarmor_runtime}.so

否则，运行 `pyarmor` 会报错 `dlopen failed: cannot locate symbol "PyFloat_Type"`

在 Python 脚本中调用 Pyarmor
============================

首先创建一个任意的脚本，例如 :file:`tool.py`

.. code-block:: python

    from pyarmor.cli.__main__ import main_entry

    args = ['gen', '-O', 'dist', '--platform', 'linux.x86_64,windows.x86_64', 'foo.py']
    main(args)

然后运行这个脚本::

    $ python tool.py

上面的例子等价于执行下面的命令::

    $ pyarmor gen -O dist --platform linux.x86_64,windows.x86_64 foo.py

以上只是一个示例说明，具体使用的加密选项和参数可以通过各种方式进行传递。

完全卸载
========

使用下面的命令可以完全卸载 Pyarmor::

    $ pip uninstall pyarmor
    $ pip uninstall pyarmor.cli.core

    # 下面的包不一定都安装，根据安装情况卸载相应的包

    $ pip uninstall pyarmor.cli.runtime
    $ pip uninstall pyarmor.cli.core.windows
    $ pip uninstall pyarmor.cli.core.themida
    $ pip uninstall pyarmor.cli.core.linux
    $ pip uninstall pyarmor.cli.core.darwin
    $ pip uninstall pyarmor.cli.core.freebsd
    $ pip uninstall pyarmor.cli.core.android
    $ pip uninstall pyarmor.cli.core.alpine

    $ rm -rf ~/.pyarmor
    $ rm -rf ./.pyarmor

.. note::

   ``~`` 是当前登录用户的个人目录，可以通过查看环境变量 ``HOME`` 得到具体位置。使用不同用户登陆，路径 ``~`` 可能不一样。

.. include:: ../_common_definitions.txt
