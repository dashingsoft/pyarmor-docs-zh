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

如果需要使用 :ref:`超级模式` 加密脚本，可以通过 pip 选项 ``pyarmor.advanced`` 来
实现。

首先运行 `pip config <https://pip.pypa.io/en/stable/cli/pip_config/>`_ ::

    pip config set pyarmor.advanced 2

然后::

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

目前只有最基本的功能被实现，如果对这个功能有新的需求，欢迎提交 pull request 到 github

.. include:: _common_definitions.txt
