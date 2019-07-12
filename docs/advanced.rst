.. _高级用法:

高级用法
========

加密和使用多个包
----------------

假定有三个包 `pkg1`, `pkg2`, `pkg2` 需要加密，使用公共的运行辅助文件，
然后可以从其他脚本导入这些加密的包。

首先切换到工作路径，创建三个工程::

    mkdir build
    cd build

    pyarmor init --src /path/to/pkg1 --entry __init__.py pkg1
    pyarmor init --src /path/to/pkg2 --entry __init__.py pkg2
    pyarmor init --src /path/to/pkg3 --entry __init__.py pkg3

生成公共的运行辅助文件，保存在 `dist` 目录下面::

    pyarmor build --output dist --only-runtime pkg1

分别加密三个包，也保存到 `dist` 下面::

    pyarmor build --output dist --no-runtime pkg1
    pyarmor build --output dist --no-runtime pkg2
    pyarmor build --output dist --no-runtime pkg3

查看并使用加密的包::

    ls dist/

    cd dist
    python -c 'import pkg1
    import pkg2
    import pkg3'

跨平台发布加密脚本
------------------

因为加密脚本的运行文件中有平台相关的动态库，所以跨平台发布需要指定目标
平台。

首先使用命令 :ref:`download` 查看和下载目标平台的动态库文件::

    pyarmor download --list
    pyarmor download linux_x86_64

然后在加密脚本的时候指定目标平台名称::

    pyarmor obfuscate --platform linux_x86_64 foo.py

如果使用工程，那么::

    pyarmor build --platform linux_x86_64

使用不同版本 Python 加密脚本
----------------------------

如果装了多个版本的 Python ，那么使用 `pip` 安装的 `pyarmor` 使用的是默
认的 Python 版本。如果需要使用其他版本的 Python 来加密脚本，需要显示指
定 Python 解释器。

例如，首先找到 :file:`pyarmor.py` 的位置::

      find /usr/local/lib -name pyarmor.py

通常在大多数 linux 系统，它会在 `/usr/local/lib/python2.7/dist-packages/pyarmor`

然后使用下面的方式运行::

    /usr/bin/python3.6 /usr/local/lib/python2.7/dist-packages/pyarmor/pyarmor.py


使用不同的模式来加密脚本
------------------------

.. _代码加密模式:

代码加密模式
~~~~~~~~~~~~
一个 Python 文件中，通常有很多个函数（代码块）

* obf_code == 0

不会加密函数对应的代码块

* obf_code == 1

在这种情况下，会加密每一个函数对应的代码块，根据 :ref:`包裹模式` 的设
置，使用不同的加密方式

.. _包裹模式:

包裹模式
~~~~~~~~~
* wrap_mode == 0

当包裹模式关闭，代码块使用下面的方式进行加密::

    0   JUMP_ABSOLUTE            n = 3 + len(bytecode)

    3    ...
         ... 这里是加密后的代码块
         ...

    n   LOAD_GLOBAL              ? (__armor__)
    n+3 CALL_FUNCTION            0
    n+6 POP_TOP
    n+7 JUMP_ABSOLUTE            0

在开始插入了一条绝对跳转指令，当执行加密后的代码块时

1. 首先执行 JUMP_ABSOLUTE, 直接跳转到偏移 n

2. 偏移 n 处是调用一个 PyCFunction `__armor__` 。这个函数的功能会恢复
   上面加密后的代码块，并且把代码块移动到最开始（向前移动3个字节）

3. 执行完函数之后，跳转到偏移 0，开始执行原来的函数。

这种模式下，除了函数的第一次调用需要额外的恢复之外，随后的函数调用就和
原来的代码完全一样。

* wrap_mode == 1

当打开包裹模式之后，代码块会使用 `try...finally` 语句包裹起来::

    LOAD_GLOBALS    N (__armor_enter__)     N = co_consts 的长度
    CALL_FUNCTION   0
    POP_TOP
    SETUP_FINALLY   X (jump to wrap footer) X = 原来代码块的长度

    这里是加密的后的代码块

    LOAD_GLOBALS    N + 1 (__armor_exit__)
    CALL_FUNCTION   0
    POP_TOP
    END_FINALLY

这样，当被加密的函数开始执行的时候

1. 首先会调用 `__armor_enter__` 恢复加密后的代码块

2. 然后执行真正的函数代码

3. 在函数执行完成之后，进入 `final` 块，调用 `__armor_exit__` 重新加密
   函数对应的代码块

在这种模式下，函数的每一次执行完成都会重新加密，每一次执行都需要恢复。

.. _模块加密模式:

模块加密模式
~~~~~~~~~~~~
* obf_mod == 1

在这种模式下，最终生成的加密脚本如下::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 1)

其中第三个参数是代码块转换而成的字符串，它通过下面的伪代码生成::

    PyObject *co = Py_CompileString( source, filename, Py_file_input );
    obfuscate_each_function_in_module( co, obf_mode );
    char *original_code = marshal.dumps( co );
    char *obfuscated_code = obfuscate_algorithm( original_code  );
    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 1)", obfuscated_code );

* obf_mod == 0

在这种模式下，最终生成的代码如下（最后一个参数为 0）::

    __pyarmor__(__name__, __file__, b'\x02\x0a...', 0)

第三个参数的生成方式和上面的基本相同，除了最后一条语句替换为::

    sprintf( buffer, "__pyarmor__(__name__, __file__, b'%s', 0)", original_code );

使用方法请参考 :ref:`使用不同加密模式`

.. _约束模式:

约束模式
--------

在约束模式下，加密脚本必须是下面的形式之一::

    __pyarmor__(__name__, __file__, b'...')

    Or

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')

    Or

    from pytransform import pyarmor_runtime
    pyarmor_runtime('...')
    __pyarmor__(__name__, __file__, b'...')

例如，下面的加密脚本能够运行::

    $ cat a.py

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')

    $ python a.py

而下面的这个就无法运行，因为加密脚本中有一条额外的语句 `print`::

    $ cat b.py
    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'...')
    print(__name__)

    $ python b.py

从 PyArmor 5.2 开始, 约束模式是默认设置。如果需要禁用约束模式, 那么使
用下面的命令加密脚本::

    pyarmor obfuscate --restrict=0 foo.py

.. _使用插件扩展认证方式:

使用插件扩展认证方式
--------------------

PyArmor 可以通过插件来扩展加密脚本的认证方式，例如检查网络时间而不是本
地时间来校验有效期。

首先定义插件文件 :file:`check_ntp_time.py`:

.. code-block:: python

    # 当调试这个脚本的时候（还没有加密），需要把下面的两行代码前面的注释
    # 去掉，否则在无法使用 pytransform 模块的功能
    # from pytransform import pyarmor_init
    # pyarmor_init()

    from pytransform import get_license_code
    from ntplib import NTPClient
    from time import mktime, strptime
    import sys

    NTP_SERVER = 'europe.pool.ntp.org'
    EXPIRED_DATE = get_license_code()[4:]

    def check_expired():
        c = NTPClient()
        response = c.request(NTP_SERVER, version=3)
        if response.tx_time > mktime(strptime(EXPIRED_DATE, '%Y%m%d')):
            sys.exit(1)

然后在主脚本 :file:`foo.py` 插入下列两行注释::

    ...

    # {PyArmor Plugins}

    ...

    def main():
        # PyArmor Plugin: check_expired()

    if __name__ == '__main__':
        logging.basicConfig(level=logging.INFO)
        main()

执行下面的命令进行加密::

    pyarmor obfuscate --plugin check_ntp_time foo.py

这样，在加密之前，文件 :file:`check_ntp_time.py` 会插入到第一个注释标
志行之后::

    # {PyArmor Plugins}

    ... check_ntp_time.py 的文件内容

同时，第二个注释标志行的注释标志会被删除，替换后的内容为::

    def main():
        check_expired()

这样插件输出的函数就可以被脚本调用。

插件对应的文件一般存放在当前目录，如果存放在其他目录的话，可以指定绝对
路径，例如::

    pyarmor obfuscate --plugin /usr/share/pyarmor/check_ntp_time foo.py

也可以设置环境变量 `PYARMOR_PLUGIN` ，例如::

    export PYARMOR_PLUGIN=/usr/share/pyarmor/plugins
    pyarmor obfuscate --plugin check_ntp_time foo.py

最后为加密脚本生成许可文件，使用 `CODE` 来指定有效期::

    pyarmor licenses NTP:20190501

.. _打包加密脚本成为一个单独的可执行文件:

打包加密脚本成为一个单独的可执行文件
------------------------------------

使用下面的命令可以把脚本 `foo.py` 加密之后并打包成为一个单独的可执行文
件::

    pyarmor pack -e " --onefile" foo.py

其中 `--onefile` 是 `PyInstaller` 的选项，使用 `-e` 可以传递任何
`Pyinstaller` 支持的选项，例如，指定可执行文件的图标::

    pyarmor pack -e " --onefile --icon logo.ico" foo.py

如果不想把加密脚本的许可文件 `license.lic` 打包到可执行文件，而是和可
执行文件放在一起，这样方便为不同的用户生成不同的许可文件。那么需要使用
`PyInstaller` 提供的 `--runtime-hook` 功能在加密脚本运行之前把许可文件
拷贝到指定目录，下面是操作步骤：

1. 新建一个文件 `copy_licese.py`::

    import sys
    from os.path import join, dirname
    with open(join(dirname(sys.executable), 'license.lic'), 'rb') as fs:
        with open(join(sys._MEIPASS, 'license.lic'), 'wb') as fd:
            fd.write(fs.read())

2. 运行下面的命令打包加密脚本::

    pyarmor pack --clean --without-license \
            -e " --onefile --icon logo.ico --runtime-hook copy_license.py" foo.py

选项 `--without-license` 告诉 `pyamor` 不要把加密脚本的许可文件打包进
去，使用 `PyInstaller` 的选项 `--runtime-hook` 可以让打包好的可执行文
件，在启动的时候首先去调用 `copy_licesen.py` ，把许可文件拷贝到相应的
目录。

命令执行成功之后，会生成一个打包好的文件 `dist/foo.exe`

尝试运行这个可执行文件，应该会报错。

3. 使用命令 `pyarmor licenses` 生成新的许可文件，并拷贝到 `dist/` 下面

4. 这时候在双击运行 `dist/foo.exe`

.. 定制保护代码:

.. include:: _common_definitions.txt
