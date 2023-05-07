.. highlight:: console

============================
 打包使用外部密钥的加密脚本
============================

下面说明如何加密并打包一个应用程序 ``src/myapp.py`` 并使用 :term:`外部密钥`

首先使用 PyInstaller 打包::

    $ pyinstaller myapp.py

接着加密脚本并替换生成的可执行文件，同时指定使用外部密钥::

    $ pyarmor gen --outer --pack dist/myapp/myapp myapp.py

然后生成外部密钥::

    $ pyarmor gen key -O keylist -e 30

对于打包成为单个目录的模式来说，一般存放运行密钥到运行辅助包的目录，例如::

    $ cp keylist/pyarmor.rkey dist/myapp/pyarmor_runtime_000000/

这样就可以在任何位置运行可执行文件 ``dist/myapp/myapp`` 。例如::

    $ dist/myapp/myapp

如果是打包成为单个可执行文件，一般把外部密钥和可执行文件存放在相同目录下面，并且重命名为 ``可执行文件名称.密钥名称`` 。例如::

    $ pyinstaller --onefile myapp.py
    $ pyarmor gen --outer --pack dist/myapp myapp.py
    $ pyarmor gen key -O keylist -e 30
    $ cp keylist/pyarmor.rkey dist/myapp.pyarmor.rkey

然后这样就可以在任意目录运行可执行文件，例如::

    $ dist/myapp

还有一种情况是把运行密钥存放在一个固定目录下面，然后使用环境变量来指定这个目录。例如::

    $ export PYARMOR_RKEY=/opt/pyarmor/runtime_data
    $ mkdir -p /opt/pyarmor/runtime_data
    $ cp keylist/pyarmor.rkey /opt/pyarmor/runtime_data/
    $ dist/foo

.. include:: ../_common_definitions.txt
