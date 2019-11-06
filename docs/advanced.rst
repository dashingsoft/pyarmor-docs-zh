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

生成公共的 :ref:`运行辅助包` ，保存在 `dist` 目录下面::

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

.. note::

   输出目录 `dist` 下面的运行辅助包 `pytransform` 可以被拷贝到任何的
   Python 可以导入的目录下面。


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

也可以创建一个便捷脚本 `/usr/local/bin/pyarmor3` ，内容如下::

    /usr/bin/python3.6 /usr/local/lib/python2.7/dist-packages/pyarmor/pyarmor.py "$*"

赋予其执行权限::

    chmod +x /usr/local/bin/pyarmor3

然后就可以直接使用 `pyarmor3`

在 Windows 下面就需要创建一个批处理 `pyarmor3.bat` ，内容如下::

    C:\Python36\python C:\Python27\Lib\site-packages\pyarmor\pyarmor.py %*

让 Python 自动识别加密脚本
--------------------------

下面有几种情况可能会需要让 Python 自动识别加密脚本:

* 几乎所有的脚本都会被作为主脚本来运行
* 在加密脚本中使用模块 `multiprocessing` 创建新进程
* 使用到 `Popen` 或者 `os.exec` 等调用加密后的脚本
* 其他任何需要在很多脚本里面插入引导代码的情况

一种解决方案就是为每一个相关的加密脚本添加引导代码，但是这会有些麻烦。
另外一种比较简单的解决方案就是让 Python 能够自动识别加密脚本，这样任何
一个加密脚本不需要引导代码就可以正常运行。

下面是基本操作步骤:

1. 首先加密一个空的脚本，同时生成 :ref:`运行辅助包`::

    echo "" > pytransform_bootstrap.py
    pyarmor obfuscate pytransform_bootstrap.py

2. 其次把 :ref:`运行辅助包` 拷贝到任意的 Python 路径。例如::

    # For windows
    mv dist/pytransform C:/Python37/Lib/site-packages/

    # For linux
    mv dist/pytransform /usr/local/lib/python3.5/dist-packages/

3. 然后把加密后的空脚本 `dist/pytransform_bootstrap.py` (包含 :ref:`引
   导代码`) 拷贝到任意的 Python 路径。例如::

     mv dist/pytransform_bootstrap.py C:/Python37/Lib/
     mv dist/pytransform_bootstrap.py /usr/lib/python3.5/

4. 修改 `{prefix}/lib/site.py` (Windows) 或者 `{prefix}/lib/pythonX.Y/site.py`
   (Linux), 插入一条导入语句，导入 `pytransform_bootstrap`::

    import pytransform_bootstrap

    if __name__ == '__main__':
        ...

也可以把这行代码添加到 `site.main` 里面，总之，只要能得到执行就可以。

这样就可以使用 `python` 直接运行加密脚本了。 这主要使用到了 Python 在启动过程中
默认会自动导入模块 `site` 的特性来实现，参考

https://docs.python.org/3/library/site.html

.. note::

    在 v5.7.0 之前，需要根据 :ref:`运行辅助文件` 人工创建 :ref:`运行辅助包`

使用不同的模式来加密脚本
------------------------

:ref:`高级模式` 是从 PyArmor 5.5.0 引入的新特性，默认情况下是没有启用的。如果需
要使用高级模式来加密脚本，额外指定选项 `--advanced`::

    pyarmor obfuscate --advanced foo.py

从 PyArmor 5.2 开始, :ref:`约束模式` 是默认设置。

使用选项 ``--restrict`` 指定其他约束模式，例如::

    pyarmor obfuscate --restrict=2 foo.py
    pyarmor obfuscate --restrict=3 foo.py

    # For project
    cd /path/to/project
    pyarmor config --restrict 4
    pyarmor build -B

如果需要禁用各种约束，那么使用下面的命令加密脚本::

    pyarmor obfuscate --restrict=0 foo.py

    # For project
    pyarmor config --restrict=0
    pyarmor build -B

指定 :ref:`代码加密模式`, :ref:`代码包裹模式`, :ref:`模块加密模式` 需
要 :ref:`使用工程` 来加密脚本，直接使用命令 :ref:`obfuscate` 无法改变
这些加密模式。例如::

    pyarmor init --src=src --entry=main.py .
    pyarmor config --obf-mod=1 --obf-code=1 --wrap-mode=0
    pyarmor build

.. _使用插件扩展认证方式:

使用插件扩展认证方式
--------------------

PyArmor 可以通过插件来扩展加密脚本的认证方式，例如检查网络时间而不是本地时间来校
验有效期。

首先定义插件文件 :file:`check_ntp_time.py`:

.. code-block:: python

    # 当调试这个脚本的时候（还没有加密），需要把下面的两行代码前面的注释
    # 去掉，否则在无法使用 pytransform 模块的功能
    # from pytransform import pyarmor_init
    # pyarmor_init()

    from ntplib import NTPClient
    from time import mktime, strptime
    import sys

    def get_license_data():
        from ctypes import py_object, PYFUNCTYPE
        from pytransform import _pytransform
        prototype = PYFUNCTYPE(py_object)
        dlfunc = prototype(('get_registration_code', _pytransform))
        rcode = dlfunc().decode()
        index = rcode.find(';', rcode.find('*CODE:'))
        return rcode[index+1:]

    def check_expired():
        NTP_SERVER = 'europe.pool.ntp.org'
        EXPIRED_DATE = get_license_data()
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
        # PyArmor Plugin: check_expired()
        check_expired()

这样插件输出的函数就可以被脚本调用。

插件对应的文件一般存放在当前目录，如果存放在其他目录的话，可以指定绝对
路径，例如::

    pyarmor obfuscate --plugin /usr/share/pyarmor/check_ntp_time foo.py

也可以设置环境变量 `PYARMOR_PLUGIN` ，例如::

    export PYARMOR_PLUGIN=/usr/share/pyarmor/plugins
    pyarmor obfuscate --plugin check_ntp_time foo.py

最后为加密脚本生成许可文件，使用 `-x` 把自定义的有效期存储到认证文件::

    pyarmor licenses -x 20190501 MYPRODUCT-0001
    cp licenses/MYPRODUCT-0001/license.lic dist/

.. note::

   为了提高安全性，可以要把 `ntplib.py` 内容全部拷贝过来，这样就不需要
   从外部导入 `NTPClient`

.. _打包加密脚本成为一个单独的可执行文件:

打包加密脚本成为一个单独的可执行文件
------------------------------------

使用下面的命令可以把脚本 `foo.py` 加密之后并打包成为一个单独的可执行文件::

    pyarmor pack -e " --onefile" foo.py

其中 `--onefile` 是 `PyInstaller` 的选项，使用 `-e` 可以传递任何 `Pyinstaller`
支持的选项，例如，指定可执行文件的图标::

    pyarmor pack -e " --onefile --icon logo.ico" foo.py

如果不想把加密脚本的许可文件 `license.lic` 打包到可执行文件，而是和可执行文件放
在一起，这样方便为不同的用户生成不同的许可文件。那么需要使用 `PyInstaller` 提供
的 `--runtime-hook` 功能在加密脚本运行之前把许可文件拷贝到指定目录，下面是具体的
操作步骤：

1. 新建一个文件 `copy_license.py`::

    import sys
    from os.path import join, dirname
    with open(join(dirname(sys.executable), 'license.lic'), 'rb') as src:
        with open(join(sys._MEIPASS, 'license.lic'), 'wb') as dst:
            dst.write(src.read())

2. 运行下面的命令打包加密脚本::

    pyarmor pack --clean --without-license \
            -e " --onefile --icon logo.ico --runtime-hook copy_license.py" foo.py

选项 `--without-license` 告诉 `pyamor` 不要把加密脚本的许可文件打包进去，使用
`PyInstaller` 的选项 `--runtime-hook` 可以让打包好的可执行文件，在启动的时候首先
去调用 `copy_licesen.py` ，把许可文件拷贝到相应的目录。

命令执行成功之后，会生成一个打包好的文件 `dist/foo.exe`

尝试运行这个可执行文件，应该会报错。

3. 使用命令 `pyarmor licenses` 生成新的许可文件，并拷贝到 `dist/` 下面

4. 这时候在双击运行 `dist/foo.exe`

.. _使用约束模式增加加密脚本安全性:

使用约束模式增加加密脚本安全性
------------------------------

默认约束模式仅限制不能修改加密脚本，为了提高安全性，可以使用约束模式 2 来加密
Python 应用程序，例如::

    pyarmor obfuscate --restrict 2 foo.py

约束模式 2 不允许从没有加密的脚本中导入加密的脚本，从而更高程度的保护了加密脚本
的安全性。

如果对安全性要求更高，可以使用约束模式 3 ，例如::

    pyarmor obfuscate --restrict 3 foo.py

约束模式 3 会检查每一个加密函数的调用，不允许加密的函数被非加密的脚本调用。

上述两种模式并不适用于 Python 包的加密，因为对于 Python 包来说，必须允许加密的脚
本被其他非加密的脚本导入和调用。为了提高 Python 包的安全性，可以采取下面的方案:

* 把需要供外部使用的函数集中到包的某一个或者几个文件
* 使用约束模式 1 加密这些需要被外部调用的文件
* 使用约束模式 4 加密其他的脚本文件

例如::

    cd /path/to/mypkg
    pyarmor obfuscate --exact __init__.py exported_func.py
    pyarmor obfuscate --restrict 4 --recursive \
            --exclude __init__.py --exclude exported_func.py .

关于约束模式的详细说明，请参考 :ref:`约束模式`

检查被调用的函数是否经过加密
----------------------------

假设主脚本为 `main.py`, 需要调用模块 `foo.py` 里面的方法 `connect`, 并且需要传递
敏感数据作为参数。两个脚本都已经被加密，但是用户可以自己写一个 `foo.py` 来代替加
密的 `foo.py` ，例如::

    def connect(username, password):
        print('password is %s', password)

然后调用加密的主脚本 `main.py` ，虽然功能不能正常完成，但是敏感数据却被泄露。

为了避免这种情况发生，需要在主脚本里面检查 `foo.py` 必须也是被加密的脚本。目前的
解决方案是在脚本 `main.py` 里面增加修饰函数 `assert_armored`，例如::

    import foo

    # 新增的修饰函数
    def assert_armored(*names):
        def wrapper(func):
            def _execute(*args, **kwargs):
                for s in names:
                    # For Python2
                    # if not (s.func_code.co_flags & 0x20000000):
                    # For Python3
                    if not (s.__code__.co_flags & 0x20000000):
                        raise RuntimeError('Access violate')
                return func(*args, **kwargs)
            return _execute
        return wrapper

    # 使用修饰函数，把需要检查的函数名称都作为参数传递进去
    @assert_armored(foo.connect, foo.connect2)
    def start_server():
        foo.connect('root', 'root password')
        foo.connect2('user', 'user password')

这样在每次运行 `start_server` 之前，都会检查被调用的函数是否被加密，如果没有被加
密，直接抛出异常。

.. 定制保护代码:

.. include:: _common_definitions.txt
