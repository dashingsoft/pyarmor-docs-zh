.. _了解加密脚本:

了解加密脚本
============

.. _全局密钥箱:

全局密钥箱
----------

全局密钥箱是存放在用户主目录的一个文件 :file:`.pyarmor_capsule.zip` 。
当执行命令 ``pyarmor obfuscate`` 如果该文件还没有存在，那么会自动创建一
个全局密钥箱，加密脚本和为加密脚本生成认证文件都需要从这里读取相关数据。

加密后的脚本
------------

和原来的脚本相比，被 PyArmor 加密后的脚本需要额外的运行辅助文件，下面是
加密后在输出目录 `dist` 下的所有文件清单::

    myscript.py
    mymodule.py

    pytransform.py
    _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
    pytransform.key
    license.lic

从 v5.7.0 开始，运行时刻的四个文件使用包的形式保存在一个单独的目录
`pytransform` 而不是和存放加密脚本放在一起，这样使得加密后的脚本目录结
构更清晰。下面是使用包之后 `dist` 下的文件清单::
  
    myscript.py
    mymodule.py
    pytransform/
        __init__.py
        _pytransform.so, or _pytransform.dll in Windows, _pytransform.dylib in MacOS
        pytransform.key
        license.lic

被加密的脚本也是一个普通的 Python 脚本，模块 `dist/mymodule.py` 加密后
会是这样::

    __pyarmor__(__name__, __file__, b'\x06\x0f...')

而主脚本 `dist/myscript.py` 被加密后则会是这样::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()

    __pyarmor__(__name__, __file__, b'\x0a\x02...')

.. _引导代码:

引导代码
--------

主脚本的前两行就是 `引导代码` ，它一般出现在主脚本中::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()

.. _运行辅助文件:

运行辅助文件
------------

除了加密脚本之外的其他文件都是 `运行辅助文件`:

* `pytransform.py`, 这是一个普通的 Python 模块
* `_pytransform.so` 或者 `_pytransform.dll` 或者 `_pytransform.dylib` 是核心的动态链接库
* `pytransform.key` 是数据文件
* `license.lic` 是加密脚本的许可文件

在客户机上运行加密脚本不需要安装 PyArmor, 但是必须要把所有的 `运行辅助
文件` 拷贝过去。

.. _加密脚本的许可文件:

加密脚本的许可文件
------------------

运行辅助文件中的 `license.lic` 作用比较特殊，它包含着对加密脚本的运行许
可信息。在加密脚本的同时会在输出目录下面生成一个默认许可文件，该文件允
许加密脚本运行在任何机器并且永不过期。

如果需要为加密脚本设置新的许可，例如设置有效期，那么需要运行命令
``pyarmor licenses`` 生成新的相应的许可文件，然后用新生成的
`license.lic` 覆盖原来的许可文件。

使用加密脚本的基本原则
----------------------

* 加密后的脚本也是一个正常的 Python 脚本，它可以无缝替换原来的脚本

* 唯一的改变是，在使用加密脚本之前，下面这两行 :ref:`引导代码` 必须被首
  先执行::

    from pytransform import pyarmor_runtime
    pyarmor_runtime()

* 运行辅助文件 `pytransform.py` 必须位于能够被 Python 导入的路径，模块
  `pytransform` 会使用 `ctypes` 装载动态库。动态库是平台相关的，所有预编
  译的动态库列表在这里 :ref:`支持的平台列表`

* `pytransform.py` 会在同目录下面搜索动态库 `_pytransform` ，具体装载过
  程可以查看函数 `pytransform._load_library` 的源代码

* 所有其他的 :ref:`运行辅助文件` 和动态库要在同一个目录下面

* 如果 :ref:`运行辅助文件` 位于其他目录，必须在 :ref:`引导代码` 中指出::

    from pytransform import pyarmor_runtime
    pyarmor_runtime('/path/to/runtime-files')

加密脚本的运行
--------------

加密脚本也是一个正常的 Python 脚本，它可以像运行普通脚本一样被运行::

    cd dist
    python myscript.py

前两行的 :ref:`引导代码` 会首先被执行:

* 从文件 `pytransform.py` 中导入 `pyarmor_runtime`
* 执行 `pyarmor_runtime` ，进行如下操作
    * 使用 `ctypes` 装载动态链接库 `_pytransform`
    * 检查许可文件 `license.lic` 的有效性
    * 添加三个内置函数 `__pyarmor__`, `__enter_armor__`, `__exit_armor__`

然后执行第三条语句:

* 调用 `__pyarmor__` 导入加密的模块
* 当每一个函数被执行的时候，调用内置函数 `__enter_armor__` 恢复被加密的函数
* 单每一个函数执行完成之后，调用 `__exit_armor__` 重新加密函数

详细的执行过程，请参考 :ref:`如何加密脚本` and :ref:`如何运行加密脚本`

运行时刻文件 `license.lic` 和 `pytransform.key` 的查找路径
----------------------------------------------------------

加密脚本会首先在当前目录查看是否存在 `license.lic` 和运行时刻文件
`pytransform.key`, 如果存在，那么使用当前目录下面的这些文件。

其次加密脚本会查看运行时刻模块 `pytransform.py` 所在的目录，看看有没有
`license.lic` 和 `pytransform.key`

如果当前目录下面存在任何一个文件，但是和加密脚本对应的运行时刻文件不匹
配，就会报下列错误::

    Invalid input packet.
    Check license failed.

两个不同类型的 `license.lic`
----------------------------

PyArmor 中有两个不同类型的 `license.lic` 文件

* 一个是 PyArmor 的许可文件，位于 PyArmor 的安装路径下面，它是由
  PyArmor 开发者生成并发布

* 一个是加密脚本的许可文件， 一般和加密脚本在相同的目录，它是由
  PyArmor 的用户使用 PyArmor 提供的命令来生成。

这两者之间的关系如下::

    license.lic of PyArmor --> .pyarmor_capsule.zip --> license.lic of Obfuscated Scripts

生成加密脚本的许可文件需要读取 :ref:`全局密钥箱` 的数据，而生成
:ref:`全局密钥箱` ，则需要读取 PyArmor 的许可文件里面的相关数据。

.. include:: _common_definitions.txt
