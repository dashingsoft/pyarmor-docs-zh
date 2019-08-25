.. _如何打包加密脚本:

如何打包加密脚本
================

虽然加密脚本可以无缝替换原来的脚本，但是打包的时候还是存在一个问题:

**加密之后所有的依赖包无法自动获取**

解决这个问题的基本思路是

1. 使用没有加密的脚本找到所有的依赖文件
2. 使用加密脚本替换原来的脚本
3. 添加加密脚本需要的运行辅助文件到安装包
4. 替换主脚本，因为主脚本会被编译成为可执行文件

PyArmor 需要 `PyInstaller` 来完成加密脚本的打包工作，如果没有安装的话，首先执行
下面的命令进行安装::

    pip install pyinstaller

PyArmor 提供了一个命令 :ref:`pack` 可以用来直接打包脚本，它会首先加密脚本，然后
调用 PyInstaller 打包，但是在某些情况下，可以打包会失败。这里详细描述了命令
:ref:`pack` 的内部工作原理，可以帮助定位问题所在，同时也可以作为自己直接使用
PyInstaller 打包加密脚本的使用方法。

`pyarmor pack` 命令的第一步是加密所有的脚本，保存到 ``dist/obf``::

    pyarmor obfuscate --output dist/obf hello.py

第二步是生成 `.spec` 文件，这是 `PyInstaller` 需要的，把加密脚本需要的运行辅助文
件也添加到里面::

    pyinstaller --add-data dist/obf/license.lic
                --add-data dist/obf/pytransform.key
                --add-data dist/obf/_pytransform.*
                hello.py dist/obf/hello.py

第三步是修改 `hello.spec`, 在 `Analysis` 之后插入下面的语句，主要作用是打包的时
候使用加密后的脚本，而不是原来的脚本::

    a.scripts[-1] = 'hello', r'dist/obf/hello.py', 'PYSOURCE'
    for i in range(len(a.pure)):
        if a.pure[i][1].startswith(a.pathex[0]):
            x = a.pure[i][1].replace(a.pathex[0], os.path.abspath('dist/obf'))
            if os.path.exists(x):
                if hasattr(a.pure, '_code_cache'):
                    with open(x) as f:
                        a.pure._code_cache[a.pure[i][0]] = compile(f.read(), a.pure[i][1], 'exec')
                a.pure[i] = a.pure[i][0], x, a.pure[i][2]

最后运行这个修改过的文件，生成最终的安装包::

    pyinstaller --clean -y hello.spec

.. note::

   必须要指定选项 `--clean` ，否则不会把原来的脚本替换成为加密脚本。

检查一下安装包中的脚本是否已经加密::

   # It works
   dist/hello/hello.exe

   rm dist/hello/license.lic

   # It should not work
   dist/hello/hello.exe

.. include:: _common_definitions.txt
