==========
 命令手册
==========

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

Pyarmor 提供了丰富的选项来满足不同应用程序加密脚本的需求。

pyarmor
=======

.. program:: pyarmor

.. describe:: 语法

    pyarmor [options] <command> ...

.. describe:: 选项

-h, --help            显示所有可用的子命令并退出
-v, --version         显示版本信息并退出
-q, --silent          在控制台不显示日志 :option:`... <-q>`
-d, --debug           打印更多的信息用于发现命令执行过程中问题 :option:`... <-d>`
--home PATH           设置 :term:`根目录` :option:`... <--home>`

这些选项可以在命令 :program:`pyarmor` 之后和下列子命令之前使用:

================================  ====================================
:ref:`gen <pyarmor gen>`          生成加密脚本以及运行辅助文件
:ref:`gen key <pyarmor gen key>`  生成外部密钥文件
:ref:`cfg <pyarmor cfg>`          显示和配置加密环境
:ref:`reg <pyarmor reg>`          激活和注册 Pyarmor
================================  ====================================

使用 :command:`pyarmor <command> -h` 来查看每一个子命令的功能和所有的选项

.. describe:: 描述

.. option:: -q, --silent

            不在控制台打印任何日志信息

例如::

    pyarmor -q gen foo.py

.. option:: -d, --debug

            在控制台打印更多的信息用于发现命令执行过程中问题

加密脚本出现问题，或者想更多的了解加密过程，可以使用该选项打开调试模式。例如::

    pyarmor -d gen foo.py

.. option:: --home PATH[,GLOBAL[,LOCAL[,REG]]]

            设置 :term:`根目录` ， :term:`全局配置` 目录 ， :term:`本地配置` 目录以及注册信息所在的目录
            这个选项主要用于一台设备上需要使用多个不同的 :term:`Pyarmor 许可证` ，可以为每一个许可证设定一个 :term:`根目录` ，

它们的默认值分别是

* :term:`根目录` 是 :file:`~/.pyarmor/`

* :term:`全局配置` 目录是 :file:`~/.pyarmor/config/`

* :term:`本地配置` 目录是 :file:`./.pyarmor/`

默认存放注册文件的目录是 :term:`根目录`

这些目录配置都可以通过本选项进行设置。例如修改根目录为 :file:`~/.pyarmor2/`::

    $ pyarmor --home ~/.pyarmor2 ...

这条命令同时修改

* :term:`全局配置` 目录是 :file:`~/.pyarmor2/config/`
* 注册文件所在目录是 :file:`~/.pyarmor2/`

下面的例子只修改 :term:`全局配置` 目录为 :file:`~/.pyarmor/config2/`::

    $ pyarmor --home ,config2 ...

下面的命令只修改 :term:`本地配置` 目录为 :file:`/var/myproject`

    $ pyarmor --home ,,/var/myproject/ ...

当需要在一台设备上注册多个 :term:`Pyarmor 许可证` 的时候，可以为每一个许可证设置一个 :term:`根目录` 。例如::

    $ pyarmor --home ~/.pyarmor1 reg pyarmor-regfile-2051.zip
    $ pyarmor --home ~/.pyarmor2 reg pyarmor-regfile-2052.zip

    $ pyarmor --home ~/.pyarmor1 gen project1/foo.py
    $ pyarmor --home ~/.pyarmor2 gen project2/foo.py

在实际使用过程中，我们可能修改了很多配置，有时候出现问题之后，需要临时测试一下使用默认配置选项是否正常，但是又不想修改当前的配置。这时候可以通过设置全局配置目录和本地配置目录到一个不存在的路径来实现。例如::

    $ pyarmor --home ,x,x, gen foo.py

.. seealso:: :envvar:`PYARMOR_HOME`

.. _pyarmor gen:

pyarmor gen
===========

生成加密脚本和所有需要的运行辅助文件。

.. program:: pyarmor gen

.. describe:: 语法

    pyarmor gen <options> <SCRIPT or PATH>

.. describe:: 选项

-h, --help                      显示选项列表并退出
-O PATH, --output PATH          设置输出目录 :option:`... <-O>`
-r, --recursive                 递归搜索目录中的脚本 :option:`... <-r>`
--exclude PATTERN               排除脚本或者子目录 :option:`... <--exclude>`

-e DATE, --expired DATE         设置脚本有效期 :option:`... <-e>`
-b DEV, --bind-device DEV       绑定脚本到设备 :option:`... <-b>`
--bind-data DATA                存储自定义数据到运行密钥 :option:`... <--bind-data>`
--period N                      周期性检查运行密钥 :option:`... <--period>`
--outer                         启用外部密钥文件 :option:`... <--outer>`

--platform NAME                 指定脚本运行的目标平台名称 :option:`... <--platform>`
-i                              保存运行辅助文件到加密包的内部 :option:`... <-i>`
--prefix PREFIX                 设置导入运行辅助包的前缀名称 :option:`... <--prefix>`

--obf-module <0,1>              指定模块加密模式，默认是 1 :option:`... <--obf-module>`
--obf-code <0,1,2>              指定代码加密模式，默认是 1 :option:`... <--obf-code>`
--no-wrap                       禁用包裹加密模式 :option:`... <--no-wrap>`
--enable <jit,rft,bcc,themida>  启用不同的保护特征 :option:`... <--enable>`
--mix-str                       混淆字符串常量 :option:`... <--mix-str>`
--private                       启用私有模块加密脚本 :option:`... <--private>`
--restrict                      启用约束模式加密包 :option:`... <--restrict>`
--assert-import                 确保导入的脚本是经过加密的 :option:`... <--assert-import>`
--assert-call                   确保调用的函数是经过加密的 :option:`... <--assert-call>`

--pack BUNDLE                   使用加密后的脚本替换打包成为可执行文件里面的原来脚本 :option:`... <--pack>`

.. describe:: 描述

该命令用来加密脚本和包，例如::

    pyarmor gen foo.py
    pyarmor gen foo.py goo.py koo.py
    pyarmor gen src/mypkg
    pyarmor gen src/pkg1 src/pkg2 libs/dbpkg
    pyarmor gen -r src/mypkg
    pyarmor gen -r main.py src/*.py libs/utils.py libs/dbpkg

所有在命令行列出的文件都会被作为 Python 脚本进行加密，即便它的扩展名不是 ``.py`` 。

所有命令行列出的目录都会被作为包来进行加密，在该目录下面的所有 ``.py`` 文件会被加密，如果包含子目录需要加密，使用选项 :option:`-r` 来递归处理所有子目录。

不要使用 ``pyarmor gen src/*`` 来加密一个包，这样会加密目录下面的任何文件。

从 8.2.2 开始，支持使用文件列出所有需要加密的脚本和包, 然后使用前缀 ``@`` 来引用选项文件，例如::

    pyarmor gen -r @filelist

文件 :file:`filelist` 列出了需要加密的两个脚本和两个包::

    src/foo.py
    src/utils.py
    libs/dbpkg
    libs/config

.. option:: -O PATH, --output PATH

设置加密脚本的输出路径，默认值是 ``dist``

.. option:: -r, --recursive

递归搜索目录下面的 Python 脚本，否则只搜索当前目录下面的脚本

.. option:: --exclude PATTERN

            排除脚本或者子目录，这个选项可以使用多次

匹配的规则和 Python 标准库 `fnmatch`__ 的方式是一样的。

排除一个指定的脚本::

    $ pyarmor gen --exclude "src/test.py" src

排除一个指定的目录::

    $ pyarmor gen -r --exclude "./test" .

排除在任何目录下面的脚本 ``test.py``::

    $ pyarmor gen -r --exclude "*/test.py" src

排除所有的 ``test`` 目录::

    $ pyarmor gen -r --exclude "*/test" src

__ https://docs.python.org/3.11/library/fnmatch.html

.. option:: -i

保存运行辅助文件到加密包的内部。例如，下面的命令把 :term:`运行辅助包` 存放到 ``dist/mypkg`` 目录下面::

    $ pyarmor gen -r -i mypkg

    $ ls dist/
    ...      mypkg/

    $ ls dist/mypkg/
    ...            pyarmor_runtime_000000/

没有这个选项，输出的目录结构是::

    $ ls dist/
    ...      mypkg/
    ...      pyarmor_runtime_000000/

这个选项同时会使用相对导入的方式导入运行辅助包，所以只能在加密包中使用，如果是单独加密脚本，不要使用这个选项。

.. option:: --prefix PREFIX

            设置导入运行辅助包的前缀

主要应用于同时加密多个包，但是 :term:`运行辅助包` 又被存放在一个包里面，使用该选项告诉其他包如何正确导入运行辅助包。例如::

    $ pyarmor gen --prefix mypkg src/mypkg mypkg1 mypkg2

这个命令用于同时加密三个包，但是运行辅助包会存放在 ``dist/mypkg`` 里面，那么在 ``dist/mypkg/__init.py`` 里面，使用的是下面方式导入运行辅助包:

.. code-block:: python

    from .pyarmor_runtime_000000 import __pyarmor__
    __pyarmor__(__name__, __file__, b'...')

而在 ``dist/mypkg1/__init.py`` 里面，使用的是下面方式导入运行辅助包:

.. code-block:: python

    from mypkg.pyarmor_runtime_000000 import __pyarmor__
    __pyarmor__(__name__, __file__, b'...')

.. option:: -e DATE, --expired DATE

            设置加密脚本的有效期

支持的格式:

* 数字，表示从现在开始的天数
* YYYY-MM-DD，直接指定有效期

如果前面有字符 "."，那么表示使用本地时间判断，否则使用网络时间

例如::

  pyarmor gen -e 30 foo.py
  pyarmor gen -e 2022-12-31 foo.py

判断是否过期会读取 NTP 服务器的时间，所以在没有联网的机器上无法运行。

而下面的格式则使用本地时间验证有效期，例如::

  pyarmor gen -e .30 foo.py
  pyarmor gen -e .2022-12-31 foo.py

.. option:: -b DEV, --bind-device DEV

绑定加密脚本到指定的机器，目前支持的硬件信息包括硬盘序列号，网卡 Mac 地址和 IPv4
地址。例如::

  pyarmor gen -b 128.16.4.10 foo.py
  pyarmor gen -b 52:38:6a:f2:c2:ff foo.py
  pyarmor gen -b HXS2000CN2A foo.py

也可以和有效期组合使用::

  pyarmor gen -e 30 -b 128.16.4.10 foo.py

如果需要同时指定一个设备的多个硬件信息，那么使用空格分开各项，例如::

  pyarmor gen -b "128.16.4.10 52:38:6a:f2:c2:ff HXS2000CN2A" foo.py

这个选项可以使用多次，以绑定加密脚本到不同的机器，例如::

  pyarmor gen -b 52:38:6a:f2:c2:ff -b 66:77:88:9a:cc:fa -b "f8:ff:c2:27:00:7f" foo.py

如果需要绑定其他网卡，那么使用尖括号把网址包含起来，例如::

  pyarmor gen -b "<2a:33:50:46:8f>" foo.py

也可以使用下面的格式绑定一台机器上全部或者部分网卡，例如::

  pyarmor gen -b "<2a:33:50:46:8f,f0:28:69:c0:24:3a>" foo.py

在 Linux 系统下，还可以指定的网络接口的名称，例如::

  pyarmor gen -b "eth1/fa:33:50:46:8f:3d" foo.py

如果需要绑定其他硬盘，那么需要指定硬盘的名称，例如::

    # 适用于 Windows，分别绑定第一个硬盘和第二个硬盘
    pyarmor gen -b "/0:FV994730S6LLF07AY" foo.py
    pyarmor gen -b "/1:KDX3298FS6P5AX380" foo.py

    # 适用于 Linux，绑定到硬盘设备名称 "/dev/vda2"
    pyarmor gen -b "/dev/vda2:KDX3298FS6P5AX380" foo.py

.. option:: --bind-data DATA

            DATA 可以是字符串或者 ``@FILENAME``

这个选项可以存储任何数据到 :term:`运行密钥` 中，但是有长度限制，一般不超过 4096 个字节。主要用于用户扩展验证运行密钥的方式，在加密脚本中读取运行密钥中存放的数据，使用自己的算法进行校验和检查。

如果传入的参数以 ``@`` 开头，那么读取后面的文件内容，否则直接把参数的内容存放到运行密钥中。

不管哪一种情况，存放到运行密钥中都是 Bytes 类型。

.. option:: --period N

周期性的检查运行设置文件，单位为 小时。默认情况下是导入加密模块的时候会检查运行
设置信息，对于一些服务器应用，这个选项可以人工设置检查周期。

支持的格式为数字+单位，单位支持时分秒，没有单位则默认为小时：

* 1
* 1s
* 1m
* 1h

下面的例子是等价的，都是每隔一个小时重新检查一下::

    $ pyarmor gen --period 1 foo.py
    $ pyarmor gen --period 3600s foo.py
    $ pyarmor gen --period 60m foo.py
    $ pyarmor gen --period 1h foo.py

.. note::

   需要注意的是即便是设置了这个值，也必须是在运行加密函数的时候才进行检查。也就
   是说，如果一个无限循环没有调用任何加密函数，那么不会触发周期性检查事件。

.. option:: --outer

            启用外部密钥

加密脚本如果需要使用外部密钥，必须在加密的时候指定这个选项，否则外部密钥不起作用。

而一旦指定使用外部密钥，就必须使用 :ref:`pyarmor gen key` 生成外部密钥文件，并拷
贝到指定目录，否则加密脚本无法运行。

外部密钥文件的名称默认为 ``pyarmor.rkey`` ，可以通过配置文件修改默认值，例如，使
用前面有点的 ``.pyarmor.key``::

    $ pyarmor cfg outer_keyname=".pyarmor.key"

这样外部密钥文件使用 :command:`ls` 就无法看到。

需要注意的是一旦修改密钥文件之后，必须重新生成运行辅助包，原来的运行辅助包无法在
使用，因为运行辅助包只会查找生成的时候指定的密钥文件名称。

.. option:: --platform NAME

            用于跨平台加密脚本指定运行加密脚本的目标平台

            这个选项可以使用多次，也可以使用逗号把多个平台名称分开

平台名称必须是 Pyarmor 定义的 :term:`运行平台` 名称

不是所有的平台都可以组合在一起发布的

跨平台加密需要安装包 :mod:`pyarmor.cli.runtime` ，只有包里面支持的平台才能使用

.. option:: --private

            启用私有模式来保护加密脚本

启用私有模式之后，函数的名称在执行框架中会被隐藏，并且加密后的脚本也不能被其他脚本导入。

选项 :option:`--restrict` 隐含启用私有模式。

.. option:: --restrict

            主要应用于保护加密包，保护包里面的模块，只能在包内部使用，不能被外部模块调用

            这个选项隐含启用 :option:`--private`

当约束模式启用之后，除了 ``__init__.py`` 输出的名称之外，其他模块都不能被外部脚本导入和使用。

例如，使用下面的命令加密一个约束包 ``dist/joker``::

    $ pyarmor gen -i --restrict joker
    $ ls dist/
    ...    joker/

然后在创建一个没有加密的脚本 ``foo.py`` 去导入加密包:

.. code-block:: python

    import joker
    print('import joker should be OK')
    from joker import queens
    print('import joker.queens should fail')

运行结果如下::

    $ cd dist
    $ python foo.py
    ... import joker should be OK
    ... RuntimeError: unauthorized use of script

约束模式一般只用于加密包（目录），如果使用该选项单独加密脚本（文件）之后，脚本无法被直接运行。

.. option:: --obf-module <0,1>

            加密模块对应 ``.pyc`` ，默认是 1

.. option:: --obf-code <0,1,2>

            加密模块中的每一个函数（二次加密），默认是 1

模式 ``2`` 是 Pyarmor 8.2 中新增加的，它能增加从 Bytecode 进行反编译的难度，同时会降低一点性能。它可以把部分表达式中的属性名称混淆，例如::

    obj.attr          ==> getattr(obj, 'xxxx')
    obj.attr = value  ==> setattr(obj, 'xxxx', value)

通常情况下，如果 RFT 模块可用的话，不需要使用这个选项。

.. option:: --no-wrap

            不使用包裹模式加密模块中函数

使用包裹模式加密函数是指在调用函数的时候解密函数，调用完成重新加密函数。禁用之后第一次调用函数的解密函数，但是调用完成之后不在加密，以后调用的时候也无需重新解密。禁用包裹模式，对于多次执行的函数可以提高性能。

如果 :option:`--obf-code` 是 ``0`` ，那么使用选项没有任何作用。

.. option:: --enable <jit,rft,bcc,themida>

            启用不同的加密模式

.. option:: --enable-jit

            使用 :term:`JIT` 来处理一些敏感数据以增强安全性

.. option:: --enable-rft

            启用 :term:`RFT 模式` :sup:`pro`

.. option:: --enable-bcc

            启用 :term:`BCC 模式` :sup:`pro`

.. option:: --enable-themida

            使用 `Themida`_ 来保护加密脚本，仅 Windows 平台可用

.. option:: --mix-str

            混淆脚本中字符串常量 :sup:`basic`

.. option:: --assert-call

            启用自动检查函数功能，确保加密函数没有被替换

.. option:: --assert-import

            启用自动检查模块功能，确保加密的模块没有被替换

.. option:: --pack BUNDLE

            使用加密脚本替换 BUNDLE 里面的 Python 脚本

参数中 ``BUNDLE`` 是使用 PyInstaller_ 生成的可执行文件。

Pyarmor 首先加密脚本，接着使用加密后的脚本替换 BUNDLE 里面的同名 Python 脚本，最后使用修改后的可执行文件覆盖原文件。

.. _pyarmor gen key:

pyarmor gen key
===============

生成 :term:`外部密钥` 文件

.. program:: pyarmor gen key

.. describe:: 语法

    pyarmor gen key <options>

.. describe:: 选项

-O PATH, --output PATH      输出路径
-e DATE, --expired DATE     设置有效期
--period N                  定时检查运行密钥
-b DEV, --bind-device DEV   绑定加密脚本到指定设备
--bind-data DATA            存储自定义数据到运行密钥

.. describe:: 描述

这个命令用来生成 :term:`外部密钥` 文件，它使用到这些选项基本是命令 :ref:`pyarmor gen` 的子集，其作用和含义也一样，请参考上面的说明。

外部密钥必须至少包含 ``-e`` 或者 ``-b`` 一个选项，没有任何约束和限制的外部密钥存放安全风险，如果你确定需要这样的外部密钥，可以指定一万年的有效期，并且使用本地时间。

通常情况下外部密钥文件保存在 ``dist/pyarmor.rkey`` 。例如::

    $ pyarmor gen key -e 30
    $ ls dist/pyarmor.rkey

使用下面的命令保存外部密钥到其他路径::

    $ pyarmor gen key -O dist/mykey2 -e 10
    $ ls dist/mykey2/pyarmor.rkey

外部密钥的默认文件名称是 ``pyarmor.rkey`` ，它不能在命令行进行修改。但是可以通过配置文件进行修改。例如，下面的命令把外部密钥文件名称修改为 ``sky.lic``::

    $ pyarmor cfg outer_keyname=sky.lic
    $ pyarmor gen key -e 30
    $ ls dist/sky.lic

外部密钥文件也可以存放在其他位置，在运行的时候会依次查找下面的路径:

- 运行辅助包的目录
- 环境变量 :envvar:`PYARMOR_RKEY` 指定的路径，路径名称中不能包含 ``..`` ，尾部不能是路径分隔符，一般用来指定一个绝对路径，例如 ``/var/data``
- 当前目录

如果在以上目录下找不到 ``pyarmor.rkey`` ，会查看是否存在文件

- 当前可执行文件的全路径名称 + ``.pyarmor.rkey`` ，例如 ``dist/foo/foo.exe.pyarmor.rkey``

如果不存在，那么报错缺失运行密钥。

.. _pyarmor cfg:

pyarmor cfg
===========

显示和配置 Pyarmor 加密环境

.. program:: pyarmor cfg

.. describe:: 语法

    pyarmor cfg <options> [OPT[=VALUE]] ...

.. describe:: 选项

-h, --help           显示选项和帮助信息然后退出
-p NAME              指定私有设置的模块名称
-g, --global         所有操作都是在 :term:`全局配置` 中进行
-r, --reset          恢复配置项的默认值
--encoding ENCODING  指定打开配置文件的编码

.. describe:: 描述

查看所有的可用配置项::

    $ pyarmor cfg

只查看一个选项 ``obf_module``::

    $ pyarmor cfg obf_module

查看所有以 ``obf`` 开头的选项::

    $ pyarmor cfg obf*

设置一个整数型配置项的值，可以使用下面的任意一种方式::

    $ pyarmor cfg obf_module 0
    $ pyarmor cfg obf_module=0
    $ pyarmor cfg obf_module =0
    $ pyarmor cfg obf_module = 0

设置布尔型配置项的值::

    $ pyarmor cfg wrap_mode 0
    $ pyarmor cfg wrap_mode=1

设置字符串配置项的值::

    $ pyarmor cfg outer_keyname "sky.lic"
    $ pyarmor cfg outer_keyname = "sky.lic"

增加一个词到列表型配置项目使用运算符 ``+`` 。例如::

    $ pyarmor cfg pyexts + ".pym"

    Current settings
        pyexts = .py .pyw .pym

删除一个词使用 ``-`` 。例如::

    $ pyarmor cfg pyexts - ".pym"

    Current settings
        pyexts = .py .pyw

增加一行到配置项使用 ``^`` ，例如::

    $ pyarmor cfg rft_excludes ^ "/win.*/"

    Current settings
        rft_excludes = super
            /win.*/

恢复配置项的默认值，可以使用下面的任意一种格式::

    $ pyarmor cfg rft_excludes ""
    $ pyarmor cfg rft_excludes=""
    $ pyarmor cfg -r rft_excludes

指定选项所在的组中使用前缀 ``group:`` 。例如 修改组 ``finder`` 中的配置项 ``excludes``::

    $ pyarmor cfg finder:excludes "ast"

如果没有组前缀 ``finder`` ，那么其他组中的配置项 ``excludes`` 也会被修改。

.. describe:: 配置组

组是多个配置项的集合，常用的组有

* finder: 如何搜索脚本
* builder: 如果加密脚本，大部分的选项是在这里
* runtime: 如何生成运行辅助包和运行密钥

另外还有几个不常用的组

* mix.str: 如何选择需要加密的字符串
* assert.call: 如何选择需要保护的函数
* assert.import: 如何选择需要保护的模块
* bcc: 如何转换 Python 函数为 :term:`C` 函数

.. option:: -p NAME

            指定 :term:`模块私有配置` 的模块名称

            使用这个选项来处理特殊的模块，这些特殊模块需要使用不同的加密配置

所有的配置操作都是在该模块中，不影响其他模块。

对于包里面的模块，需要使用全路径名称来指定，例如 ``pkgname.modname.submodule``

例如，这两个模块 ``joker/__init__.py`` 和 ``joker/card.py`` 不使用任何约束，包里面的其他模块都使用约束模式::

    $ pyarmor cfg -p joker.__init__ restrict_module = 0
    $ pyarmor cfg -p joker.card restrict_module = 0
    $ pyarmor gen -r --restrict joker

.. option:: -g, --global

            所有操作都是在 :term:`全局配置` 中进行

没有指定这个选项的话，所有的操作都是在 :term:`本地配置` 中，通常就是 ``./.pyarmor/`` 。 指定了这个选项，所有的操作都在 :term:`全局配置` 中，通常就是 ``~/.pyarmor/config/``

.. option:: -r, --reset

            恢复配置项的默认值

.. _pyarmor reg:

pyarmor reg
===========

激活，注册和升级 Pyarmor

.. program:: pyarmor reg

.. describe:: 语法

    pyarmor reg [OPTIONS] [FILENAME]

.. describe:: 选项

-h, --help            显示可用选项和帮助信息然后退出
-p NAME, --product NAME
                      指定许可证绑定的产品名称
-u, --upgrade         升级老版本的 Pyarmor 许可证
-g ID, --device ID    指定组内设备编号，仅用于集团版许可证注册

.. describe:: 参数

参数 ``FILENAME`` 必须是下面任意一种文件

* ``pyarmor-regcode-xxxx.txt`` 激活文件，一般在购买许可证之后会发送到注册邮箱
* ``pyarmor-regfile-xxxx.zip`` 注册文件，初始登记之后自动生成

.. describe:: 描述

检查当前设备的注册信息::

    $ pyarmor -v

**初始登记**

初始登记使用下面的命令，产品名称必须输入，使用实际产品名称替换 ``NAME`` ，用于非商业化产品的使用名称 ``non-profits``::

    $ pyarmor reg -p NAME pyarmor-regcode-xxxx.txt

初始登记完成之后会生成相应的 :term:`注册文件` ``pyarmor-regfile-xxxx.zip`` ，在其他设备以及后续的注册均使用这个文件，并且不需要输入产品名称::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

**升级许可证**

升级使用下面的命令，其中产品名称必须和原来的许可证绑定名称一致，如果不一致的话会被忽略::

    $ pyarmor reg -p NAME pyarmor-regcode-xxxx.txt

升级完成之后会生成相应的 :term:`注册文件` ``pyarmor-regfile-xxxx.zip`` ，在其他设备以及后续的注册均使用这个文件，并且不需要输入产品名称::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

**集团版许可证**

集团版许可证也需要在有网络的机器上进行初始登记，并生成相应的 :term:`注册文件` ``pyarmor-regfile-xxxx.zip``

一个集团版许可证最多使用 100 个离线设备，每一个设备都有一个编号，从 1 到 100。

为了在离线设备上注册 Pyarmor，需要为每一台设备分别生成相应的离线注册文件。

例如，为第一台设备生成离线注册文件，首先在第一台设备运行下面的命令，生成一个组内的设备文件 ``pyarmor-group-device.1``::

    $ pyarmor reg -g 1

然后把这个文件拷贝到进行初始登记的机器上面，并存放在指定目录 ``.pyarmor/group/`` 下面，再使用下面的命令生成离线注册文件 ``pyarmor-device-regfile-xxxx.1.zip``::

    $ mkdir -p .pyarmor/group
    $ cp pyarmor-group-device.1 .pyarmor/group/

    $ pyarmor reg -g 1 pyarmor-regfile-xxxx.zip

拷贝这个离线注册文件到第一台设备，运行下面的命令进行离线注册::

    $ pyarmor reg pyarmor-device-regfile-xxxx.1.zip

对于第二台，第三台等设备，也需要进行相应的操作，设备的序号必须依次增加。

.. option:: -p NAME, --product NAME

            使用 ``.txt`` 文件注册时候指定许可证绑定的产品名称

            使用 ``.zip`` 文件注册的时候不需要使用这个选项

            非商业化产品中使用请设置名称为 ``non-profits``

第一次注册或者升级的时候指定许可证绑定的产品名称

绑定的产品名称一旦设定之后就无法更改，除了一个特殊名称 ``TBD``

如果产品名称设置为 ``TBD`` ，那么在六个月之内可以在修改一次。这个主要用于产品还在开发阶段，尚未确定名称的时候使用，在产品正式销售之前需要修改为正确的产品名称。如果六个月之内还没有进行修改，那么产品名称会被自动设定为 ``non-profits`` ，并且不能在被修改。

只有基础版和专家版许可证的产品名称可以被设定为 ``TBD`` ，升级许可证，以及集团版许可证的必须指定实际使用的产品名称。

.. option:: -u, --upgrade

            升级老版本的许可证为 Pyarmor 8 的许可证

不是所有的老版本的许可证都可以升级到新版本，请参考 :doc:`../licenses` 里面的升级说明

.. option:: -g ID, --group ID

            指定离线设备的组内编号，仅适用于集团版许可证

            有效值为 1 到 100

环境变量
========

下列环境变量是在 :term:`开发机器` 上加密脚本的时候会使用到

.. envvar:: PYARMOR_HOME

            设置方法和选项 :option:`pyarmor --home` 相同

主要用来设置 :term:`根目录` ，如果使用了选项 :option:`pyarmor --home` ，这个环境变量会被忽略

.. envvar:: PYARMOR_PLATFORM

            设置 Pyarmor 的 :term:`运行平台` 名称

主要用来支持一些能够运行 Pyarmor 但是无法正确得到平台名称的系统

.. envvar:: PYARMOR_CC

            设置 :term:`BCC 模式` 使用的 :term:`C` 编译器

.. envvar:: PYARMOR_CLI

            仅当需要和 Pyarmor 7.x 兼容的时候使用

如果安装了 Pyarmor 8，但是又需要保持和 Pyarmor 7.x 的兼容性，例如不修改原来的构建环境的脚本等，那么需要输出设置环境变量为 ``7`` 。

例如，在 Linux 或者 Apple 下面::

    export PYARMOR_CLI=7
    pyarmor -h

    PYARMOR_CLI=7 pyarmor -h


在 Windows 下面，使用下面的命令::

    set PYARMOR_CLI=7
    pyarmor -h

另外一种选择是直接使用命令 `pyarmor-7` 来和 Pyarmor 7.x 保持兼容。

.. include:: ../_common_definitions.txt
