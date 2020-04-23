.. _使用 PyArmor:


使用 PyArmor
============

命令 ``pyarmor`` 的基本语法为:

    ``pyarmor`` [command] [options]

加密脚本
--------

命令 ``obfuscate`` 用来加密脚本。最常用的一种情况是切换到脚本 `myscript.py` 所在
的路径，然后执行::

    pyarmor obfuscate myscript.py

PyArmor 会加密 :file:`myscript.py` 和相同目录下面的所有 :file:`*.py` 文件:

* 在用户根目录下面创建 :file:`.pyarmor_capsule.zip` （仅当不存在的时候创建）
* 创建输出子目录 :file:`dist`
* 生成加密的主脚本 :file:`myscript.py` 保存在输出目录 :file:`dist`
* 加密相同目录下其他所有 :file:`*.py` 文件，保存到输出目录 :file:`dist`
* 生成运行加密脚本所需要的全部辅助文件，保存到输出目录 :file:`dist`

输出目录 :file:`dist` 包含运行加密脚本所需要的全部文件::

    dist/
        myscript.py
        pytransform
            __init__.py
            _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
            pytransform.key
            license.lic

除了加密脚本之外，额外的那个目录 `pytransform` 叫做 :ref:`运行辅助包` ，它是运行
加密脚本不可缺少的。

通常情况下第一个脚本叫做主脚本，它加密后的内容如下::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()
    __pyarmor__(__name__, __file__, b'\x06\x0f...')

其中前两行是 :ref:`引导代码`, 它们只在主脚本出现，并且只能被运行一次。对于其他所
有加密脚本，只有这样一行::

    __pyarmor__(__name__, __file__, b'\x0a\x02...')

运行加密脚本::

    cd dist
    python myscript.py

默认情况下，只有和主脚本相同目录的其他 :file:`*.py` 会被同时加密。如果想递归加密
子目录下的所有 :file:`*.py` 文件，使用下面的命令::

    pyarmor obfuscate --recursive myscript.py

发布加密的脚本
--------------

发布加密脚本给客户只需要把输出路径 `dist` 的所有文件拷贝过去即可。需要
注意的是 :ref:`运行辅助包` 必须也拷贝到运行环境，否则无法运行加密脚本。

:ref:`运行辅助包` 并不是必须和加密脚本在一起，也可以拷贝到运行环境的任
意的 Python 路径下面，只要能正确 `import pytransform` 就可以。

关于加密脚本的安全性的说明，参考 :ref:`PyArmor 的安全性`

.. note::

   运行加密脚本不需要安装 PyArmor，没有必要在运行环境里面安装 PyArmor

生成新的许可文件
----------------

使用命令 :ref:`licenses` 为加密脚本生成新的许可文件 :file:`license.lic`

加密脚本的同时会在输出目录下面生成一个默认许可文件 :file:`dist/license.lic` ，它
允许加密脚本运行在任何设备上并且永不过期。

如果需要设置加密脚本的使用期限或者限制脚本在特定的机器使用，需要生成新的许可文件，
并覆盖默认许可文件。

例如::

    pyarmor licenses --expired 2019-01-01 code-001

执行这条命令 PyArmor 会生成一个带有效期的认证文件:

* 从 :file:`.pyarmor_capsule.zip` 读取相关数据
* 创建 :file:`license.lic` ，保存在 ``licenses/code-001``
* 创建 :file:`license.lic.txt` ，保存在 ``licenses/code-001``

然后，使用新生成的许可文件覆盖默认的许可文件::

    cp licenses/code-001/license.lic dist/pytransform/

这样，加密脚本在2019年1月1日之后就无法在运行了。

如果想绑定加密脚本到固定机器上，首先在该机器上面运行下面的命令获取硬件信息::

    pyarmor hdinfo

然后在生成绑定到固定机器的许可文件::

    pyarmor licenses --bind-disk "100304PBN2081SF3NJ5T" --bind-mac "20:c1:d2:2f:a0:96" code-002

同样，覆盖默认许可文件，这样加密脚本就只能在指定机器上运行::

    cp licenses/code-002/license.lic dist/pytransform/

    cd dist/
    python myscript.py

.. note::

   在 v5.7.0 之前，默认许可文件 ``license.lic`` 没有在 ``dist/pytransform`` ，而是在 ``dist``

扩展其他认证方式
----------------

除了上述认证方式之外，还可以在 Python 脚本中增加其他任何认证代码，因为加密的脚本
对于客户来说就是黑盒子。当需要扩展认证方式的时候，那么需要了解 :ref:`运行时刻模
块 pytransform` 提供的相关功能。

直接在脚本中添加认证代码可能会影响调试，并且这些代码有时候必须在加密环境下才能运
行。所以推荐的方式是 :ref:`使用插件扩展认证方式` 。插件是一个普通的脚本，等价于
把认证代码单独提取出来，然后在加密的时候注入到被加密的脚本中去，这样不需要对原来
的脚本进行任何修改。更多详细的内容参考 :ref:`如何处理插件` 。

这里有一些插件的例子可以参考

https://github.com/dashingsoft/pyarmor/tree/master/plugins


加密单个模块
------------

如果只需要单独加密一个模块，使用选项 ``--exact``::

    pyarmor obfuscate --exact foo.py

这样，就只有 :file:`foo.py` 被加密，导入这个加密模块::

    cd dist
    python -c "import foo"

加密整个 Python 包
------------------

加密整个 Python 包使用下面的命令::

    pyarmor obfuscate --recursive --output dist/mypkg mykpg/__init__.py

使用这个加密的包::

    cd dist
    python -c "import mypkg"

打包加密脚本
------------

命令 ``pack`` 用来打包并加密脚本

首先需要安装 `PyInstaller`::

    pip install pyinstaller

然后运行下面的命令::

    pyarmor pack myscript.py

PyArmor 通过以下的步骤将所有需要的文件打包成为一个独立可运行的安装包:

* 执行 ``pyarmor obfuscate`` 加密脚本 :file:`myscript.py` 和同目录下的所有其他脚本
* 执行 ``pyinstaller myscipt.py`` 创建 :file:`myscript.spec`
* 修改 :file:`myscript.spec`, 把原来的脚本替换成为加密后的脚本
* 再次执行 ``pyinstaller myscript.spec`` ，生成最终的安装包

输出的文件在目录 ``dist/myscript`` ，这里面包含了脱离 Python 环境可以运行的所有文件。

运行打包好的可执行文件::

    dist/myscript/myscript

检查脚本是否加密。如果加密，下面的第二条命令应该执行失败::

    rm dist/myscript/license.lic
    dist/myscript/myscript

为加密脚本设置有效期::

    pyarmor licenses --expired 2019-01-01 code-003
    cp licenses/code-003/license.lic dist/myscript

    dist/myscript/myscript

对于复杂的应用，例如传入额外的参数到 `Pyinstaller` 等，请参考命令 :ref:`pack` 和
:ref:`如何打包加密脚本`

进一步提高加密脚本的安全性
--------------------------

下面这些 `PyArmor`_ 的特征可以进一步提高加密脚本的安全性:

1. 如果平台支持启用 :ref:`高级模式`
2. 使用相应 :ref:`约束模式` 来加密脚本
3. :ref:`使用工程` 来配置更高安全级别的加密模式 `obf_mode=2`
4. :ref:`使用插件来进一步提高安全性`

关于加密脚本的安全性的说明，请参考 :ref:`PyArmor 的安全性`

.. include:: _common_definitions.txt
