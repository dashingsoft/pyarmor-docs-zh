.. highlight:: console

========================
 打包加密脚本成为 Wheel
========================

测试项目的目录结构如下::

    $ tree test-project

    test-project
    ├── MANIFEST.in
    ├── pyproject.toml
    ├── setup.cfg
    └── src
        └── parent
            ├── child
            │   └── __init__.py
            └── __init__.py

文件 ``MANIFEST.in`` 的内容如下::

    recursive-include dist/parent/pyarmor_runtime_00xxxx *.so

文件 ``pyproject.toml``

.. code-block:: ini

    [build-system]
        requires = [
            "setuptools>=66.1.1",
            "wheel"
        ]
        build-backend = "setuptools.build_meta"

文件 ``setup.cfg``

.. code-block:: ini

    [metadata]
        name = parent.child
        version = attr: parent.child.VERSION

    [options]
        package_dir =
            =dist/

        packages =
            parent
            parent.child
            parent.pyarmor_runtime_00xxxx

        include_package_data = True

:file:`src/parent/__init__.py` 和 :file:`src/parent/child/__init__.py` 是相同的:

.. code-block:: python

    VERSION = '0.0.1'

首先加密包::

    $ cd test-project
    $ pyarmor gen --recursive -i src/parent

加密成功之后输出的目录结构如下::

    $ tree dist

    dist
    └── parent
        ├── child
        │   ├── __init__.py
        │   └── __pycache__
        │       └── __init__.cpython-311.pyc
        ├── __init__.py
        └── pyarmor_runtime_00xxxx
            ├── __init__.py
            └── pyarmor_runtime.so

接着就构建 wheel::

    $ python -m build --skip-dependency-check --no-isolation

但是却报错了

.. code-block:: python

    * Building sdist...
    Traceback (most recent call last):
      File "/usr/lib/python3/dist-packages/setuptools/config/expand.py", line 81, in __getattr__
        return next(
               ^^^^^
    StopIteration

    The above exception was the direct cause of the following exception:

    Traceback (most recent call last):
      File "/usr/lib/python3/dist-packages/setuptools/config/expand.py", line 191, in read_attr
        return getattr(StaticModule(module_name, spec), attr_name)

检查错误堆栈发现问题出现在 ``StaticModule`` ，通过异常中文件名称和行号，查看在包 ``setuptools`` 源代码中这个类的定义，它使用了 ``ast.parse`` 直接解析源代码获取模块的局部变量，这当然在加密脚本中行不通的。为了解决这个问题，可以直接在加密后的脚本 ``dist/parent/child/__init__.py``  中增加一行

.. code-block:: python

    from pyarmor_runtime_00xxxx import __pyarmor__
    VERSION = '0.0.1'
    ...

但是默认情况下加密脚本是不允许修改的，为了能够修改这个脚本，需要使用下面的命令进行配置::

    $ pyarmor cfg -p parent.child.__init__ restrict_module = 0
    $ pyarmor gen --recursive -i src/parent

其中选项 :option:`pyarmor cfg -p` ``parent.child.__init__`` 是只取消对单个模块的 ``parent/child/__init__.py`` 的限制，而其他模块依旧使用约束模式。

现在修改 ``dist/parent/child/__init__.py`` 之后再次运行下面的命令进行打包::

    $ python -m build --skip-dependency-check --no-isolation

**自定义运行辅助包的名称**

如果你想自定义运行辅助包的名称为 ``libruntime`` ，并且存放到子目录 ``parent.child`` 下面，你需要修改 ``MANIFEST.in``::

    recursive-include dist/parent/child/libruntime *.so

和 ``setup.cfg``::

    [options]
        ...
        packages =
            parent
            parent.child
            parent.child.libruntime
        ...

然后使用下面的配置进行加密::

    $ pyarmor cfg package_name_format "libruntime"
    $ pyarmor gen --recursive --prefix parent.child src/parent

在创建 wheel 之前不要忘记对 ``dist/parent/child/__init__.py`` 进行修改::

    $ python -m build --skip-dependency-check --no-isolation

**更进一步**

为了能够在加密完成之后自动修改 ``dist/parent/child/__init__.py`` ，你可以创建一个 :term:`加密插件` ``.pyarmor/myplugin.py``:

.. code-block:: python

    __all__ = ['VersionPlugin']

    class VersionPlugin:

        @staticmethod
        def post_build(ctx, inputs, outputs, pack):
            script = os.path.join(outputs[0], 'parent', 'child', '__init__.py')
            with open(script, 'a') as f:
                f.write("\nVERSION = '0.0.1'")

然后启用这个插件::

    $ pyarmor cfg plugins + "myplugin"

这样以后每次打包只需要运行下面的命令就可以了::

    $ pyarmor gen --recursive --prefix parent.child src/parent
    $ python -m build --skip-dependency-check --no-isolation

.. include:: ../_common_definitions.txt
