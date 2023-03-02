.. highlight:: console

==========
 Man Page
==========

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: top

Pyarmor is a powerful tool to obfuscate Python scripts with rich option set that
provides both high-level operations and full access to internals.

pyarmor
=======

.. program:: pyarmor

.. describe:: Syntax

    pyarmor [options] <command> ...

.. describe:: Options

-h, --help            show available command set then quit
-v, --version         show version information then quit
-q, --silent          suppress all normal output :option:`... <-q>`
-d, --debug           show more information in the console :option:`... <-d>`
--home PATH           set Pyarmor HOME path :option:`... <--home>`

These options can be used after :program:`pyarmor` but before command, here are
available commands:

================================  ====================================
:ref:`gen <pyarmor gen>`          Obfuscate scripts
:ref:`gen key <pyarmor gen key>`  Generate outer runtime key
:ref:`cfg <pyarmor cfg>`          Show and configure environments
:ref:`reg <pyarmor reg>`          Register Pyarmor
================================  ====================================

See :command:`pyarmor <command> -h` for more information on a specific command.

.. describe:: Description

.. option:: -q, --silent

            Suppress all normal output.

For example::

    pyarmor -q gen foo.py

.. option:: -d, --debug

            Show more information in the console

When something is wrong, print more debug informations in the console. For
example::

    pyarmor -d gen foo.py

.. option:: --home PATH[,GLOBAL[,LOCAL[,REG]]]

            Set Pyarmor :term:`Home Path`, :term:`Global Configuration Path`,
            :term:`Local Configuration Path` and :term:`Registration File Path`

The default paths

* :term:`Home Path` is :file:`~/.pyarmor`

* :term:`Global Configuration Path` is :file:`~/.pyarmor/config`, it's always
  relative to :term:`Home Path`

* :term:`Local Configuration Path` is :file:`.pyarmor`

* :term:`Registration File Path` is same as :term:`Home Path`

All of them could be changed by this option. For example, change home path to
:file:`~/.pyarmor2`::

    $ pyarmor --home ~/.pyarmor2 ...

Then

* :term:`Global Configuration Path` is :file:`~/.pyarmor2/config`
* :term:`Registration File Path` is :file:`~/.pyarmor2`
* :term:`Local Configuration Path` still is :file:`.pyarmor`

Another example, keep all others but change global path only::

    $ pyarmor --home ,config2 ...

This command sets :term:`Global Configuration Path` to :file:`~/.pyarmor/config2`

Another example, keep all others but change local path only::

    $ pyarmor --home ,,/var/myproject/ ...

This command sets :term:`Local Configuration Path` to :file:`/var/myproject`

Another example, set :term:`Registration File Path` to :file:`/opt/pyarmor/`::

    $ pyarmor --home ,,,/opt/pyarmor ...

It's useful when may use :command:`sudo` to run :command:`pyarmor`
occassionally. This makes sure the registration file could be found even switch
to another user.

When there are many Pyarmor Licenses registerred in one machine, set each
license to different :term:`Registration File Path`

There are 2 soltions

* one license one home
* same home, one license one path

For example, the first solution::

    $ pyarmor --home ~/.pyarmor1 reg pyarmor-regfile-2051.zip
    $ pyarmor --home ~/.pyarmor2 reg pyarmor-regfile-2052.zip

    $ pyarmor --home ~/.pyarmor1 gen project1/foo.py
    $ pyarmor --home ~/.pyarmor2 gen project2/foo.py

The second solution::

    $ pyarmor --home ,,,pyarmor1 reg pyarmor-regfile-2051.zip
    $ pyarmor --home ,,,pyarmor2 reg pyarmor-regfile-2052.zip
                     ,,,
    $ pyarmor --home ,,,pyarmor1 gen project1/foo.py
    $ pyarmor --home ,,,pyarmor2 gen project2/foo.py

Start pyarmor with clean configuration by setting :term:`Global Configuration
Path` and :term:`Local Configuration Path` to any non-exists path ``x``::

    $ pyarmor --home ,x,x, gen foo.py

.. seealso:: :envvar:`PYARMOR_HOME`

.. _pyarmor gen:

pyarmor gen
===========

Generate obfuscated scripts and all the required runtime files.

.. program:: pyarmor gen

.. describe:: Syntax

    pyarmor gen <options> <SCRIPT or PATH>

.. describe:: Options

-h, --help                      show option list and help information then quit
-O PATH, --output PATH          output path :option:`... <-O>`
-r, --recursive                 search scripts recursively :option:`... <-r>`

-e DATE, --expired DATE         expired date of obfuscated scripts :option:`... <-e>`
-b DEV, --bind-device DEV       bind obfuscated scripts to device :option:`... <-b>`
--period N                      check runtime key periodically :option:`... <--period>`
--outer                         enable outer runtime key :option:`... <--outer>`

--platform NAME                 cross platform :option:`... <--platform>`
-i                              store runtime files inside package path :option:`... <-i>`
--prefix PREFIX                 运行辅助包的前缀名称，主要用于同时加密多个包的情况

--obf-module <0,1>              指定模块加密模式，默认是 1
--obf-code <0,1>                指定代码加密模式，默认是 1
--no-wrap                       禁用包裹加密模式
--enable <jit,rft,bcc,themida>  启用不同的保护特征
--mix-str                       混淆字符串常量
--restrict                      禁止外部脚本导入加密模块
--assert-import                 确保导入的脚本是加密后的模块
--assert-call                   确保调用的模块函数是经过加密的函数

--pack BUNDLE                   使用加密后的脚本替换打包成为可执行文件里面的原来脚本

.. describe:: Description

.. option:: -O PATH, --output PATH

设置加密脚本的输出路径，默认值是 ``dist``

.. option:: -r, --recursive

递归搜索目录下面的 Python 脚本，默认值搜索当前目录下面的脚本。

.. option:: -i

            保存运行辅助文件到加密包的内部

.. option:: -e DATE, --expired DATE

设置加密脚本的有效期，支持的格式:

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

.. option:: --period N

周期性的检查许可证，单位为 小时。默认情况下是导入加密模块的时候会检查许可证，对
于一些服务器应用，这个选项可以人工设置检查周期。

支持的格式为数字+单位，单位支持时分秒，没有单位则默认为小时：

* 1
* 1s
* 1m
* 1h

需要注意的是即便是设置了这个值，也必须是在运行加密函数的时候才进行检查。也就是说，
如果一个无限循环没有调用任何加密函数，那么也不会检查许可证。变通的方式是在里面循
环体里面增加一个空函数调用。

.. option:: --outer

            启用外部密钥

加密脚本如果需要使用外部密钥，必须在加密的时候指定这个选项，否则外部密钥不起作用。

而一旦指定使用外部密钥，就必须使用 :ref:`pyarmor gen key` 生成外部密钥文件，并拷
贝到指定目录，否则加密脚本无法运行。

外部密钥文件的名称默认为 ``pyarmor.rkey`` ，可以通过配置文件修改默认值，例如，使
用前面有点的 ``.pyarmor.key``::

    $ pyarmor cfg builder outer_keyname=".pyarmor.key"

这样外部密钥文件使用 :command:`ls` 就无法看到。

需要注意的是一旦修改密钥文件之后，必须重新生成运行辅助包，原来的运行辅助包无法在
使用，因为运行辅助包只会查找生成的时候指定的密钥文件名称。

.. option:: --platform NAME

            用于跨平台加密脚本指定运行加密脚本的目标平台

            这个选项可以使用多次，也可以使用逗号把多个平台名称分开

平台名称必须是 Pyarmor 定义的名称

不是所有的平台都可以组合在一起发布的

跨平台加密需要安装包 :mod:`pyarmor.cli.runtime` ，只有包里面支持的平台才能使用

.. option:: --restrict

            主要应用于保护加密包，保护包里面的模块，只能在包内部使用，不能被外部模块调用

.. _pyarmor gen key:

pyarmor gen key
===============

生成外部运行许可证

.. program:: pyarmor gen key

.. describe:: Syntax

    pyarmor gen key <options>

.. describe:: Options

-h, --help                      show option list and help information then quit
-O PATH, --output PATH          output path :option:`... <-O>`

.. _pyarmor cfg:

pyarmor cfg
===========

Configure or show Pyarmor environments

.. program:: pyarmor cfg

.. describe:: Syntax

    pyarmor cfg <options> [OPT[=VALUE]] ...

.. describe:: Options

-h, --help           show this help message and exit
-p NAME              private settings for special module or package
-g, --global         do everything in global settings, otherwise local
                     settings
-r, --reset          reset option to default value
--encoding ENCODING  specify encoding to read configuration file

.. describe:: Description

Run this command without arguments to show all available options::

    $ pyarmor cfg

Show one exact option ``obf_module``::

    $ pyarmor cfg obf_module

Show all options which start with ``obf``::

    $ pyarmor cfg obf*

Set option to new value::

    $ pyarmor cfg obf_module=0

Reset option to default::

    $ pyarmor cfg -r obf_module

Change option ``excludes`` in the section ``finder`` by this form::

    $ pyarmor cfg finder:excludes=ast

If no prefix ``finder``, for example::

    $ pyarmor cfg excludes=ast

Not only option ``excludes`` in section ``finder``, but also in other sections
``assert.call``, ``mix.str`` etc. are changed.

.. option:: -p NAME
            private settings for special module or package

All the settings is only used for specified module `NAME`.

.. option:: -g, --global
            do everything in global settings

Without this option, all the changed settings are soted in :term:`Local
Configuration Path`, generally it's ``.pyarmor`` in the current path. By this
option, everything is stored in :term:`Global Configuration Path`, generally
it's ``~/.pyarmor/config``

.. option:: -r, --reset
            Reset option to default value

.. _pyarmor reg:

pyarmor reg
===========

register Pyarmor or upgrade Pyarmor license

.. program:: pyarmor cfg

.. describe:: Syntax

    pyarmor reg [OPTIONS] [FILENAME]

.. describe:: Options

-h, --help            show this help message and exit
-p NAME, --product NAME
                      license to this product
-u, --upgrade         upgrade Pyarmor license
-y, --confirm         register Pyarmor without asking for confirmation

.. describe:: Arguments

The ``FILENAME`` must be one of these forms:

* ``pyarmor-keycode-xxxx.txt`` got by purchasing Pyarmor license
* ``pyarmor-regfile-xxxx.zip`` got by initial registration with above file

.. describe:: Description

Before register Pyarmor, first purchase one :term:`pyarmor-basic` license or
:term:`pyarmor-pro` license as needed in `MyCommerce website
<https://order.mycommerce.com/product?vendorid=200089125&productid=301044051>`_

After payment is complete successfully, in a few hours an activation file like
:file:`pyarmor-keycode-xxxx.txt` will be sent to you by email.

This file is used to initial registration. At the first time to register
Pyarmor, :option:`-p` (product name) should be set. If not set, this Pyarmor
license is bind to "non-profits", and could not be used for commercial product.

For example, assume this license is used to protect your product ``Robot
Studio``, initial registration by this command::

    $ pyarmor reg -p "Robot Studio" /path/to/pyarmor-keycode-xxxx.txt

Pyarmor will show registration information and ask your confirmation. If
everything is fine, type :kbd:`yes` and :kbd:`Enter` to continue.

If initial registration is successful, it prints final license information in
the console. And a registration file named :file:`pyarmor-regfile-xxxx.zip` for
this license is generated in the current path at the sametime. This file is used
for next registration in other machines.

Once register successfully, product name can't be changed.

There is only one exception, if product name is set to ``TBD`` at the first
time, it can be changed once later.

Check the registration information::

    $ pyarmor -v

Show verbose information::

    $ pyarmor reg

.. option:: -y, --confirm

            In initial registration, without asking for confirmation

.. option:: -u, --upgrade

            Upgrade Pyarmor license from prior to 8.0

This option is mainly used for purchased license before Pyarmor 8.0

.. important::

   Once initial registration successfully, :file:`pyarmor-keycode-xxxx.txt` may
   not work again. Using registration file :file:`pyarmor-regfile-xxxx.zip` for
   next registration instead.

   PLEASE BACKUP registration file :file:`pyarmor-regfile-xxxx.zip` carefully,
   Pyarmor doesn't provide lost-found service

Registration in other devices
-----------------------------

Using registration file :file:`pyarmor-regfile-xxxx.zip` to register Pyarmor in
other machine.

Copy it to target device, then run this command::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

Upgrade to pyarmor-basic
------------------------

如果老版本许可证的注册文件的名称是像 pyarmor-keycode-xxxx.txt 这样的，可以免费升
级到 pyarmor-basic，其它的无法升级到新版本，需要购买新的许可证使用最新版本。

如果选择免费升级, 那么需要遵循新的许可协议。最明显的变化在于老版本的许可证升级之
后，不管是个人版还是企业版，都必须绑定到一个产品。如果原来没有指定产品名称的，那
么升级的时候需要指定产品名称。这一点主要是对个人版影响比较大，新的许可协议要求每
一个产品对应一个许可证。所以个人版升级之后，升级后的许可证只能用于指定的产品，其
它产品如需使用 Pyarmor 8.0+ 需要购买新的许可证。

升级需要原来的注册码文件，是购买之后发送到注册邮箱里面的一个附件。

下面的命令要在原来已经注册成功的机器上运行，首先升级到 Pyarmor 8.0 以上::

    $ pip install pyarmor

如果原来许可证没有产品名称，假定现在绑定的产品名称为 ``Robot Studio`` ，那么使用
下面的命令进行升级::

    $ pyarmor reg -p "Robot Studio" -u pyarmor-keycode-xxxx.txt

如果原来的许可证已经有产品名称，不要指定产品名称，直接使用下面的命令进行升级::

    $ pyarmor reg -u pyarmor-keycode-xxxx.txt

Upgrade to pyarmor-pro
------------------------

如果老版本许可证的注册文件的名称是像 pyarmor-keycode-xxxx.txt 这样的，可以付费升
级到 pyarmor-pro，其它的无法升级到新版本，需要购买新的许可证使用最新版本。


根据需要购买 Pyarmor 基础版或者专家版许可证，

支付成功之后会通过电子邮件收到许可证的激活码，在邮件的附件中，一般名称为
:file:`pyarmor-keycode-xxxx.txt`

第一次注册
~~~~~~~~~~

第一次注册 PyArmor 需要指定产品名称，根据 Pyarmor 许可协议，每一个许可证都绑定到
一个指定产品，假设产品名称为 ``Robot Studio``::

    pyarmor reg -p "Robot Studio" /path/to/pyarmor-keycode-xxxx.txt

查看注册信息::

    pyarmor -v

注册成功的同时会在当前目录下生成离线注册文件 :file:`pyarmor-regfile-xxxx.zip`

文本格式的激活码文件 :file:`pyarmor-keycode-xxxx.txt` 只能使用十次，超过之后将无法使用。

所以一旦第一次注册成功之后，就可以使用离线注册文件在其它机器上注册 Pyarmor，而不
要在使用激活码文件进行注册。

请妥善保管离线注册文件，以后的在其它机器上注册都要使用此文件。如果丢失，Pyarmor
不负责找回，由此造成该许可证无法使用的后果由用户自己承担。在此情况下，如果用户还
需要使用 Pyarmor 需要重新购买。

在其它设备注册
~~~~~~~~~~~~~~

离线注册文件可以在授权范围内的任何机器上使用，同样使用 :ref:`pyarmor reg` 命令进
行注册，但是不需要指定产品名称。例如::

      pyarmor reg pyarmor-regfile-xxxx.zip

对于同时使用一个许可证的设备数量目前是限制在 100 台，所谓同时使用就是在 24 小时
内使用 Pyarmor 进行过加密操作的不同设备的总数。可以安装在 1000 台设备上，只要不
同时使用的数目不超过限制就可以。

Environment Variables
=====================

The following environment variables only used in :term:`Build Machine` when
generating the obfuscated scripts, not in :term:`Target Device`.

.. envvar:: PYARMOR_HOME

            Same as :option:`pyarmor --home`

It mainly used in the shell scrits to change Pyarmor settings. If
:option:`pyarmor --home` is set, this environment var is ignored.

.. envvar:: PYARMOR_PLATFORM

            Set the right :term:`Platform` to run :command:`pyarmor`

It's mainly used in some platforms Pyarmor could not tell right but still works.

.. envvar:: PYARMOR_CC

            Specify C compiler for bccmode

.. envvar:: PYARMOR_CLI

            Only for compatible with old Pyarmor, ignore this if you don't use
            old command prior to Pyarmor 8.0

If you always use old Pyarmor command prior to Pyarmor 8.0, set it to ``7``, for
example::

    # In Linux
    export PYARMOR_CLI=7
    pyarmor -h

    # Or
    PYARMOR_CLI=7 pyarmor -h

    # In Windows
    set PYARMOR_CLI=7
    pyarmor -h

By default :command:`pyarmor` first check new cli, if the command line couldn't
be parsed by new cli, fallback to old cli.

It could force :command:`pyarmor` to use specify cli directly.

.. include:: ../_common_definitions.txt
