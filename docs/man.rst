.. _命令手册:


命令手册
========

PyArmor 是一个命令行工具，用来加密脚本，绑定加密脚本到固定机器或者设置加密脚本的有效期。

`pyarmor` 的语法格式::

    pyarmor <command> [options]

常用的命令包括::

    obfuscate    加密脚本
    licenses     为加密脚本生成新的许可文件
    pack         加密脚本然后直接打包成为单独执行的文件
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
    runtime      创建运行辅助包

可以运行 `pyarmor <command> -h` 查看各个命令的详细使用方法。

.. note::

   从 v5.7.1 开始，下面这些命令的首字母可以作为别名直接使用::

       obfuscate, licenses, pack, init, config, build

   例如::

       pyarmor o 等价于 pyarmor obfuscate

通用选项
--------

-v, --version                显示版本信息
-q, --silent                 在控制台不显示日志
-d, --debug                  打印更多的信息用于发现命令执行过程中问题
--home PATH                  指定 pyarmor 注册时的路径，通常用于同一台电脑注册有多个 pyarmor
--boot PLATID                指定加密使用的平台特征，仅用于特殊情况下的跨平台加密

这些选项可以被用在 `pyarmor` 之后，子命令之前，例如，使用调试选项打印
更多的错误信息::

    pyarmor -d obfuscate foo.py

不要打印加密过程中的输出信息::

    pyarmor --silent obfuscate foo.py

在同一台电脑上使用为第二个产品购买的许可证加密脚本::

    pyarmor --home ~/.pyarmor-2 register pyarmor-keyfile-2.zip
    pyarmor --home ~/.pyarmor-2 obfuscate foo.py

.. _obfuscate:

obfuscate
---------

加密 Python 脚本。

**语法**::

    pyarmor obfuscate <options> SCRIPT...

.. _obfuscate 命令选项:

**选项**

-O, --output PATH               输出路径，默认是 `dist`
-r, --recursive                 递归模式加密所有的脚本
-s, --src PATH                  当主脚本不在顶层目录的时候指定搜索脚本的路径
--exclude PATH                  在递归模式下排除某些目录，多个目录使用逗号分开，或者使用该选项多次
--exact                         只加密命令行中列出的脚本
--no-bootstrap                  在主脚本中不要插入引导代码
--no-cross-protection           在主脚本中不要插入交叉保护代码
--plugin NAME                   在加密之前，向主脚本中插入代码。这个选项可以使用多次。
--platform NAME                 指定运行加密脚本的平台
--advanced <0,1,2,3,4>          使用高级模式 `1` ，超级模式 `2` ， 虚拟模式 `3` 和 `4` 加密脚本
--restrict <0,1,2,3,4>          设置约束模式
--package-runtime <0,1>         是否把运行文件保存为包的形式
--no-runtime                    不生成任何运行辅助文件，只加密脚本
--runtime PATH                  使用预先生成的运行辅助包
--bootstrap <0,1,2,3>           如何生成引导代码
--enable-suffix                 生成带有后缀名称的运行辅助包
--obf-code <0,1,2>              指定代码加密模式
--obf-mod <0,1,2>               指定模块加密模式
--wrap-mode <0,1>               指定包裹加密模式
--with-license FILENAME         使用指定的许可文件，特殊值 `outer` 表示使用外部许可文件
--cross-protection FILENAME     使用定制的交叉保护脚本

**描述**

PyArmor 首先检查用户根目录下面是否存在 :file:`.pyarmor_capsule.zip` ，如果不存在，
那么创建一个新的。

接着搜索需要加密的脚本，共有三种搜索模式：

* 默认模式： 搜索和主脚本相同目录下面的所有 `.py` 文件
* 递归模式： 递归搜索和主脚本相同目录下面的所有 `.py` 文件，使用选项 ``--recursive``
* 精准模式： 仅仅加密命令行中列出的脚本，使用选项 ``--exact``

加密命令只处理 `.py` 文件，不会拷贝数据文件到输出目录。如果被加密的包有很多数据
文件，可以先把整个包拷贝到输出目录，然后在进行加密，这样加密后的 `.py` 文件就会
覆盖输出目录中 `.py` 文件。

PyArmor 首先会修改主脚本，在其中插入交叉保护代码，详细处理过程请参考 :ref:`对主脚本的特殊处理`

如果在命令行中指定了插件，那么 PyArmor 会扫描全部的脚本，根据规则注入相关插件代
码，详细处理过程参考 :ref:`如何处理插件` 。

接着把搜索到脚本全部加密，保存到输出目录 `dist` 。

然后创建 :ref:`运行辅助包` ，保存到输出目录 `dist` 。

最后插入 :ref:`引导代码` 到主脚本。

当主脚本没有在顶层目录的时候，需要使用选项 ``--src`` 用来指定搜索 ``.py`` 文件的
路径。例如::

    # 没有 --src 的话，"./mysite" 是搜索脚本的路径
    pyarmor obfuscate --src "." --recursive mysite/wsgi.py

选项 ``--plugin`` 主要用于扩展加密脚本的授权方式，例如检查网络时间来校验有效期等，
这个选项指定的插件名称会在加密之前插入到主脚本中。插件对应的文件为当前目录下面
`名称.py` ，如果插件不在当前路径，可以使用绝对路径指定插件名称，更多详细的内容参
考 :ref:`如何处理插件` 。关于插件的使用实例，请参考 :ref:`使用插件扩展认证方式`

选项 ``--platform`` 用于指定加密脚本的运行平台，仅用于跨平台发布。因为加密脚本的
运行文件中包括平台相关的动态库，所以跨平台发布需要指定该选项。这个选项可以使用多
次，以支持加密脚本运行于不同平台。从 v5.7.5 开始，所有平台名称已经标准化，可用的
平台名称可以使用命令 `download`_ 查看。

选项 ``--restrict`` 用于指定加密脚本的约束模式，关于约束模式的详细说明，参考
:ref:`约束模式`

选项 ``--advanced`` 用来启用一些高级特性以增加安全性，它可用的值有

* 0: 禁用所有高级特性
* 1: 启用 :ref:`高级模式`
* 2: 启用 :ref:`超级模式`
* 3: 启用 :ref:`高级模式` 和 :ref:`虚拟模式`
* 4: 启用 :ref:`超级模式` 和 :ref:`虚拟模式`

关于选项 ``--runtime`` 的用法，请参考命令 `runtime`_

**运行辅助文件**

如果启用了超级模式，那么只有一个运行辅助文件::

    pytransform.pyd / pytransform.so

对于其他任何模式，默认情况下，所有运行时刻文件会作为包保存在一个单独的目录
`pytransform` 下面::

    pytransform/
        __init__.py
        _pytransform.so / _pytransform.dll / _pytransform.dylib

只有当选项 ``--package-runtime`` 设置为 `0` 的时候，生成的运行辅助文件和加密脚本
存放在相同的目录下面::

    pytransform.py
    _pytransform.so / _pytransform.dll / _pytransform.dylib

如果指定了选项 ``--enable-suffix`` ，那么运行辅助包（模块）的名称会包含一个后缀，
例如， ``pytransform_xxxx`` 。这里 ``xxxx`` 是根据 PyArmor 注册码得到的具有唯一
性的字符串。

**引导代码**

如果启用了超级模式，那么不存在引导代码的概念，所有脚本都会有一行从运行辅助模块导
入的语句::

    from pytransform import pyarmor

对于其他任何模式，默认情况下，下面的 :ref:`引导代码` 会被插入到加密后的主脚本中::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()

当主脚本是 ``__init__.py`` 的时候， 会使用包含一个 ``.`` 的相对导入的方式::

    from .pytransform import pyarmor_runtime
    pyarmor_runtime()

如果选项 ``--bootstrap`` 是 ``2``, 那么 :ref:`引导代码` 总是使用绝对导入的方式；
如果它被设置为 ``3`` ，那么总是使用包含前置 ``.`` 的相对导入方式。

另外如果指定了选项 ``--enable-suffix`` 的话， :ref:`引导代码` 可能会是这样子的::

    from pytransform_vax_000001 import pyarmor_runtime
    pyarmor_runtime(suffix='vax_000001')

如果设置了 ``--no-bootstrap`` 或者 ``--bootstrap`` 为 `0` ，那么就不会插入引导代
码到主脚本中。

**示例**

* 加密当前目录下的所有 `.py` 脚本，保存到 `dist` 目录::

     pyarmor obfuscate foo.py

* 加密当前目录下的所有 `.py` 脚本，保存到 `dist` 目录，但是有多个启动脚本::

     pyarmor obfuscate foo.py foo-svr.py foo-client.py

* 递归加密当前目录下面的所有 `.py` 脚本，保存到 `dist` 目录::

     pyarmor obfuscate --recursive foo.py

* 递归加密当前目录下面的所有脚本，主脚本在子目录 `mysite/` 下面::

     pyarmor obfuscate --src "." --recursive mysite/wsgi.py

* 仅仅递归加密指定目录下面的所有 `.py` 脚本，没有主脚本，也不生成运行辅助文件::

     pyarmor obfuscate --recursive --no-runtime .
     pyarmor obfuscate --recursive --no-runtime src/

* 除了 `build` 和 `dist` 之外，递归加密当前目录下面的所有 `.py` 脚本，
  保存到 `dist` 目录::

     pyarmor obfuscate --recursive --exclude build,dist foo.py
     pyarmor obfuscate --recursive --exclude build --exclude tests foo.py

* 仅仅加密两个脚本 `foo.py`, `moda.py`::

     pyarmor obfuscate --exact foo.py moda.py

* 只加密主脚本 `foo.py` ，不要生成其他任何运行文件::

    pyarmor obfuscate --no-runtime --exact foo.py

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

* 仅当插件中的函数 `assert_armored` 被调用的时候导入插件 `assert_armored`::

     pyarmor obfuscate --plugin @assert_armored foo.py

* 在 MacOS 平台下加密脚本，这些加密脚本将在 Ubuntu 下面运行，使用下面
  的命令进行加密::

    pyarmor obfuscate --platform linux.x86_64 foo.py

* 使用高级模式加密脚本::

    pyarmor obfuscate --advanced 1 foo.py

* 使用约束模式 2 加密脚本::

    pyarmor obfuscate --restrict 2 foo.py

* 使用约束模式 4 加密当前目录下面除了 `__init__.py` 之外的所有 `.py` 文件::

    pyarmor obfuscate --restrict 4 --exclude __init__.py --recursive .

* 生成一个可以被其他加密模块导入的包::

    cd /path/to/mypkg
    pyarmor obfuscate -r --enable-suffix --output dist/mypkg __init__.py

* 使用超级模式进行加密，并使用有限制期限的许可文件::

    pyarmor licenses -e 2020-10-05 regcode-01
    pyarmor obfuscate --with-license licenses/regcode-01/license.lic \
                      --advanced 2 foo.py

* 使用超级模式进行加密，并使用定制的交叉保护脚本，同时使用外部的许可文
  件，不要把 ``license.lic`` 嵌入到扩展模块中::

    pyarmor obfuscate --cross-protection build/pytransform_protection.py \
                      --with-license outer --advanced 2 foo.py

* 使用预先生成的运行辅助包来加密脚本::

    pyarmor runtime --advanced 2 --with-license outer -O myruntime-1
    pyarmor obfuscate --runtime myruntime-1 --with-license licenses/r001/license.lic foo.py
    pyarmor obfuscate --runtime @myruntime-1 --exact foo-2.py foo-3.py

.. _licenses:

licenses
--------

为加密脚本生成新的许可文件

**语法**::

    pyarmor licenses <options> CODE

.. _licenses 命令选项:

**选项**

-O OUTPUT, --output OUTPUT            输出路径，可以为 stdout 或者 stderr
-e YYYY-MM-DD, --expired YYYY-MM-DD   加密脚本的有效期
-d SN, --bind-disk SN                 绑定加密脚本到硬盘序列号
-4 IPV4, --bind-ipv4 IPV4             绑定加密脚本到指定IP地址
-m MACADDR, --bind-mac MACADDR        绑定加密脚本到网卡的Mac地址
-x, --bind-data DATA                  用于扩展认证类型的时候传递认证数据信息
--disable-restrict-mode               运行加密脚本时候禁用所有约束模式
--enable-period-mode                  运行加密脚本时候周期性（每小时）检查许可文件
--fixed key,...                       绑定加密脚本到特定的 Python 解释器

**描述**

运行加密脚本必须有一个认证文件 :file:`license.lic` 。一般在加密脚本的同时，会自
动生成一个缺省的认证文件。但是这个缺省的认证文件允许加密脚本运行在任何机器并且永
不过期。如果你需要对加密脚本进行限制，那么需要使用该命令生成新的许可文件，并覆盖
原来的许可文件。

例如，下面的命令生成一个有使用期限的认证文件::

    pyarmor licenses --expired 2019-10-10 mycode

生成的新的认证文件保存在默认输出路径和注册码组合路径 `licenses/mycode` 下面，使
用这个新的许可文件覆盖默认的许可文件::

    cp licenses/mycode/license.lic dist/pytransorm/

另外一个例子，限制加密脚本在固定 Mac 地址，同时设置使用期限::

    pyarmor licenses --expired 2019-10-10 --bind-mac 2a:33:50:46:8f tom
    cp licenses/tom/license.lic dist/

在这之前，一般需要运行命令 :ref:`hdinfo` 得到硬件的相关信息::

    pyarmor hdinfo

选项 `-x` 可以把任意字符串数据存放到许可文件里面，主要用于自定义认证类型的时候，
传递参数给自定义认证函数。例如::

    pyarmor licenses -x "2019-02-15" tom

然后在加密脚本中，可以从认证文件的信息中查询到传入的数据。例如::

    from pytransfrom import get_license_info
    info = get_license_info()
    print(info['DATA'])

也可以输出生成的许可文件到标准输出，例如::

    pyarmor --silent licenses --output stdout -x "2019-05-20" reg-0001

有时候我们需要把加密脚本绑定到某一个 Python 解释器上，例如，不允许用户使用自定义
的 Python 解释器来执行加密脚本，必须使用官方版本。这时候可以使用选项 ``--fixed``
来指定解释器的特征码。例如，使用特殊的特征码 `1` 绑定到当前 Python 解释器::

    pyarmor licenses --fixed 1

也可以绑定到多个 Python 解释器，只需要把多个特征码使用逗号分开即可::

    pyarmor licenses --fixed 4265050,5386060

如何获得解释器的特征码，请参考 :ref:`绑定加密脚本到固定的 Python 解释器`

需要注意的是在32位的 Windows 平台上面无法使用这个特性，因为在不同的机器上甚至每
次运行 Python 的特征码都是不相同的。

.. note::

   这里有一个实际使用的例子 :ref:`使用插件扩展认证方式`

.. _pack:

pack
----

加密并打包脚本或者工程。

**语法**::

    pyarmor pack <options> SCRIPT | PROJECT

.. _pack 命令选项:

**选项**

-O, --output PATH       输出路径
-e, --options OPTIONS   传递额外的参数到 `PyInstaller`_
-x, --xoptions OPTIONS  传递额外的参数到 `obfuscate`_ 去加密脚本
-s FILE                 使用外部的 .spec 文件来打包加密脚本
--clean                 打包之前删除缓存的文件
--without-license       不要将加密脚本的许可文件打包进去
--with-license FILE     使用指定的许可文件替换加密脚本默认的许可文件
--debug                 不要删除打包过程生成的中间文件
--name                  指定最终生成的包的名称，默认是脚本的名称

**描述**

命令 `pack`_ 首先调用 `PyInstaller`_ 生成一个和主脚本同名的 `.spec` 文件，选项
``--e`` 的值会原封不动的被传递给 `PyInstaller`_ ，但是不能传递这些选项 ``-y`` ，
``--noconfirm`` ， ``-n`` ， ``--name`` ， ``--distpath`` ， ``--specpath`` ，因
为这些会被 `pack`_ 命令内部使用。

当 `pack`_ 命令失败的时候，首先要确认脚本文件可以直接用 `PyInstaller`_ 打包成功，例如::

    pyinstaller foo.py

通常情况下，只要 `PyInstaller`_ 能打包成功，然后额外的参数通过 ``-e`` 传递过去，
命令 `pack`_ 也不会有问题。

接下来 `pack`_ 会递归加密主脚本所在目录下面的所有 `.py` 文件。它会使用 ``-x`` 中
指定的额外选项来调用命令 `obfuscate`_ ，但是不可以在 ``-x`` 传入选项 ``-r`` ，
``--output`` ， ``--package-runtime`` 等，这些会被 `pack`_ 内部使用。当打包一个
工程的时候， `pack`_ 会直接调用命令 `build`_ 加密工程，选项 ``-x`` 的值会被忽略，
这时候加密选项的控制是通过配置工程来实现。

然后 `pack`_ 会基于原来的 `.spec` 文件，创建一个新的 `.spec` 文件，增加一些语句
用于把原来的脚本替换为加密后的脚本。

最后 `pack`_ 再次调用 `PyInstaller`_ 使用这个打过补丁的 `.spec` 文件来创建最终的
输出。更多详细说明，请参考 :ref:`如何打包加密脚本`.

如果设置了选项 ``--debug`` ，例如::

    pyarmor pack --debug foo.py

下面这些中间文件会被保留下来，正常情况下它们在打包成功之后会被删除::

    foo.spec
    foo-patched.spec
    dist/obf/temp/hook-pytransform.py
    dist/obf/*.py                       # All the obfuscated scripts

这个打过补丁的 `foo-patched.spec` 可以被 `PyInstaller`_ 直接用来打包加密脚本，例
如::

    pyinstaller -y --clean foo-patched.spec

当有些脚本被修改之后，只需要重新加密，然后调用这个命令快速打包，而加密命令
`obfuscate`_ 需要的全部选项可以从命令 `pack`_ 在控制台的输出中找到。

如果需要更改输出的可执行文件的名称，直接设置选项 ``--name`` ，不要使用 ``-e`` 把
这个选项传递给 `PyInstaller`_ ，因为这个选项需要一些特殊处理。

如果已经有一个能够打包的 `.spec` 文件，只需要通过选项 ``-s`` 指定这个文件（这种
情况下选项 ``-e`` 的值会被忽略），例如::

    pyarmor pack -s foo.spec foo.py

主脚本（这里是 `foo.py`) 需要在命令行列出，否则 `pack`_ 就不知道那些脚本需要加密，
详细说明请参考 :ref:`使用定制的 .spec 文件打包加密脚本` 。

如果有很多数据文件或者隐含模块，最好的方式是使用 Hook 脚本来自动发现它们。首先创
建一个文件 ``hook-sys.py`` ，下面是一些示例代码::

    from PyInstaller.utils.hooks import collect_data_files, collect_all
    datas, binaries, hiddenimports = collect_all('my_module_name')
    datas += collect_data_files('submodule')
    hiddenimports += ['_gdbm', 'socket', 'h5py.defs']
    datas += [ ('/usr/share/icons/education_*.png', 'icons') ]

接下来使用额外选项 ``--additional-hooks-dir .`` 来调用 `pack`_ ，告诉
`PyInstaller`_ 在当前路径下面搜索 Hook 脚本::

    pyarmor pack -e " --additional-hooks-dir ." foo.py

更多关于 Hook 脚本的信息，参考
https://pyinstaller.readthedocs.io/en/stable/hooks.html#understanding-pyinstaller-hooks

最后，如果打包过程出现问题，打开 PyArmor 调试标志来输出详细的错误信息::

    pyarmor -d pack ...

**示例**

* 加密脚本 foo.py 并打包到 `dist/foo` 下面::

    pyarmor pack foo.py

* 删除缓存的 `foo.spec` 和其他中间文件，开始一个全新的打包::

    pyarmor pack --clean foo.py

* 使用已经写好的 `myfoo.spec` 来打包加密脚本::

    pyarmor pack -s myfoo.spec foo.py

* 传递额外的参数运行 `PyInstaller`::

    pyarmor pack --options '-w --icon app.ico' foo.py
    pyarmor pack -e " --icon images\\app.ico" foo.py

* 打包的时候不要加密目录 `venv` 和 `test` 下面的所有文件::

    pyarmor pack -x " --exclude venv --exclude test" foo.py

* 使用高级模式加密脚本，然后打包成为一个可执行文件::

    pyarmor pack -e " --onefile" -x " --advanced 1" foo.py

* 使用有时间限制的许可证打包一个可执行文件::

    pyarmor licenses -e 2020-12-25 cy2020
    pyarmor pack --with-license licenses/cy2020/license.lic foo.py

* 生成名称为 ``my_app`` 的安装包::

    pyarmor pack --name my_app foo.py

* 使用高级模式加密脚本并打包一个工程::

    pyarmor init --entry main.py
    pyarmor config --advanced 1
    pyarmor pack .

.. note::

   从 v5.9.0 开始，也可以直接打包一个工程，只需要最后一个参数指定工程所在的路径，
   就可以加密工程中的文件，并使用工程主脚本进行打包。例如::

     pyarmor init --entry main.py
     pyarmor pack .

   首先在当前目录创建一个工程，然后直接打包该工程，使用工程的好处是可以完全定制
   加密选项。

.. note::

   在 Windows 下面，附加选项里面如果用到反斜杠，必须是两个。例如::

     pyarmor pack -e " --icon images\\app.ico" foo.py

.. important::

   命令 `pack` 会自动加密脚本，所以不要使用该命令去打包加密后的脚本，打包加密脚
   本会导致错误，因为脚本加密之后是无法自动找到的其他被引用的模块的。

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

这个命令会在 `PATH` 指定的路径创建一个工程配置文件 :file:`.pyarmor_config` ，这
个一个 JSON 格式的文件。

如果选项 ``--type`` 是 `auto` （也是默认情况），那么工程类型根据主脚本命令来判断。
如果主脚本是 `__init__.py` , 那么工程类型就是 `pkg` , 否则就是 `app` 。

如果新的工程类型为 `pkg` ，不管是自动判断还是选项指定， `init` 命令都会设置工程
属性 `is_package` 为 `1` ，这个属性的默认值是 `0` 。

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

--name NAME                     工程名称
--title TITLE                   显示标题
--src SRC                       工程脚本所在的路径，用于匹配模板命令
--output OUTPUT                 保存加密脚本的输出路径
--manifest TEMPLATE             过滤脚本的模板语句
--entry SCRIPT                  工程主脚本，可以多个，使用逗号分开
--is-package <0,1>              管理的脚本是一个 Python 包类型
--restrict <0,1,2,3,4>          设置约束模式
--obf-mod <0,1,2>               是否加密整个模块对象
--obf-code <0,1,2>              是否加密每一个函数
--wrap-mode <0,1>               是否启用包裹模式加密函数
--advanced <0,1,2,3,4>          使用高级模式 `1` ，超级模式 `2` ，虚拟模式 `3` 和 `4` 加密脚本
--cross-protection <0,1>        是否插入交叉保护代码到主脚本，也可以直接指定脚本名称
--runtime-path RPATH            设置运行文件所在路径
--plugin NAME                   设置需要插入到主脚本的代码文件，这个选项可以使用多次
--package-runtime <0,1>         是否保存运行文件为包的形式
--bootstrap <0,1,2,3>           如何生成引导代码
--with-license FILENAME         使用指定的许可文件，特殊值 `outer` 表示使用外部许可文件

**描述**

在工程所在路径运行该命令，修改一个或者多个工程属性::

    pyarmor config --option new-value

或者在命令的最后面指定工程所在的路径::

    pyarmor config --option new-value /path/to/project

选项 ``--entry`` 用来指定工程主脚本，可以是多个，以逗号分开::

    main.py, another/main.py, /usr/local/myapp/main.py

主脚本可以是绝对路径，也可以是相对路径，相对于 `src` 指定的路径。

选项 ``--manifest`` 用来选择和设置工程包含的脚本。默认值为 `src` 下面的所有 `.py`
文件::

    global-include *.py

多个模式使用逗号分开，例如::

    global-include *.py, exclude __mainfest__.py, prune test

关于所有支持的模式，参考 https://docs.python.org/2/distutils/sourcedist.html#commands

选项 ``--plugin`` 为空字符串有特殊作用，用来清除所有的插件。

所有选项的作用，请参考 :ref:`工程配置文件`

**示例**

* 修改工程名称和标题::

    pyarmor config --name "project-1"  --title "My PyArmor Project"

* 修改工程主脚本::

    pyarmor config --entry foo.py,hello.py

* 排除路径 `build` 和 `dist` ，下面的的所有 `.py` 文件会被忽略::

    pyarmor config --manifest "global-include *.py, prune build, prune dist"

* 使用非包裹模式加密脚本，这样可以提高加密脚本运行速度，但是会降低安全性::

    pyarmor config --wrap-mode 0

* 配置主脚本的插件，下面的例子中会把 `check_ntp_time.py` 的内容插入到主脚本，这
  个脚本会检查网络时间，超过有效期会自动退出::

    pyarmor config --plugin check_ntp_time.py

* 清除所有插件::

    pyarmor config --plugin ''

.. _build:

build
-----

加密工程中的所有脚本。

**语法**::

    pyarmor build <options> [PATH]

**选项**

-B, --force                     强制加密所有脚本，默认情况只加密上次构建之后修改过的脚本
-r, --only-runtime              只生成运行依赖文件
-n, --no-runtime                只加密脚本，不要生成运行依赖文件
--runtime PATH                  使用预先生成的运行辅助包
-O, --output OUTPUT             输出路径，如果设置，那么工程属性里面的输出路径就无效
--platform NAME                 指定加密脚本的运行平台，仅用于跨平台发布
--package-runtime <0,1>         是否保存运行文件为包的形式

**描述**

可以直接在工程所在路径运行该命令::

    pyarmor build

或者在命令行指定工程所在路径::

    pyarmor build /path/to/project

选择 ``--no-runtime`` 可能会影响 :ref:`引导代码` 的生成方式，设置之后在主脚本中
引导代码总是会使用绝对导入的方式。

选项 ``--platform`` 和 ``--package-runtime`` 的使用，请参考命令 `obfuscate`_

选项 ``--runtime`` 的使用，请参考命令 `runtime`_

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

    pyarmor build -B --platform linux.x86_64

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

-m, --obf-mod <0,1,2>        是否加密模块
-c, --obf-code <0,1,2>       是否单独加密每一个函数
-w, --wrap-mode <0,1>        是否使用包裹模式加密函数
-a, --advanced <0,1,2,3,4>   是否高级模式/超级模式加密函数
--debug                      保留测试使用的测试脚本

**描述**

主要用来检查加密脚本的性能，命令输出包括初始化加密脚本运行环境需要的额外时间，以
及不同加密模式下面导入模块、运行不同大小的代码块需要消耗的额外时间。

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

生效购买的 PyArmor 注册文件::

    pyarmor register /path/to/pyarmor-regfile-1.zip

查看注册信息::

    pyarmor register

.. _download:

download
--------

查看和下载不同平台下面的预编译的动态库。

**语法**::

    pyarmor download <options> NAME

**选项**:

--help-platform       显示所有支持的规范化平台名称
--list PATTERN        查看所有可用的预编译动态库
-O, --output PATH     下载之后保存的路径
--update              更新已经下载的动态库

**描述**

这个命令主要是用来下载其他平台的动态库，一般用于交叉平台的发布。

列出所有可用平台的规范化名称，例如::

    pyarmor download
    pyarmor download --help-platform
    pyarmor download --help-platform windows
    pyarmor download --help-platform linux.x86_64

下载其中的一个。例如::

    pyarmor download linux.armv7
    pyarmor download linux.x86_64

默认情况下，下载的文件是保存在目录 ``~/.pyarmor/platforms`` 下面，使用相应的路径
来存放不同平台的动态库。

选项 ``--list`` 会显示详细的动态库信息，同时也可以过滤平台，搜索名称、CPU 架构、
动态库特征等。例如::

    pyarmor download --list
    pyarmor download --list windows
    pyarmor download --list windows.x86_64
    pyarmor download --list JIT
    pyarmor download --list armv7

当 `pyarmor` 升级之后，已经下载的动态库不会自动更新，可以使用 ``--update`` 更新
全部已经下载的动态库。例如::

    pyarmor download --update

.. _runtime:

runtime
-------

创建 :ref:`运行辅助包`

**SYNOPSIS**::

    pyarmor runtime <options>

**OPTIONS**:

-O, --output PATH             输出路径，默认是 `dist`
-n, --no-package              不要使用包的形式来存放生成运行文件
-i, --inside                  创建包含引导脚本的包 `pytransform_bootstrap`
-L, --with-license FILE       使用这个文件替换默认的加密脚本许可文件，特殊值 `outer` 表示使用外部许可证
--platform NAME               生成其他平台下的运行辅助包
--enable-suffix               生成带有后缀名称的运行辅助包
--advanced <0,1,2,3,4>        生成高级运行辅助包

**DESCRIPTION**

这个命令主要用来创建 :ref:`运行辅助包`

因为使用相同的 :ref:`全局密钥箱` 加密的脚本可以共享 :ref:`运行辅助包` ，所以单独
创建运行辅助包之后，在加密脚本的时候就不需要每一次都重新生成运行辅助文件。

它同时也会在输出目录下面创建一个引导脚本 ``pytransform_bootstrap.py`` ，这是对一
个空脚本进行加密之后生成的，包含有 :ref:`引导代码` 。它最主要用途就是可以让没有
加密的脚本能够运行 :ref:`引导代码` 。例如，在脚本中导入这个模块之后，其他加密模
块就都可以被正常导入了::

    import pytransform_bootstrap
    import obf_mod

如果选项 ``--inside`` 被指定，那么将在输出目录使用包 ``pytransform_bootstrap``
的形式来保存引导脚本。

从 v6.2.0 开始，这个命令还会另外创建辅助脚本 ``pytransform_protection.py`` ，这
是默认的交叉保护脚本，这个脚本在运行时刻并不需要，它主要是作为模版来定制自己的交
叉保护脚本，参考 :ref:`定制交叉保护脚本`

选项 ``--advanced`` 用来生成高级运行辅助文件，例如为 :ref:`超级模式` 生成运行辅
助包。

选项 ``--platform`` 和 ``--enable-suffix`` 的使用，请参考命令 `obfuscate`_

在 v6.3.7 之后，运行辅助包会记住创建时候的选项 ``--advanced`` ， ``--platform``
， ``--enable-suffix`` 。当使用选项 ``--runtime`` 加密脚本的时候，就可以自动读取
这些设置，而不需要在命令行重新指定。例如::

    pyarmor runtime --platform linux.armv7 --enable-suffix --advanced 1 -O myruntime-1
    pyarmor obfuscate --runtime myruntime-1 foo.py

这里的加密命令等价于::

    pyarmor obfuscate --platform linux.armv7 --enable-suffix --advanced 1 foo.py

如果有多个入口脚本，加密的时候只需要自动读取相关的选项，而不需要再次拷贝运行辅助
文件，那么在路径的前面增加一个 ``@`` 就可以。例如::

    pyarmor obfuscate --runtime @myruntime-1 --exact foo-2.py foo-3.py

**EXAMPLES**

* 在默认输出路径 `dist` 下面创建 :ref:`运行辅助包` ``pytransform``::

    pyarmor runtime

* 创建独立的 :ref:`运行辅助文件` ，但是不使用包的形式存放::

    pyarmor runtime -n

* 创建引导脚本在一个单独的包 ``pytransform_bootstrap``::

    pyarmor runtime -i

* 为 `armv7` 平台创建 :ref:`运行辅助包` ，并且设置加密脚本的使用期限::

    pyarmor licenses --expired 2020-01-01 code-001
    pyarmor runtime --with-license licenses/code-001/license.lic --platform linux.armv7

* 为超级模式创建运行辅助包::

    pyarmor runtime --super-mode
    pyarmor runtime --advanced 2

* 为超级模式创建运行辅助包，同时使用外部许可文件，也就是说，不要把许可
  文件嵌入到扩展模块里面::

    pyarmor runtime --super-mode --with-license outer

.. include:: _common_definitions.txt
