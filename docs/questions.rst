.. _常见问题:

常见问题
========

如果你的问题是如何在某一种特定场景下使用 PyArmor，请首先浏览一下 :ref:`高级用法`
的目录标题，里面有大量的各种不同应用场景。

当出现问题的时候，首先仔细检查输出日志，它对于解决问题和定位错误很有帮助，并且尝
试下面的一些方法看能否解决它。

对于运行 `pyarmor` 出现的问题:

* 查看控制台的输出，有没有什么路径错误，和一些有效的错误信息
* 使用调试选项 ``-d`` 运行，显示执行堆栈和更多的调试信息。例如::

      pyarmor -d obfuscate --recurisve foo.py

对于运行加密脚本出现的问题:

* 尝试打开 Python 的调试选项查看更多的错误信息。例如::

      python -d obf_foo.py

在任何情况下，打开 Python 的调试开关之后，会在当前目录创建一个日志文件
`pytransform.log` ，里面包含有帮助定位问题的更多信息。

.. note::

   有大量的问题报告和解决方法在 PyArmor 的问题库 `issues`_ 中，请首先搜索这里看
   看有没有相同的问题报告

Segment fault
-------------

下面的情况都可能会导致导致程序崩溃

* 使用调试版本的 Python 来运行加密脚本
* 使用 Python 2.6 加密脚本，但是却使用 Python 2.7 来运行加密脚本

如果使用的是 PyArmor v5.5.0 之后的版本，有的机器可能因为不支持高级模式
而崩溃。一个快速的解决方案是禁用高级模式，直接修改 pyarmor 安装包路径
下面的 `pytransform.py` , 找到函数 `_load_library` ，把禁用高级模式的
注释去掉，修改成为下面的样子::

    # Disable advanced mode if required
    m.set_option(5, c_char_p(1))


启动问题
--------

Could not find `_pytransform`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
通常情况下动态库 `_pytransform` 在 :ref:`运行辅助包` 里面（在 v5.7.0
之前，是和加密脚本在相同的目录下）:

* `_pytransform.so` in Linux
* `_pytransform.dll` in Windows
* `_pytransform.dylib` in MacOS

首先检查这个文件是否存在。如果文件存在:

* 检查文件权限是否正确。如果没有执行权限，在 Windows 系统会报错::

    [Error 5] Access is denied

* 检查 `ctypes` 是否可以直接装载 `_pytransform`::

    from pytransform import _load_library
    m = _load_library(path='/path/to/dist')

* 如果上面的语句执行失败，尝试在 :ref:`引导代码` 中设置运行时刻路径::

    from pytransform import pyarmor_runtime
    pyarmor_runtime('/path/to/dist')

如果还是有问题，那么请报告 issue_


ERROR: Unsupport platform linux.xxx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
有些平台 `pyarmor` 无法自动识别，如果在 `其他平台的动态库清单`_ 中有可
用的动态库。可以直接下载下来，保存到这个平台的搜索路径
``~/.pyarmor/platforms/SYSTEM/ARCH`` 下面。如果不能确定存放的路径，可
以使用命令 ``pyarmor -d download`` 查看，在开始的时候会显示 `pyarmor`
去那里查找动态库。

如果需要在上面没有列出的平台使用 PyArmor，请发送邮件到 jondy.zhao@gmail.com


/lib64/libc.so.6: version 'GLIBC_2.14' not found
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在一些没用 `GLIBC_2.14` 的机器上，会报这个错误。

解决方案是对动态库 `_pytransform.so` 打补丁。

首先查看依赖的版本信息::

    readelf -V /path/to/_pytransform.so
    ...

    Version needs section '.gnu.version_r' contains 2 entries:
     Addr: 0x00000000000056e8  Offset: 0x0056e8  Link: 4 (.dynstr)
      000000: Version: 1  File: libdl.so.2  Cnt: 1
      0x0010:   Name: GLIBC_2.2.5  Flags: none  Version: 7
      0x0020: Version: 1  File: libc.so.6  Cnt: 6
      0x0030:   Name: GLIBC_2.7  Flags: none  Version: 8
      0x0040:   Name: GLIBC_2.14  Flags: none Version: 6
      0x0050:   Name: GLIBC_2.4  Flags: none  Version: 5
      0x0060:   Name: GLIBC_2.3.4  Flags: none  Version: 4
      0x0070:   Name: GLIBC_2.2.5  Flags: none  Version: 3
      0x0080:   Name: GLIBC_2.3  Flags: none  Version: 2

然后把版本依赖项 `GLIBC_2.14` 替换为 `GLIBC_2.2.5`:

* 从 0x56e8+0x10=0x56f8 拷贝四个字节到 0x56e8+0x40=0x5728
* 从 0x56e8+0x18=0x5700 拷贝四个字节到 0x56e8+0x48=0x5730

下面是使用 `xxd` 进行打补丁的脚本命令::

    xxd -s 0x56f8 -l 4 _pytransform.so | sed "s/56f8/5728/" | xxd -r - _pytransform.so
    xxd -s 0x5700 -l 4 _pytransform.so | sed "s/5700/5730/" | xxd -r - _pytransform.so

.. note::

   从 v5.7.9 开始，在 linux/x86_64 平台（例如，CentOS6）已经不需要这个补丁就可以工作了。

   在交叉发布的时候，可以在使用下面的命令加密脚本来解决这个问题::

     pyarmor obfuscate --platform centos6.x86_64 foo.py

__snprintf_chk: symbol not found
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在一些 Docker 中运行 pyarmor 可能会抛出这个异常，因为这些 Docker 使用的是
musl-libc, 但是 pyarmor 的默认动态库使用的是 glibc, 这个函数 ``__snprintf_chk``
在 musl-libc 中并不存在。

这时候，需要下载对应的动态库，这是 X86_64 的下载链接

http://pyarmor.dashingsoft.com/downloads/latest/alpine/_pytransform.so

这是 ARM 的下载链接
http://pyarmor.dashingsoft.com/downloads/latest/alpine.arm/_pytransform.so

然后使用这个覆盖原来的动态库，原来的动态库的位置可以在抛出的异常中找到

加密脚本的问题
--------------

'GBK' codec can't decode byte 0xXX
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在源代码的第一行指定字符编码，例如::

    # -*- coding: utf-8 -*-

关于源文件字符编码，请参考 https://docs.python.org/2.7/tutorial/interpreter.html#source-code-encoding


Warning: code object xxxx isn't wrapped
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
这是因为函数中包含特殊情况的跳转指令而引起的，例如，某一个函数编译成为
byte code 之后如果有类似这样的一条指令 `JMP 255`

在加密之前，这条指令占用两个字节。而在加密之后，因为在函数头部插入了额
外的指令，这个跳转指令的操作数变成了 `267` 。在 Python3.6 之后，这条指
令需要 4 个字节::

    EXTEND 1
    JMP 11

遇到这种情况，会使用非包裹方式加密这个函数。当然，当前模块中的其他函数
还是被使用包裹模式正常加密的。

如果想要避免这个问题，可以尝试在函数里面增加一些冗余的语句，让跳转长度
不要在临界值即可。

后面的版本会考虑解决这个问题，因为一旦修改原来的指令长度，代码块所有的
相对跳转、绝对跳转指令都需要调整，情况比较复杂，所以遇到这种情况，暂时
忽略了。

.. note::

   在 v5.5.0 之前，遇到这种情况，这个函数不会被加密。

Code object could not be obufscated with advanced mode 2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这是因为函数中包含无法处理的跳转指令而引起的，在这种情况下，可以重构函数，只要函
数第一条语句不是带有跳转指令的语句就可以。 像赋值，函数调用，或者其他简单语句都
不会有跳转指令，而 `try`, `for`, `while`, `if`, `with` 等复合语句一般都会生成相
应的跳转指令。所以一般情况下，只要把函数的第一条语句调整为赋值语句，就可以解决这
个问题。如果重构无法实现的，可以使用下面的冗余语句作为函数的第一条语句::

  [None, None]

这样会在函数的最开始的地方插入几条冗余指令，而不会影响原来的业务逻辑。

Error: Try to run unauthorized function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

试图使用没有授权的功能。出现这个问题一般是当前目录下面存在 `license.lic` 或者
`pytransform.key` 而导致的认证问题，解决方案是一是删除这些不必要文件，或者升级到
PyArmor 5.4.5 以后的版本。


为什么插件不工作
~~~~~~~~~~~~~~

如果加密脚本的时候指定了插件，但是插件却没有像期望的那样工作。那么首先
要检查插件是否被正确注入到主脚本中。例如::

  # In linux
  export PYTHONDEBUG=y
  # In Windows
  set PYTHONDEBUG=y

  pyarmor obfuscate --exact --plugin check_ntp_time foo.py

这样会生成一个调试文件 ``foo.py.pyarmor-patched`` ，检查这个文件，确保
插件脚本被正确的插入到里面，并且能被调用。


运行加密脚本的问题
------------------

NameError: name '__pyarmor__' is not defined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
原因是 :ref:`引导代码` 没有被执行。

* 当使用模块 `subprocess` 或者 `multiprocessing` ， 调用 `Popen` 或者 `Process`
  创建新的进程的时候，确保 :ref:`引导代码` 在新进程中也得到执行。否则新进程是无法
  使用加密脚本的。
* 如果是 `pytransform.py` 或者 `pytransform/__init__.py` 那么报这个错误，那么确
  认这个模块没有被加密，这个模块是不能被加密的
* 同时检查系统模块 `os` ， `ctypes` 确保它们也没有被加密，，不要尝试加密这些文件


Marshal loads failed when running xxx.py
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
当出现这个问题，依次进行下面的检查

1. 检查运行加密脚本的 Python 的版本和加密脚本的 Python 版本是否一致

2. 使用 `python -d` 运行加密脚本，查看更多的错误信息

3. 确保生成许可使用的密钥箱和加密脚本使用的密钥箱是相同的（当运行
   PyArmor 的命令时，该命令使用的密钥箱的文件名称会显示在控制台）

4. 如果是在 ARM 平台上，并且加密脚本是跨平台进行加密，那么确认加密时候使用的是特
   征为 0 的动态库，参考 :ref:`使用不同特征的动态库`

_pytransform can not be loaded twice
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
如果 :ref:`引导代码` 被执行两次，就会报这个错误。通常的情况下，是因为
在加密模块中插入了 :ref:`引导代码` 。 因为引导代码在主脚本已经执行过，
所以导入这样的加密模块就出现了问题。

.. note::

   这个限制是从 PyArmor 5.1 引入的，并且在 PyArmor 5.3.5 中已经移除，
   之后的版本都没有这个限制。


Check restrict mode failed
~~~~~~~~~~~~~~~~~~~~~~~~~~
违反了加密脚本的使用约束，默认情况下，加密脚本是不能被进行任何修改的。
更多信息参考 :ref:`约束模式`

如果使用 `pack` 命令去打包加密后的脚本，也会出现这个错误提示。命令
`pack` 正确的使用方式是直接打包原始的脚本。


Protection Fault: unexpected xxx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
违反了加密脚本的使用约束，默认情况下，下列文件不能进行任何修改：

* pytransform.py
* _pytransform.so/.dll/.dylib

更多信息参考 :ref:`对主脚本的特殊处理`


运行脚本时候提示: Invalid input packet
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
如果加密脚本是在不同的平台下进行加密，参考 :ref:`跨平台发布加密脚本`
里面的备注。

在 v5.7.0 之前，检查当前目录下面是否有存在文件 `license.lic` 或者
`pytransform.key` ，如果存在的话，确保它们是加密脚本对应的运行时刻文件。

因为加密脚本会首先在当前目录查看是否存在 `license.lic` 和运行时刻文件
`pytransform.key`, 如果存在，那么使用当前目录下面的这些文件。

其次加密脚本会查看运行时刻模块 `pytransform.py` 所在的目录，看看有没有
`license.lic` 和 `pytransform.key`

如果当前目录下面存在任何一个文件，但是和加密脚本对应的运行时刻文件不匹
配，就会报这个错误。


The `license.lic` generated doesn't work
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
通常情况下是因为加密脚本使用的密钥箱和生成许可文件时候使用的密钥箱不一
样，例如在试用版本加密脚本，但是在正式版本下面生成许可文件。

通用的解决方法就是重新把加密脚本生成一下，然后在重新生成许可文件。


导入 OpenCV 失败： `NEON - NOT AVAILABLE`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
在一些 Raspberry Pi 平台上面，在加密脚本中导入 OpenCV 会报错::

    ************************************************** ****************
    * FATAL ERROR: *
    * This OpenCV build doesn't support current CPU / HW configuration *
    * *
    * Use OPENCV_DUMP_CONFIG = 1 environment variable for details *
    ************************************************** ****************

    Required baseline features:
    NEON - NOT AVAILABLE
    terminate called after throwing an instance of 'cv :: Exception'
      what (): OpenCV (3.4.6) /home/pi/opencv-python/opencv/modules/core/src/system.cpp:538: error:
    (-215: Assertion failed) Missing support for required CPU baseline features. Check OpenCV build
    configuration and required CPU / HW setup. in function 'initialize'

当前的解决方案使用选项 ``--platform=linux.armv7.0`` ，例如::

    pyarmor obfuscate --platform linux.armv7.0 foo.py
    pyarmor build --platform linux.armv7.0
    pyarmor runtime --platform linux.armv7.0

另一种解决方案是设置环境变量 `PYARMOR_PLATFORM=linux.armv7.0` ，例如::

    PYARMOR_PLATFORM=linux.armv7.0 pyarmor obfuscate foo.py
    PYARMOR_PLATFORM=linux.armv7.0 pyarmor build

    或者，

    export PYARMOR_PLATFORM=linux.armv7.0
    pyarmor obfuscate foo.py
    pyarmor build

如何定制错误消息
~~~~~~~~~~~~~~~~

当加密脚本检查许可失败的时候，例如超过使用期限，会抛出异常 “License is expired”。
那么，是否可以定制这个错误信息呢？

可以通过修改 PyArmor 包所在目录下面的脚本 `pytransform.py` 来实现。它里面定义了
一个函数 `pyarmor_runtime`

.. code:: python

    def pyarmor_runtime(path=None, suffix='', advanced=0):
        ...
        try:
            pyarmor_init(path, is_runtime=1, suffix=suffix, advanced=advanced)
            init_runtime()
        except Exception as e:
            if sys.flags.debug or hasattr(sys, '_catch_pyarmor'):
                raise
            sys.stderr.write("%s\n" % str(e))
            sys.exit(1)

按照自己的需要修改里面的异常处理语句即可。

但是这种方式对于超级模式加密的脚本并不起作用，对于超级模式加密的脚本，可以使用创
建一个启动脚本在其中导入加密的主脚本 ``foo.py`` 并捕获异常。例如

.. code:: python

   try:
       import foo
   except Exception as e:
       print('something is wrong')

但是这种方式的副作用就是不仅仅是许可失败的异常，正常脚本的异常也会被捕获到的。如
果只需要捕获许可失败的异常，可以使用 :ref:`runtime` 创建一个独立的运行辅助包，并
使用这个运行辅助包加密脚本::

    pyarmor runtime --advanced 2 -O dist
    pyarmor obfuscate --advanced 2 --runtime @dist foo.py

然后创建一个启动脚本 ``dist/foo_boot.py`` ，例如
.. code:: python

   try:
       import pytransform_bootstrap
   except Exception as e:
       print('something is wrong')
   else:
       import foo

导入的这个脚本 ``pytransform_bootstrap.py`` 是被 :ref:`runtime` 自动创建的，它是
由空脚本加密生成的，所以它抛出的异常就只有 pyarmor 引导过程的异常。


undefined symbol: PyUnicodeUCS4_AsUTF8String
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果 Python 解释器使用的 UCS2 ，那么当它运行超级模式加密的脚本，会抛出这个异常。
解决方式就是使用平台 ``centos6.x86_64`` 进行加密，例如::

    pyarmor obfuscate --advanced 2 --platform centos6.x86_64 foo.py


打包加密问题
------------

No module name pytransform
~~~~~~~~~~~~~~~~~~~~~~~~~~
在使用 `pyarmor pack` 打包的时候报这个错误:

* 确认命令行指定的脚本是没有加密的
* 使用选项 ``--clean`` 清除缓存的 `myscript.spec`::

    pyarmor pack --clean foo.py

打包好的可执行文件运行有问题
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

首先确认脚本可以直接使用 PyInstaller 打包，并且打包好的可执行文件运行正常。

其次确认加密后的脚本，没有打包可以正确运行。

如果两者都正常，那么删除输出路径 `dist` 和 PyInstaller 的缓存路径
`build` ，然后使用选项 ``--debug`` 进行打包::

    pyarmor pack --debug foo.py

这样所有的中间文件都会保留下来，其中有打过补丁的 `foo-patched.spec` 可
以被 PyInstaller 直接调用来打包加密脚本，例如::

    pyinstaller -y --clean foo-patched.spec

检查这个文件，并调整里面的选项，确保最后打包好的文件能够正常工作。

NameError: name ‘__pyarmor__’ is not defined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

检查错误堆栈的信息，发现抛出这个异常的脚本名称，这对于定位问题很有帮助:

* 如果是 `pytransform.py` 或者 `pytransform/__init__.py` 那么报这个错误，那么确
  认这个模块没有被加密，这个模块是不能被加密的
* 同时检查系统模块 `os` ， `ctypes` 确保它们也没有被加密，，不要尝试加密这些文件
* 尝试只拷贝需要打包的脚本到一个空目录，然后进行打包。
* 如果试用版能正常打包，但是注册后出现这个问题，试试 :ref:`完全卸载` ，然后重
  装 PyArmor 之后在打包

PyArmor 注册问题
----------------

购买的私有密钥箱没有起作用
~~~~~~~~~~~~~~~~~~~~~~~~~~

使用私有密钥箱加密的脚本，依然可以在试用版本生成的许可证下运行:

* 确认命令 `pyarmor register` 能显示正确的注册信息
* 确认当前登陆用户和注册 PyArmor 使用的是相同用户
* 确认环境变量 `PYARMOR_HOME` 没有被设置，或者被正确设置
* 把 PyArmor :ref:`完全卸载` ，然后在重新注册
* 重新启动系统

Could not query registration information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

注册 PyArmor 显示成功::

    ~ % pyarmor register pyarmor-regfile-1.zip
    INFO     PyArmor Version 6.5.2
    INFO     Start to register keyfile: pyarmor-regfile-1.zip
    INFO     Save registration data to: /Users/Jondy/.pyarmor
    INFO     Extracting license.lic
    INFO     Extracting .pyarmor_capsule.zip
    INFO     This keyfile has been registered successfully.

但是查询注册信息失败::

    ~ % pyarmor register
    INFO     PyArmor Version 6.5.2
    PyArmor Version 6.5.2
    Registration Code: pyarmor-vax-000383
    Because of internet exception, could not query registration information.

有可能是 DNS 解析问题，运行下面的命令确认域名 `api.dashingsoft.com` 能够解析，例
如，显示出正确的 Ip 地址::

    ~ % ping api.dashingsoft.com

    PING api.dashingsoft.com (119.23.58.77): 56 data bytes
    Request timeout for icmp_seq 0
    Request timeout for icmp_seq 1

如果主机无法解析，增加一行到文件 ``/etc/hosts``::

    119.23.58.77 pyarmor.dashingsoft.com


已知的问题
----------

交叉发布的脚本无法运行
~~~~~~~~~~~~~~~~~~~~~~
从 v5.6.0 到 v5.7.0 这几个版本，交叉发布功能有一个问题。在 Windows / Ubuntu /
MacOS 等上面使用跨平台加密方式加密的脚本，拷贝到下面的任一平台都不能正常运行::

  armv5, android.aarch64, ppc64le, ios.arm64, freebsd, alpine, alpine.arm, poky-i586


其他问题
--------

在 Linux 下面无法获取硬盘序列号
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
获取硬盘序列号需要超级用户权限，首先确认有相关权限。

其次检查一下目录 `/dev/disk/by-id` ，这里会列出已经挂载的硬盘的接口和
序列号。如果这里没有文件，那么是无法获取硬盘序列号信息的。在 Docker 环
境里面的话，确保运行 docker 的时候挂载硬盘设备。

目前支持的硬盘接口包括 IDE，SCSI 以及 NVME 固态硬盘，对于其他接口的尚
不支持。

如何在不联网的机器上下载其他平台的动态库
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我的机器是在不联网的情况，无法下载so库，请问还有其的加密方式嘛？比如说我们把so库
下载，配置到不联网环境下可以嘛？如果可以的话，如何配置so库呢？

请参考 :ref:`如何人工下载和配置动态库`

.. include:: _common_definitions.txt
