.. _常见问题:

常见问题
========

使用 PyArmor 需要一些必要的技能和知识，在报告问题之前，请确保你的问题不属于下面
这些 :ref:`必要的知识和技能` 里面列出的问题。

.. _必要的知识和技能:

必要的知识和技能
----------------

命令行
~~~~~~

pyarmor 是一个命令行工具，你必须知道如何在终端运行命令。如果你对命令行不了解，也
不想学习，那么请使用网页版的图形界面包 `pyarmor-webui`_

如果在使用 pyarmor 的过程中出现参数错误，命令行格式不正确等，请使用选项 ``-h``
执行命令，这样会显示所有可用选项和格式，请根据提示输入正常的参数格式。例如::

  pyarmor obfuscate -h

关于命令行在不同平台的一些参考链接
https://docs.microsoft.com/zh-cn/windows/terminal/

Python
~~~~~~

你必须了解如何在命令行运行 Python
https://docs.python.org/zh-cn/3.8/tutorial/interpreter.html#using-the-python-interpreter

脚本的字符编码
~~~~~~~~~~~~~~

如果加密脚本的输出结果中出现乱码，你需要了解 Python 脚本的字符编码

https://docs.python.org/zh-cn/3.8/tutorial/interpreter.html#source-code-encoding

请使用正确的方式设置文件的字符编码，然后首先运行不加密的脚本，检查输出结果是否正
常，然后在重新加密并运行脚本。

Python如何导入模块和包
~~~~~~~~~~~~~~~~~~~~~~

加密脚本需要运行辅助包来运行，运行辅助包是一个普通的 Python 包，可以像普通的模块
一样被导入和使用。缺少运行辅助包，或者存放的位置不正确，都会导致加密脚本运行失败，
保持类似这样的异常::

   ModuleNotFoundError: No module named 'app.pytransform'

这并不是 PyArmor 本身的错误，而是使用方法不正确导致的问题。你必须了解 Python 是
如何导入模块，以及绝对导入和相对导入等概念，知道 ``sys.path`` 的作用和意义，这样
才可以快速解决问题。

https://docs.python.org/3.8/library/sys.html#sys.path

加密脚本是一个非常简单的 Python 脚本，第一行是一个 import 语句，第二行是一个函数
调用语句。对于任何模块无法找到的错误，例如::

    ImportError: No module named model.NukepediaDB

请确认对于的模块文件存在并且在正确的位置，能被 Python 模块导入机制找到。这里加密
脚本完全和一个正常脚本一样，请参考下面的链接或者直接通过搜索引擎等其他方式掌握这
些 Python 的基本概念

https://docs.python.org/zh-cn/3.8/reference/import.html#the-import-system


PyInstaller
~~~~~~~~~~~

如果你需要把加密脚本打包成为可执行文件，那你必须了解 `PyInstaller`_ ，并且能够使
用 `PyInstaller`_ 直接打包你的脚本。

https://pyinstaller.readthedocs.io/en/stable/usage.html

因为 pyarmor 的打包功能是完全调用 PyInstaller 生成可执行文件，然后把其他的相关脚
本替换成为加密脚本。也就是说，如果 `PyInstaller`_ 不能成功打包，或者打好的包无法
正常运行，pyarmor 也不可以。

常见问题的解决方案
------------------

在我处理的大部分的问题报告中，除了少数的 PyArmor 本身的缺陷外，很多的问题报告都
是不了解如何使用 pyarmor 。所以当遇到问题的时候，花一点时间去了解一下 pyarmor ，
可能会很快的解决这些问题，这样可以让我们都节省很多时间。

首先你应当懂得 pyarmor 的基本使用方法 :ref:`使用 PyArmor`

然后浏览一下 :ref:`了解加密脚本` ，特别是 :ref:`加密脚本和原脚本的区别` 。当加密
脚本运行结果和原来的脚本不一样的时候，确保不是在脚本中使用到这些已知的区别点造成的。

如果你的问题是如何在某一种特定场景下使用 PyArmor，请首先浏览一下 :ref:`高级用法`
的目录标题，里面有大量的各种不同应用场景。

下面是通用解决方案

* 升级 pyarmor 到最新的稳定版本，如果以前 pyarmo 工作正常，然后突然无法工作，那
  么参考文档彻底删除 pyarmor 以及当前目录中 pyarmor 生成的相关文件，然后重新安装
  pyarmor，从一个空白状态重新开始原来的操作。

* 对于加密过程出现的问题，除了查看控制台的输出，看看有没有什么路径错误，和一些有
  效的错误信息，还可以使用调试选项 ``-d`` 运行，显示执行堆栈和更多的调试信息。不
  要只看到最后一行的错误，要仔细检查每一步的日志输出，了解 pyarmor 都接受了那些
  选项，根据选项具体做了那些工作，大部分的问题通过看日志就可以知道问题所在并得到
  正确的解决方法。例如::

      pyarmor -d obfuscate --recurisve foo.py

* 对于运行加密脚本出现的问题，尝试打开 Python 的调试选项查看更多的错误信息。如果
  抛出的异常堆栈中包含有脚本名称和行号，查看脚本对应行附近的代码，确认其没有使用
  这些加密脚本改变的特性 :ref:`加密脚本和原脚本的区别` 。例如::

      python -d obf_foo.py


* 如果你是跨平台发布，或者在不同机器上运行，请确保你使用了正确的跨平台加密参数。
  因为加密脚本在不同的平台需要不同的动态库，并且要求目标机器的 Python 版本必须和
  加密时候使用的版本一致，直接拷贝到不同平台的机器可能无法正确运行。

* 如果你是使用 :ref:`pack` 命令进行打包，那么首先不要加密脚本，确保不加密脚本能
  正常打包。

* 如果你使用 :ref:`约束模式` 3 以上，首先使用默认的约束模式进行加密，如果问题在
  默认约束模式不存在，那么你可能需要修改脚本使之符合高级约束模式的要求。

* 如果你在加密一个复杂的脚本或者包，先尝试一个简单的脚本或者包，看看它们是否工作

* 如果对 pyarmor 的某些特性不了解，花费几分钟的时间，在一个空白的目录做一个简单
  的测试。

pyarmor 的每一个命令都有很多选项，为各种不同的情况服务。如果默认选项不能满足你的
需求，你需要了解更多的选项。最常用的方式是使用选项 ``-h`` 列出命令的所以选项和支
持的格式。例如，下面的命令列出 ``obfuscate`` 的所有选项::

    pyarmor obfuscate -h

通过选项的简单说明你会发现某一个选项可能适合你的情况，如果不能确定这个选项是否真
的满足需要，可以参考 :ref:`命令手册` 中的详细说明。

其实最简单的方式是花费一分钟的时间，去做一个简单的测试，就可以很好的了解各个选项
的具体作用。例如，选项 ``--bootstrap`` 是用来控制如何生成引导代码的，在一个空白
目录下面做下面一个测试就可以完全了解它的作用::

  cd /path/to/test
  mkdir case-1
  cd case-1
  echo "print('Hello')" > foo.py
  pyarmor obfuscate --bootstrap 2 foo.py
  ls dist/
  cat dist/foo.py

  cd /path/to/test
  mkdir case-2
  cd case-2
  echo "print('Hello')" > foo.py
  pyarmor obfuscate --bootstrap 3 foo.py
  ls dist/
  cat dist/foo.py

你也可以组合其他选项做一些类似的测试，这样你可以很快的了解怎么使用 pyarmor

.. note::

   有大量的问题报告和解决方法在 PyArmor 的问题库 `issues`_ 中，请首先搜索这里看
   看有没有相同的问题报告

.. _reporting an issue:

.. _问题报告和报告模版:

问题报告和模版
--------------

如果文档中没有找到解决方案，请按照模版点击 `issues`_ 提交问题报告，并提供必要的
信息

1. 加密或者打包使用的完整命令和完整的输出日志（必要）
2. 如果是发布到不同机器，请说明那些文件被拷贝到目标机器（可选）
3. 运行加密脚本的命令和完整的错误日志

命令的输出可以使用重定向命令写到文件，然后拷贝文件内容到这里，例如::

  pyarmor obfuscate foo.py >log.txt 2>&1

下面是一个报告示例，报告的标题::

  cannot import name 'pyarmor' from 'pytransform'

报告的内容（可以直接拷贝下面的内容到 github 上，然后根据自己的情况进行修改）::

  1. 在 MacOS 10.14 上面使用 pyarmor 加密脚本
  ```
  $ pyarmor obfuscate --exact main.py
  INFO     Create pyarmor home path: /Users/jondy/.pyarmor
  INFO     Create trial license file: /Users/jondy/.pyarmor/license.lic
  INFO     Generating public capsule ...
  INFO     PyArmor Trial Version 7.0.1
  INFO     Python 3.7.10
  INFO     Target platforms: Native
  INFO     Source path is "/Users/jondy/workspace/pyarmor-webui/test/__runner__/__src__"
  INFO     Entry scripts are ['main.py']
  INFO     Use cached capsule /Users/jondy/.pyarmor/.pyarmor_capsule.zip
  INFO     Search scripts mode: Exact
  INFO     Save obfuscated scripts to "dist"
  INFO     Read product key from capsule
  INFO     Obfuscate module mode is 2
  INFO     Obfuscate code mode is 1
  INFO     Wrap mode is 1
  INFO     Restrict mode is 1
  INFO     Advanced value is 0
  INFO     Super mode is False
  INFO     Super plus mode is not enabled
  INFO     Generating runtime files to dist/pytransform
  INFO     Extract pytransform.key
  INFO     Generate default license file
  INFO     Update capsule to add default license file
  INFO     Copying /Users/jondy/workspace/pyarmor-webui/venv/lib/python3.7/site-packages/pyarmor/platforms/darwin/x86_64/_pytransform.dylib
  INFO     Patch library dist/pytransform/_pytransform.dylib
  INFO     Patch library file OK
  INFO     Copying /Users/jondy/workspace/pyarmor-webui/venv/lib/python3.7/site-packages/pyarmor/pytransform.py
  INFO     Rename it to pytransform/__init__.py
  INFO     Generate runtime files OK
  INFO     Start obfuscating the scripts...
  INFO     	/Users/jondy/workspace/pyarmor-webui/test/__runner__/__src__/main.py -> dist/main.py
  INFO     Insert bootstrap code to entry script dist/main.py
  INFO     Obfuscate 1 scripts OK.
  ```
  2. 拷贝 `dist/` 下面的所有目录到目标机器 Ubuntu
  3. 在 Ubuntu 上面使用 Python 3.7 运行加密脚本出现下面的错误
  ```
  $ cd dist/
  $ python3 main.py
  Traceback (most recent call last):
  File "main.py", line 1, in <module>
    from pytransform import pyarmor
  ImportError: cannot import name 'pyarmor' from 'pytransform' (/home/jondy/dist/pytransform/__init__.py)
  ```

.. important::

   关于安全方面的问题请发送邮件到 `pyarmor@163.com` ，其它任何问题请提交到
   `issues`_ ，微信不回答任何技术问题。

   下来任何一种报告将被标记为 ``invalid`` 并直接关闭：

   * 没有按照模版或者缺少必要信息的问题
   * 在文档中已经清楚的描述了解决方案

Segment fault
-------------

下面的情况都可能会导致导致程序崩溃

* 使用调试版本的 Python 来运行加密脚本
* 使用 Python X.Y 加密脚本，但是却使用不同版本的 Python M.N 来运行加密脚本
* 在不同的平台架构上运行加密脚本，但是加密的时候没有指定正确的选项 ``--platform``

  - Docker， 是 Alpine Linux，它的标准名称是 `musl.x86_64` ，而不是 `linux.x86_64`
  - 32位 Windows 和 64位 Windows 是不同的平台
  - 在 64位 Windows 平台，32位 Python 和 64位 Python 也是不同的平台

* 任何尝试访问 ``co_code`` 或者其他加密代码对象的属性都可能导致崩溃，有些第三方
  的包会分析代码对象，从而导致问题
* 在没有加密的脚本导入使用约束模式 3 以上加密的脚本。即便是使用 ``obf-code=0``
  加密的脚本，也无法导入约束模式 3 以上的加密脚本
* 混合使用不同 ``--advanced`` 值加密的脚本
* 在 MacOS 下面，如果 Python 的安装位置不在通常的位置，也有可能导致加密脚本崩溃。
  这时候要使用 ``install_name_tool`` 修改 ``rpath`` 解决这类问题。

如果使用的是 PyArmor v5.5.0 ～ v6.6.0 之间的版本，有的机器可能因为不支持高级模式
而崩溃。一个快速的解决方案是禁用高级模式，直接修改 pyarmor 安装包路径下面的
``pytransform.py`` , 找到函数 ``_load_library`` ，把禁用高级模式的注释去掉，修改成为
下面的样子::

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

如果还是有问题，那么请报告 issues_


ERROR: Unsupport platform linux.xxx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
请参考 :ref:`支持的平台列表`


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
~~~~~~~~~~~~~~~~

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
  创建新的进程的时候，确保 :ref:`引导代码` 在新进程中也得到执行，否则新进程是无法
  使用加密脚本的，一般的解决方案就是把这些脚本也要设置成为主脚本。
* 如果是 `pytransform.py` 或者 `pytransform/__init__.py` 那么报这个错误，那么确
  认这个模块没有被加密，这个模块是不能被加密的
* 同时检查系统模块 `os` ， `ctypes` 确保它们也没有被加密，，不要尝试加密这些文件，
  使用选项 ``--exclude`` 把 Python 系统库所在的目录排除。

如何检查引导代码是否运行？ 一种简单的方法是在前面插入一条 print 语句，例如

.. code:: python

    print('Start to run bootstrap code')
    from pytransfrom import pyarmor_runtime
    pyarmor_runtime()

如果调试信息被打印出来，那么说明引导代码已经被运行，删除这条 print 语句，排除引
导代码的问题，查找其它可能原因。

解决这个问题的另外一种方案是 :ref:`使用超级模式加密脚本`

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

如果使用新版本重新加密了主脚本，但是运行辅助文件没有替换，也可能会出现这个异常，
可以在加密的时候使用选项 ``--no-cross-protection`` 禁用交叉保护功能，或者使用选
项 ``--runtime`` 指定相同的运行辅助文件。

更多信息参考 :ref:`对主脚本的特殊处理`


运行脚本时候提示: Invalid input packet
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
如果混合使用试用版和注册版可能会导致这个问题，请确保删除所以试用版本生成的加密脚本
/许可文件以及运行辅助文件。

如果加密脚本导入的运行辅助模块 `pytransform` 不是和加密脚本一起发布的，而是位于
其他位置的同名包，那么也会出现这个错误。例如，在 pyarmor 包所在的目录运行加密脚
本，这时候 pyarmor 包内部的文件 `pytransform.py` 就会被加密脚本错误的导入。

如果加密脚本是在不同的平台下进行加密，参考 :ref:`跨平台发布加密脚本` 里面的备注。

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
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

NameError: name '__armor_wrap__' is not defined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

不正确的使用 :ref:`约束模式` 可能导致出现这个问题，尝试使用约束模式 2 来加密脚本

如果是线程导致的这个问题，参考这里 :ref:`在约束模块中使用 threading 和 multiprocessing`

如果是在方法 `__del__` 中抛出这个异常，请参考下一个问题

Object method `__del__` raise NameError exception
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果加密脚本中的类方法 `__del__` 抛出下面的异常::

    NameError: name '__armor_enter__' is not defined
    NameError: name '__armor_wrap__' is not defined

请升级 pyarmor 到 v6.7.3+，并且重新加密脚本，主要是要重新生成运行辅助包。

如果加密不是使用的超级模式，但是 Python 的版本有大于等于 3.7，请使用超级模式重新加密脚本。
或者重构代码，不要加密方法 `__del__` 。例如

.. code:: python

    class MyData:

        ...

        def lambda_del(self):
           # Real code for method __del__
           ...

        __del__ = lambda_del

任何以 `lambda_` 开头的函数不会被 pyarmor 加密，所以上面的示例中，类方
法 `__del__` 是没有加密的函数 ``lambda_del`` 。

另外一种解决方案是重构代码，把 ``__del__`` 中使用的函数集中到一个脚本里面，然后
不要加密这个脚本。最简单的方法就是显示按照原来的方式加密全部脚本，然后把这个脚本
拷贝过去，覆盖加密后的同名脚本，也可以使用 ``--obf-code 0`` 来加密脚本，也可以达
到同样效果。

SystemError: module filename missing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
运行使用外部许可证的超级模式加密的脚本，如果当前目录下面没有找到 `license.lic`
，那么加密脚本会抛出这个异常。如果 `license.lic` 在其他目录，那么使用环境变量
`PYARMOR_LICNSE` 指向这个文件，例如::

    export PYARMOR_LICNSE=/path/to/license.lic

Android 保护机制导致的问题
~~~~~~~~~~~~~~~~~~~~~~~~~~
大部分的 Android 系统只允许从特定目录装载动态库，而运行加密脚本需要的辅助文件就
有一个动态库文件 `_pytransform.so` ，所以直接拷贝加密脚本到 Android 系统运行可能
会抛出这样的异常::

    dlopen failed: couldn't map "/storage/emulated/0/dist/_pytransform.so"
    segment 1: Operation not permitted

请参考 Android 开发文档，拷贝整个目录 `pytransform` 到允许装载动态库的目录，然后
设置 `PYTHONPATH` 或者其他任何方式让 Python 能够导入包 `pytransform` 即可。

libpython3.9.so.1.0: cannot open shared object file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
如果运行加密脚本的时候缺少Python系统文件，例如 `python39.dll`, `libpython3.9.so`
等, 请确认Python解释器是使用选项 `--enable-shared` 进行编译的，并搜索本机找到对
应的文件. 这是因为默认情况下，运行辅助需要的扩展模块 `pytransform` 是链接到
Python的动态库上的。

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

首先请仔细阅读命令手册中 :ref:`pack` 的所有相关内容。

然后确认脚本可以直接使用 PyInstaller 打包，并且打包好的可执行文件运行正常。例如::

    pyinstaller foo.py
    dist/foo/foo

如果提示模块无法找到，或者其它原因而无法正常运行，说明缺少必要的 PyInstaller 的
参数，请参考 https://pyinstaller.readthedocs.io/ 解决问题，确保直接使用
PyInstaller 打包能够正常运行。

其次确认加密后的脚本，没有打包可以正确运行。例如::

    pyarmor obfuscate foo.py
    python dist/foo.py

如果两者都正常，那么删除输出路径 `dist` 和 PyInstaller 的缓存路径
`build` ，然后使用选项 ``--debug`` 进行打包::

    pyarmor pack --debug foo.py

这样所有的中间文件都会保留下来，其中有打过补丁的 `foo-patched.spec` 可
以被 PyInstaller 直接调用来打包加密脚本，例如::

    pyinstaller -y --clean foo-patched.spec

检查这个文件，并调整里面的选项，确保最后打包好的文件能够正常工作。

也可以参考这里 :ref:`使用加密脚本直接替换PyInstaller生成的可执行文件` ，尝试使用
这种方式打包加密脚本。

NameError: name ‘__pyarmor__’ is not defined
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

检查错误堆栈的信息，发现抛出这个异常的脚本名称，这对于定位问题很有帮助:

* 如果是 `pytransform.py` 或者 `pytransform/__init__.py` 那么报这个错误，那么确
  认这个模块没有被加密，这个模块是不能被加密的
* 同时检查系统模块 `os` ， `ctypes` 确保它们也没有被加密，不要尝试加密这些文件，
  使用选项 ``-x`` 把系统库目录排除，具体示例参考命令 :ref:`pack`
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

关于许可证的问题
----------------

请参考 :ref:`关于许可证的常见问题`

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

能否先试用一下贵公司产品，测试能否满足我们的需求
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

pyarmor 本身就是可以试用，我理解你的意思是解除试用限制的试用。但是目前 pyarmor
的许可机制是支持离线（不连网）就可以正常加密的，所以不支持这种方式的试用。

pyarmor 加密后的脚本基本和原来的脚本是兼容的，除了这里列出的这些不同 :ref:`加密脚本和原脚本的区别`

大部分的包和 pyarmor 都是兼容的，个别的打个补丁也可以工作，只有那些需要访问 byte
code 等这种类型的包是无法使用 pyarmor 的。

对于不能完全肯定兼容的包，可以使用小脚本测试一下。

请问如何在客户环境中 不安装pyarmor，可以正确的获取到客户硬盘的序列号
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

参考 :ref:`运行时刻模块 pytransform`, 在加密脚本中使用 `get_hd_info` 获取相关硬
件信息。

另外还可以从下面的链接中下载相应平台的 `hdinfo` ，在目标平台运行这个可执行文件获
取硬件信息

https://github.com/dashingsoft/pyarmor-core#hdinfo

.. include:: _common_definitions.txt
