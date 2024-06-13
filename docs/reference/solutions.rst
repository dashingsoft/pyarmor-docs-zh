==================
 常见问题解决方案
==================

.. program:: pyarmor gen

如果是加密过程发生的错误，请查看章节 :ref:`build device` 并根据相应的条件查找解决方案

如果是运行加密过程发生的错误，请查看章节 :ref:`target device` 根据相应的条件查找解决方案

.. _build device:

运行 Pyarmor 的设备
===================

启动失败
--------

1. 检查是否已经正确安装 `pyarmor.cli.core` 包

   搜索 pyarmor 的安装目录树中是否存在扩展模块 `pytransform3.pyd` 或者 `pytransform3.so`

   如果不存在，请参考安装文档进行正确安装

2. 检查扩展模块 `pytransform3` 是否适用于当前平台

   使用 Python 直接运行下面的命令:

   .. code-block:: bash

       $ python
       >>> from pyarmor.cli.core import Pytransform3
       >>> Pytransform3.version()
       1
       >>>

   如果不能正确返回，尝试使用下面的命令检查扩展模块是否适用于当前平台

   在 Linux 平台下面，使用 `ldd` 执行下面的命令

   .. code-block:: bash

       $ ldd /path/to/pytransform3.so

   在 MacOS 平台下面，使用 `codesign` 和 `otool` 执行下面的命令

   .. code-block:: bash

       $ codesign -v /path/to/pytransform3.so
       $ otool -L /path/to/pytransform3.so

   在 Windows 平台下面，使用 [cygcheck]_ 进行检查::

       C:\> cygcheck \path\to\pytransform3.pyd

   或者::

       C:\> dumpbin /dependents \path\to\pytransform3.pyd

   根据报错信息安装相应的包看能否解决问题

3. 如果上面的命令报错并且无法解决，查看 :doc:`environments` 是否支持当前平台

   对于 Linux 的变体系统，可以查看包 `pyarmor.cli.core.linux` ， `pyarmor.cli.core.alpine` ，以及 `pyarmor.cli.core.android` 等的内容，每一个包里面都有多个 `pytransform.so` ，例如::

       $ unzip ./pyarmor.cli.core.linux-6.5.3-cp310-none-any.whl

       Archive:  ./pyarmor.cli.core.linux-6.5.3-cp310-none-any.whl
         inflating: pyarmor/cli/core/linux/__init__.py
         inflating: pyarmor/cli/core/linux/aarch64/pyarmor_runtime.so
         inflating: pyarmor/cli/core/linux/aarch64/pytransform3.so
         inflating: pyarmor/cli/core/linux/armv7/pyarmor_runtime.so
         inflating: pyarmor/cli/core/linux/armv7/pytransform3.so
         inflating: pyarmor/cli/core/linux/loongarch64/pyarmor_runtime.so
         inflating: pyarmor/cli/core/linux/loongarch64/pytransform3.so
         inflating: pyarmor/cli/core/linux/mips32el/pyarmor_runtime.so
         inflating: pyarmor/cli/core/linux/mips32el/pytransform3.so
         ...

   依次使用 `ldd` 检查，如果某一个包中某一个目录下面的 `pytransfrom.so` 可以用，那么拷贝它们到包 `pyarmor.cli.core` 的安装目录，例如::

       $ cp pyarmor/cli/core/linux/loongarch64/*.so /path/to/pyarmor/cli/core
       $ pyarmor gen foo.py

   或者安装这个包，同时设置环境变量::

       $ pip install ./pyarmor.cli.core.linux-6.5.3-cp310-none-any.whl
       $ export PYARMOR_PLATFORM=linux.loongarch64
       $ pyarmor gen foo.py

注册失败
--------

如果使用的是 :term:`激活文件` （ `pyarmor-regcode-xxxx.txt` ），确保使用次数没有超过 3 次，正常情况下这个文件仅用于第一次激活注册，以后的所有注册都应该使用 :term:`注册文件` ``pyarmor-regfile-xxxx.zip``

**基础版或者专家版许可证**

1. 检查设备的日期时间设置是否正确
2. 是否一分钟之内运行超过 3 次的注册命令
3. 如果是在运行 Docker 容器的话，是否在一分钟之内运行超过 3 个的 Docker 容器
4. 是否在 24 小时之内在超过 100 台不同设备（Docker 容器）上面运行注册命令

如果以上限制都没有超过的话，

1. 尝试在浏览器访问 http://pyarmor.dashingsoft.com/api/auth2/ 或者直接使用 `curl` 等进行测试

   如果返回 "NO:missing parameters" ，这说明网络和 Pyarmor 服务器都是好着的

   否则请检查本机的网络配置

2. 检查 Python 解释器是否可以访问 Pyarmor 服务器。使用 Python 解释器执行下面的命令，如果无法返回期望结果，那么请检查本机防火墙配置或者公司网络配置

   .. code-block:: bash

      $ python
      >>> from urllib.request import urlopen
      >>> res = urlopen('http://pyarmor.dashingsoft.com/api/auth2/')
      >>> print(res.read())
      b'NO:missing parameter'

   注意如果当前设备安装有多个 Python 的话，这个 Python 解释器必须是用来执行 Pyarmor 的解释器，否则没有意义

3. 如果以上都正确返回，还是无法注册成功，请提交问题报告

   报告中需要包含注册失败的许可证编号，例如 pyarmor-vax-6058

**集团版许可证**

1. 当前设备的 Machine ID 是否每次启动发生变化

   - 如果使用的是 Pyarmor 8.5 之前的版本，首先升级 Pyarmor 到最新版本
   - 如果是在硬件信息重启之后会发生变化的虚拟机器，例如 Github Action 的默认 runner ，无法直接使用集团版许可证

2. 是否已经使用集团版许可证的 :term:`注册文件` （一般为 ``pyarmor-regfile-????.zip`` ）生成了当前设备的注册文件（一般为 ``pyarmor-device-regfile-????.N.zip`` ）

3. 设备注册文件中的 Machine ID 是否和当前设备匹配

4. 如果设备 Machine ID 每次启动都保持不变，并且使用了正确的设备注册文件，请提交问题报告，至少包含下列信息：

   - 设备的 Machine ID
   - 设备类型，是物理设备，还是虚拟机，还是云服务器等
   - 操作系统类型
   - 对于类似 Linux 系统，提供 `uname -a` 的输出

**使用集团版许可证运行多个 Docker 容器**

确保 Docker 主机 和 Docker 容器在同一个网络中，使用 `ifconfig` 命令检查网路配置。

加密失败
--------

**update license token failed**

请参考 `注册失败` 中的解决方案

**加密出现其他异常或者错误信息**

1. 加密一个简单脚本 foo.py ，看能否成功
2. 如果不能成功，那么参考上面 `启动失败` 的检查点，确保扩展模块 `pytransform3` 适用当前平台
3. 如果扩展模块没有问题，请尝试在忽略配置环境产生的影响

   例如，把当前目录下面的局部配置目录 `.pyarmor` 更名为 `.pyarmor.bak` ， 把全局配置目录 `~/.pyarmor/config` 更名为 `~/.pyarmor/config.bak`

4. 使用更少加密选项，看能否成功，确定导致问题的选项
5. 报告问题，使用调试选项 `-d` 生成 `pyarmor.report.bug` ，并基于此

   - 提交一个尽可能简单的可以重现的脚本
   - 加密选项使用最少的可以重现的选项，尽量不要提交不会导致崩溃的无关选项

打包失败
--------

1. 直接使用 PyInstaller 打包没有加密的脚本，确保打包可以成功
2. 查看 :doc:`../topic/repack`

.. _target device:

运行加密脚本的设备
==================

导入运行辅助库装载失败
----------------------

1. 检查 :ref:`运行辅助包` 是否存在

   在加密脚本目录树中，搜索扩展模块 `pyarmor_runtime.pyd` 或者 `pyarmor_runtime.so` ，确保其存在

   :ref:`运行辅助包` 是一个正常的 Python 包，可以把它当作第三方的包来使用，但是必须能够被加密脚本导入

2. 检查加密脚本的导入语句是否正确

   直接打开加密脚本，应该可以看到加密脚本的第一条语句是一个标准的 `from ... import` 语句，请按照 Python 本身的导入机制确保能导入 :ref:`运行辅助包`

   可以通过移动运行辅助包的位置或者修改导入语句，以及使用选项 :option:`-i` 或者 :option:`--prefix` 重现生成加密脚本来解决这个问题

**如果加密设备和目标设备不是同一台机器**

1. 确保运行加密脚本的 Python 版本和加密时候使用的 Python 大小版本（Major.Minor）一致

2. 如果两者架构不同，请确保加密的时候使用了正确的跨平台加密选项 :option:`--platform`

3. 检查扩展模块 `pyarmor_runtime` 是否匹配目标设备的 Python 版本和平台架构

   在 Linux 平台下面，使用 `ldd` 执行下面的命令

   .. code-block:: bash

       $ ldd /path/to/pyarmor_runtime.so

   在 MacOS 平台下面，使用 `codesign` 和 `otool` 执行下面的命令

   .. code-block:: bash

       $ codesign -v /path/to/pyarmor_runtime.so
       $ otool -L /path/to/pyarmor_runtime.so

   在 Windows 平台下面，使用 [cygcheck]_ 进行检查::

       C:\> cygcheck \path\to\pyarmor_runtime.pyd

   或者::

       C:\> dumpbin /dependents \path\to\pyarmor_runtime.pyd

   根据报错信息安装相应的包看能否解决问题。

如果无法解决问题，查看 :doc:`environments` 是否支持当前平台

另外对于跨平台加密，尝试使用不同的目标平台，例如目标平台是 Android 的话，如果 `--platform android.aarch64` 加密的脚本无法运行，可以尝试 `--platform linux.aarch64` 或者 `--platform alpine.aarch64` ，因为它们的区别主要在于使用不同的 LIBC 库

**unauthorized use of script**

1. 不要使用 :option:`--private` ， :option:`--restrict` ， :option:`--assert-call` ， :option:`--assert-import` 等约束选项

2. 使用 `pyarmor cfg assert.call:excludes "xxx"` 或者 `pyarmor cfg assert.import:excludes "xxx"` 排除出现问题的函数和模块

3. 发现导致出现问题的选项，并报告问题，至少包含以下内容

   - 加密时候使用的完整选项
   - 可以重现问题的简单脚本，不要使用第三方库，尤其是依赖很多，安装包很大的第三方库
   - 运行加密脚本的完整命令和完整的异常信息堆栈

运行加密脚本出现错误或者异常
----------------------------

1. 尝试不要使用 RFT/BCC/约束选项等进行加密，看是否可以运行，可以运行的话说明问题是 RFT/BCC/约束选项 造成的，请参考下面的相应分支继续检查

2. 如果使用最少的选项加密脚本依旧不可以运行，尝试加密一个简单脚本，看看是否可以正确运行

3. 简化脚本，尝试找到一个可以重现问题的最简单脚本，最好不需要使用第三方库，如果使用到第三方库，查看 :doc:`../how-to/third-party`

4. 提交问题报告，至少包含下列内容

   - 加密时候使用的完整选项
   - 可以重现问题的简单脚本，不要使用第三方库，尤其是依赖很多，安装包很大的第三方库
   - 运行加密脚本的完整命令和完整的异常信息堆栈

**使用了约束选项进行加密**

参考上面的 **unauthorized use of script**

**使用了 RFT 模式进行加密**

参阅 :ref:`using rftmode` 中的解决方案

**使用了 BCC 模式进行加密**

首先使用一个简单脚本验证环境配置正确，如果简单脚本不工作，那么是多数是配置不正确或者未知的环境变量的影响

然后参阅 :ref:`using bccmode` 中的解决方案

运行打包的加密脚本出现错误或者异常
----------------------------------

1. 不要打包，只是使用相同选项加密脚本，然后运行加密后的脚本，检查是否运行正常，如果不正常，那么参考上面加密脚本运行失败的解决方案

2. 不要加密脚本，直接使用 PyInstaller_ 打包没有加密的脚本，检查打包后的可执行文件是否运行正常，如果不正常，请参考 PyInstaller_ 文档解决问题

3. 如果以上都正常，尝试使用更少的加密选项，重复步骤 1 和 步骤 2，找到导致问题出现的选项，然后报告问题

特别的，如果打包设备和目标设备不是同一台机器，必须在目标设备上检查运行结果

例如，在步骤1的检查中，直接在加密设备上运行加密脚本是没有意义的，必须在目标设备上运行加密脚本进行检查

.. _platform issues:

平台相关问题
============

Cygwin 需要创建链接

Darwin Apple Silicon 需要正确签名

.. rubric:: 注释

.. [cygcheck] 如何快速获取 `cygcheck.exe`

   直接下载 https://pyarmor.dashingsoft.com/downloads/tools/cygcheck.zip 解压

   或者从官网下载

   - 打开 https://cygwin.com/mirrors.html
   - 选择一个网站，依次进入目录 `x86_64/release/cygwin/`
   - 下载最新的 cygwin-3.5.3-1.tar.xz
   - 解压 tar xJf cygwin-3.5.3-1.tar.xz
   - 提取其中的 cygcheck.exe

.. include:: ../_common_definitions.txt
