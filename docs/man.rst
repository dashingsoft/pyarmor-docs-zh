.. _命令手册:


命令手册
========

PyArmor 是一个命令行工具，用来加密脚本，绑定加密脚本到固定机器或者设置加密脚本的有效期。

`pyarmor` 的语法格式::

    pyarmor <command> [options]

常用的命令包括::

    obfuscate    加密脚本
    licenses     为加密脚本生成新的许可文件
    pack         打包加密脚本
    hdinfo       获取硬件信息

和工程相关的命令::

    init         创建一个工程，用于管理需要加密的脚本
    config       修改工程配置信息
    build        加密工程里面的脚本

    info         显示工程信息
    check        检查工程配置信息是否正确

其他不常使用的命令::

    benchmark    测试加密脚本的性能
    register     生效注册文件
    download     查看和下载预编译的动态库

可以运行 `pyarmor <command> -h` 查看各个命令的详细使用方法。

.. _obfuscate:

obfuscate
---------

加密 Python 脚本。

**语法**::

    pyarmor obfuscate <options> SCRIPT...

.. _obfuscate 命令选项:

**选项**

-O, --output PATH           输出路径，默认是 `dist`
-r, --recursive             递归模式加密所有的脚本
--exclude PATH              在递归模式下排除某些目录，多个目录使用逗号分开，或者使用该选项多次
--exact                     只加密命令行中列出的脚本
--no-bootstrap              在主脚本中不要插入引导代码
--no-cross-protection       在主脚本中不要插入交叉保护代码
--plugin NAME               在加密之前，向主脚本中插入代码
--platform NAME             指定运行加密脚本的平台
--advanced                  使用高级模式加密脚本
--restrict <0,1,2,3,4>      设置约束模式
--package-runtime <0,1,2>   是否保存运行文件到一个单独的目录

**描述**

PyArmor 首先检查用户根目录下面是否存在 :file:`.pyarmor_capsule.zip` ，
如果不存在，那么创建一个新的。

接着搜索需要加密的脚本，共有三种搜索模式：

* 默认模式： 搜索和主脚本相同目录下面的所有 `.py` 文件
* 递归模式： 递归搜索和主脚本相同目录下面的所有 `.py` 文件
* 精准模式： 仅仅加密命令行中列出的脚本

PyArmor 会修改主脚本，插入交叉保护代码，然后把搜索到脚本全部加密，保存
到输出目录 `dist`

在为加密脚本生成默认的许可文件 :file:`license.lic` 以及所有其他的
:ref:`运行辅助文件` ，也到保存到输出目录 `dist`

最后插入 :ref:`引导代码` 到主脚本。特别的，如果主脚本的名称为 `__init__.py` 并且
选项 `package-runtime` 不等于 `2`，那么会使用包含“.”的相对导入方式::

    from .pytransform import pyarmor_runtime
    pyarmor_runtime()

如果命令行有多个脚本的话，除了第一个脚本，不会在其他脚本中插入引导代码和交叉保护
代码。

选项 `--plugin` 主要用于扩展加密脚本的授权方式，例如检查网络时间来校验有效期等，
这个选项指定的插件名称会在加密之前插入到主脚本中。插件对应的文件为当前目录下面
`名称.py` ，如果插件存放在其他路径，可以使用绝对路径指定插件名称，也可以设置环境
变量 `PYARMOR_PLUGIN` 为相应路径名称。

关于插件的使用实例，请参考 :ref:`使用插件扩展认证方式`

选项 `--platform` 用于指定加密脚本的运行平台，仅用于跨平台发布。因为加密脚本的运
行文件中包括平台相关的动态库，所以跨平台发布需要指定该选项。

选项 `--restrict` 用于指定加密脚本的约束模式，关于约束模式的详细说明，参考
:ref:`约束模式`

如果选项 `--package-runtime` 设置为 `0` ，那么生成的运行辅助文件和加密脚本存放在
相同的目录下面::
  
    pytransform.py
    _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
    pytransform.key
    license.lic

如果是非 `0` 值，所有运行时刻文件会作为包保存在一个单独的目录 `pytransform` 下面::

    pytransform/
        __init__.py
        _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
        pytransform.key
        license.lic

`1` 表示运行时刻这个包会和加密脚本在相同目录， `2` 则表示运行时刻这个包会在其他
路径。

**示例**

* 加密当前目录下的所有 `.py` 脚本，保存到 `dist` 目录::

     pyarmor obfuscate foo.py

* 递归加密当前目录下面的所有 `.py` 脚本，保存到 `dist` 目录::

     pyarmor obfuscate --recursive foo.py

* 除了 `build` 和 `dist` 之外，递归加密当前目录下面的所有 `.py` 脚本，
  保存到 `dist` 目录::

     pyarmor obfuscate --recursive --exclude build,dist foo.py
     pyarmor obfuscate --recursive --exclude build --exclude tests foo.py

* 仅仅加密两个脚本 `foo.py`, `moda.py`::

     pyarmor obfuscate --exact foo.py moda.py

* 加密包 `mypkg` 所在目录下面的所有 `.py` 文件::

     pyarmor obfuscate --output dist/mypkg mypkg/__init__.py

* 加密当前目录下面所有的 `.py` 文件，但是不要插入交叉保护代码到主脚本
  :file:`dist/foo.py`::

     pyarmor obfuscate --no-cross-protection foo.py

* 加密当前目录下面所有的 `.py` 文件，但是不要插入引导代码到主脚本
  :file:`dist/foo.py`::

     pyarmor obfuscate --no-bootstrap foo.py

* 在加密 `foo.py` 之前，把当前目录下面的 `check_ntp_time.py` 的内容
  插入到 `foo.py` 中::

     pyarmor obfuscate --plugin check_ntp_time foo.py

* 在 MacOS 平台下加密脚本，这些加密脚本将在 Ubuntu 下面运行，使用下面
  的命令进行加密::

    pyarmor download --list
    pyarmor download linux_x86_64

    pyarmor obfuscate --platform linux_x86_64 foo.py

* 使用高级模式加密脚本::

    pyarmor obfuscate --advanced foo.py

* 使用约束模式 2 加密脚本::

    pyarmor obfuscate --restrict 2 foo.py

* 使用约束模式 4 加密当前目录下面除了 `__init__.py` 之外的所有 `.py` 文件::

    pyarmor obfuscate --restrict 4 --exclude __init__.py --recursive .

* 加密一个模块，并且把运行时刻文件保存为一个单独的包::

    cd /path/to/mypkg
    pyarmor obfuscate -r --package-runtime 2 --output dist/mypkg __init__.py

.. _licenses:

licenses
--------

为加密脚本生成新的许可文件

**语法**::

    pyarmor licenses <options> CODE

.. _licenses 命令选项:

**选项**

-O OUTPUT, --output OUTPUT            输出路径
-e YYYY-MM-DD, --expired YYYY-MM-DD   加密脚本的有效期
-d SN, --bind-disk SN                 绑定加密脚本到硬盘序列号
-4 IPV4, --bind-ipv4 IPV4             绑定加密脚本到指定IP地址
-m MACADDR, --bind-mac MACADDR        绑定加密脚本到网卡的Mac地址
-x, --bind-data DATA                  用于扩展认证类型的时候传递认证数据信息

**描述**

运行加密脚本必须有一个认证文件 :file:`license.lic` 。一般在加密脚本的
同时，会自动生成一个缺省的认证文件。但是这个缺省的认证文件允许加密脚本
运行在任何机器并且永不过期。如果你需要对加密脚本进行限制，那么需要使用
该命令生成新的许可文件，并覆盖原来的许可文件。

例如，下面的命令生成一个有使用期限的认证文件::

    pyarmor licenses --expired 2019-10-10 mycode

生成的新的认证文件保存在默认输出路径和注册码组合路径 `licenses/mycode`
下面，使用这个新的许可文件覆盖默认的许可文件::

    cp licenses/mycode/license.lic dist/
    
另外一个例子，限制加密脚本在固定 Mac 地址，同时设置使用期限::

    pyarmor licenses --expired 2019-10-10 --bind-mac 2a:33:50:46:8f tom
    cp licenses/tom/license.lic dist/

在这之前，一般需要运行命令 :ref:`hdinfo` 得到硬件的相关信息::

    pyarmor hdinfo

选项 `-x` 可以把任意字符串数据存放到许可文件里面，主要用于自定义认证类
型的时候，传递参数给自定义认证函数。例如::

    pyarmor licenses -x "2019-02-15" tom

然后在加密脚本中，可以从认证文件的信息中查询到传入的数据。例如::

    from pytransfrom import get_license_info
    info = get_license_info()
    print(info['DATA'])

.. note::

   这里有一个实际使用的例子 :ref:`使用插件扩展认证方式`

.. _pack:

pack
----

加密并打包脚本

**语法**::

    pyarmor pack <options> SCRIPT

.. _pack 命令选项:

**选项**

-t TYPE, --type TYPE            cx_Freeze, py2exe, py2app, PyInstaller(default).
-O OUTPUT, --output OUTPUT      输出路径
-e OPTIONS, --options OPTIONS   运行外部打包工具的额外参数
-x OPTIONS, --xoptions OPTIONS  加密脚本的额外参数
--clean                         打包之前删除上一次生成的中间文件
--without-license               不要将加密脚本的许可文件打包进去
--debug                         不要删除打包过程生成的中间文件

**描述**

PyArmor 首先调用第三方工具（例如，PyInstaller）对脚本打包，得到相关的
依赖文件。

接着加密主脚本所在路径下面所有 `.py` 文件，注意依赖的库中的 `.py` 文件不会被加密的。

然后使用加密脚本替换原来的脚本。

最后在把所有的文件打包到一起。

选项 `--options` 用来传递额外的参数给外部打包工具。例如，`PyInstaller` 是通过下面的方式调用的::

    pyinstaller --distpath DIST -y EXTRA_OPTIONS SCRIPT

其中 `EXTRA_OPTIONS` 会被该选项所替换。

选项 `--xoptions` 用来传递额外的参数来加密脚本。 `pack` 使用下面的命令来加密脚本::

    pyarmor obfuscate -r --output DIST EXTRA_OPTIONS SCRIPT

其中 `EXTRA_OPTIONS` 会被该选项所替换。

更多详细说明，请参考 :ref:`如何打包加密脚本`.

.. important::

   不要使用 `pack` 打包加密过的脚本，直接打包原来的脚本就可以

**示例**

* 加密脚本 foo.py 并打包到 `dist/foo` 下面::

    pyarmor pack foo.py

* 传递额外的参数运行 `PyInstaller`::

    pyarmor pack --options '-w --icon app.ico' foo.py

* 打包的时候不要加密目录 `venv` 和 `test` 下面的所有文件::

    pyarmor pack -x " --exclude venv --exclude test" foo.py

* 使用高级模式加密脚本，然后打包成为一个可执行文件::

    pyarmor pack -e " --onefile" -x " --advanced" foo.py

* 如果使用了 `PyInstaller` 的选项 `-n` 改变了打包文件的名称，必须同时
  使用选项 `-s`, 例如::

    pyarmor pack -e " -n my_app" -s "my_app.spec" foo.py


.. _hdinfo:

hdinfo
------

显示当前机器的硬件信息，例如硬盘序列号，网卡Mac地址等。

这些信息主要用来为加密脚本生成许可文件的时候使用。

**语法**::

    pyarmor hdinfo

如果没有装 `pyarmor`, 也可以在这里下载获取硬件信息的小工具 `hdinfo`

    https://github.com/dashingsoft/pyarmor-core/tree/master/#hdinfo

然后直接运行::

    hdinfo

获取得到的硬件信息和这里显示的是一样的。

.. _init:

init
----

创建管理加密脚本的工程文件。

**语法**::

    pyarmor init <options> PATH

**选项**:

-t, --type <auto,app,pkg>  工程类型，默认是 `auto`
-s, --src SRC              脚本所在路径，默认是当前路径
-e, --entry ENTRY          主脚本名称

**描述**

这个命令会在 `PATH` 指定的路径创建一个工程配置文件
:file:`.pyarmor_config` ，这个一个 JSON 格式的文件。

如果选项 `--type` 是 `auto` （也是默认情况），那么工程类型根据主脚本命
令来判断。如果主脚本是 `__init__.py` , 那么工程类型就是 `pkg` , 否则就
是 `app` 。

如果新的工程类型为 `pkg` ，不管是自动判断还是选项指定， `init` 命令都
会设置工程属性 `is_package` 为 `1` ，这个属性的默认值是 `0` 。

工程创建之后，可以使用命令 config_ 进行修改和配置。

**示例**

* 在当前路径创建一个工程::

    pyarmor init --entry foo.py

* 创建一个工程在构建路径 `obf`::

    pyarmor init --entry foo.py obf

* 创建一个 `pkg` 类型的工程::

    pyarmor init --entry __init__.py

* 在构建路径 `obf` 创建一个工程，管理在 `/path/to/src` 处的脚本::

    pyarmor init --src /path/to/src --entry foo.py obf

.. _config:

config
------

修改工程配置。

**语法**::

    pyarmor config <options> [PATH]

**选项**

--name NAME                     内部名称
--title TITLE                   显示标题
--src SRC                       脚本所在路径
--output OUTPUT                 保存加密脚本的输出路径
--manifest TEMPLATE             过滤脚本的模板语句
--entry SCRIPT                  工程主脚本，可以多个，使用逗号分开
--is-package <0,1>              管理的脚本是一个 Python 包类型
--restrict-mode <0,1,2,3,4>     设置约束模式
--obf-mod <0,1>                 是否加密整个模块对象
--obf-code <0,1,2>              是否加密每一个函数
--wrap-mode <0,1>               是否启用包裹模式加密函数
--advanced-mode <0,1>           是否使用高级模式加密脚本
--cross-protection <0,1>        是否插入交叉保护代码到主脚本
--runtime-path RPATH            设置运行文件所在路径
--plugin NAME                   设置需要插入到主脚本的代码文件
--package-runtime <0,1,2>       是否保存运行文件到一个单独的目录

**描述**

在工程所在路径运行该命令，修改一个或者多个工程属性::

    pyarmor config --option new-value

或者在命令的最后面指定工程所在的路径::

    pyarmor config --option new-value /path/to/project

选项 `--entry` 用来指定工程主脚本，可以是多个，以逗号分开::

    main.py, another/main.py, /usr/local/myapp/main.py

主脚本可以是绝对路径，也可以是相对路径，相对于 `src` 指定的路径。

选项 `--manifest` 用来选择和设置工程包含的脚本。默认值为 `src` 下面的
所有 `.py` 文件::

    global-include *.py

多个模式使用逗号分开，例如::

    global-include *.py, exclude __mainfest__.py, prune test

关于所有支持的模式，参考 https://docs.python.org/2/distutils/sourcedist.html#commands

**示例**

* 修改工程名称和标题::

    pyarmor config --name "project-1"  --title "My PyArmor Project"

* 修改工程主脚本::

    pyarmor config --entry foo.py,hello.py

* 排除路径 `build` 和 `dist` ，下面的的所有 `.py` 文件会被忽略::

    pyarmor config --manifest "global-include *.py, prune build, prune dist"

* 使用非包裹模式加密脚本，这样可以提高加密脚本运行速度，但是会降低安全性::

    pyarmor config --wrap-mode 0

* 配置主脚本的插件，下面的例子中会把 `check_ntp_time.py` 的内容插入到
  主脚本，这个脚本会检查网络时间，超过有效期会自动退出::

    pyarmor config --plugin check_ntp_time.py

* 清除所有插件::

    pyarmor config --plugin clear

.. _build:

build
-----

加密工程中的所有脚本。

**选项**

-B, --force               强制加密所有脚本，默认情况只加密上次构建之后修改过的脚本
-r, --only-runtime        只生成运行依赖文件
-n, --no-runtime          只加密脚本，不要生成运行依赖文件
-O, --output OUTPUT       输出路径，如果设置，那么工程属性里面的输出路径就无效
--platform NAME           指定加密脚本的运行平台，仅用于跨平台发布

**描述**

可以直接在工程所在路径运行该命令::

    pyarmor build

或者在命令行指定工程所在路径::

    pyarmor build /path/to/project

**示例**

* 递增式加密工程中的脚本，上次运行该命令之后，没有修改过的脚本不会再被
  加密::

    pyarmor build

* 强制加密工程中的所有脚本，即便是没有修改::

    pyarmor build -B

* 仅仅生成运行加密脚本需要的依赖文件，不要加密脚本::

    pyarmor build -r

* 只加密脚本，不要生成其他的依赖文件::

    pyarmor build -n

* 忽略工程中设置的输出路径，保存加密脚本到新路径::

    pyarmor build -B -O /path/to/other

* 在 MacOS 平台下加密脚本，这些加密脚本将在 Ubuntu 下面运行，使用下面
  的命令进行加密::

    pyarmor download --list
    pyarmor download linux_x86_64

    pyarmor build -B --platform linux_x86_64

.. _info:

info
----

显示工程配置信息。

**语法**::

    pyarmor info [PATH]

**描述**

可以直接在工程所在路径运行该命令::

    pyarmor info

或者在命令行指定工程所在路径::

    pyarmor info /path/to/project

.. _check:

check
-----

检查工程文件的配置是否正确。

**语法**::

    pyarmor check [PATH]

**描述**

可以直接在工程所在路径运行该命令::

    pyarmor check

或者在命令行指定工程所在路径::

    pyarmor check /path/to/project

.. _benchmark:

banchmark
---------

测试加密脚本的性能。

**语法**::

    pyarmor benchmark <options>

**选项**:

-m, --obf-mode <0,1>   是否加密模块
-c, --obf-code <0,1>   是否单独加密每一个函数
-w, --wrap-mode <0,1>  是否使用包裹模式加密函数
--debug                不要清理生成的测试脚本

**描述**

主要用来检查加密脚本的性能，命令输出包括初始化加密脚本运行环境需要的额
外时间，以及不同加密模式下面导入模块、运行不同大小的代码块需要消耗的额
外时间。

**示例**

* 测试默认加密模式的性能::

    pyarmor benchmark

* 测试不使用包裹模式的性能::

    pyarmor benchmark --wrap-mode 0

* 查看测试过程中使用的脚本，保存在 `.benchtest` 目录下面::

    pyarmor benchmark --debug

.. _register:

register
--------

生效注册文件，显示注册信息。

**语法**::

    pyarmor register [KEYFILE]

**描述**

购买 PyArmor 之后会通过邮件发送注册文件给你，然后使用这个命令使之生效::

    pyarmor register /path/to/pyarmor-regfile-1.zip

查看注册信息::

    pyarmor register

.. _download:

download
--------

查看和下载不同平台下面的预编译的动态库。

**语法**::

    pyarmor download <options> PLAT-ID

**选项**:

--list PATTERN        查看所有可用的预编译动态库
-O, --output NAME     下载之后保存的名称

**描述**

常用平台的预编译动态库已经和 PyArmor 的安装包一起发布，大部分的嵌入式
设备可以自动下载相应的预编译动态库。但是对于部分无法识别平台的嵌入式设
备，就需要人工下载。例如，启动过程提示::

    ERROR: Unsupport platform linux32/armv7l

那么首先查看所有的预编译动态库::

    pyarmor download --list

在列表中发现平台 `armv7` 可以使用，那么就可以使用下面的命令进行下载::

    pyarmor download --output linux32/armv7l armv7

也可以对平台进行过滤，例如查看 `linux32` 下所有可用的预编译动态库::

    pyarmor download --list linux32

.. include:: _common_definitions.txt
