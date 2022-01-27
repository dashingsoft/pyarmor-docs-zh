.. _build pyarmored wheel:

构建加密的 Wheel
================

很多最新的 Python 包都包含一个 `pyproject.toml
<https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/>`_
文件，这个文件可以被 pip 用来进行打包 Python 包。

从 v7.2.0 开始，pyarmor 也可以使用这个文件作为 PEP 517 后端去构建一个加密的 Wheel。

下面是一个包的结构示例::

  mypkg/
      setup.py
      pyproject.toml
      src/
          __init__.py
          ...

文件 `pyproject.toml` 的内容如下::

    [build-system]
    requires = ["setuptools", "wheel"]
    build-backend = "setuptools.build_meta"

首先要确保使用后端 ``setuptools.build_meta`` 可以成功打包，生成 Wheel 文件。如果
下面的命令执行不成功，请学习和打包相关的基础知识，确保能够生成一个正确的 Wheel::

    cd mypkg/
    pip wheel .

现在修改文件 ``pyproject.toml`` ，把后端设置为 ``pyarmor.build_meta``::

    [build-system]
    requires = ["setuptools", "wheel", "pyarmor>=7.2.0"]
    build-backend = "pyarmor.build_meta"

这样使用下面的命令就可以生成一个加密的 Wheel::

    cd mypkg/
    pip wheel .

如果需要使用 :ref:`超级模式` 加密脚本，那么通过环境变量 ``PIP_PYARMOR_OPTIONS``
传递相应的加密参数。例如::

    cd mypkg/

    # In Windows
    set PIP_PYARMOR_OPTIONS=--advanced 2
    pip wheel .

    # In Linux or MacOs
    PIP_PYARMOR_OPTIONS="--advanced 2" pip wheel .

从 v7.2.4 开始，也可以通过 pip 选项 ``pyarmor.advanced`` 来实现，首先运行 `pip
config <https://pip.pypa.io/en/stable/cli/pip_config/>`_ ::

    pip config set pyarmor.advanced 2

然后运行构建命令::

    cd mypkg/
    pip wheel .

使用下面的命令删除这个选项::

    pip config unset pyarmor.advanced

内部工作原理
------------

因为 pyarmor 加密后的脚本可以无缝替换原来的 Python 脚本，所以 pyarmor 只是做了下
面的工作

1. 调用 ``setuptools.build_meta`` 生成一个正常的 Wheel
2. 解压 Wheel 到临时目录
3. 加密临时目录中所有 .py 脚本
4. 把加密脚本的 :ref:`运行辅助包` 中所有文件增加到 Wheel 的记录文件 ``RECORD``
5. 把临时目录中文件重新生成一个 Wheel ，这就是最终加密后的 Wheel

关于实现细节，请参考 `pyarmor/build_meta.py
<https://github.com/dashingsoft/pyarmor/blob/master/src/build_meta.py#L86>`_ 中
的函数 ``bdist_wheel``

这是一个非常简单的脚本，只有最基本的功能被实现，如果运行出错，可以使用人工方式或
者写一个脚本来实现:

1. 使用没有加密前的包直接创建一个 `.whl` 包
2. 解压创建好的包到临时目录，参考使用下面的命令::
     python3 -m wheel unpack --dest /path/to/temp xxx.whl
3. 使用 `pyarmor obfuscate` 命令加密脚本，然后把加密脚本拷贝到解压目录中，覆盖原
   理的同名脚本
4. 增加 pyarmor 的运行辅助文件到这个文件 ``RECORD`` ， 直接在解压目录下面搜索这
   个文件名，然后参考里面存在的文件格式把运行辅助文件都添加进去
5. 最后生成包含加密脚本的 `.whl` 包::
     python3 -m wheel pack /path/to/temp

如果对这个功能有新的需求，欢迎提交 pull request 到 github

.. important::

    这个功能是一个辅助功能，不提供更多的技术支持。如果你不知道如何构建一个 Wheel
    包，尤其是包含二进制文件（扩展模块）和数据文件的 Python 包，请首先参考相关文
    档学习掌握。掌握了这个之后， 参考 :ref:`使用加密脚本的基本原则` ，把加密脚本
    当作正常的 Python 包进行处理即可。

.. include:: _common_definitions.txt
