.. _使用实例:

使用实例
========

下面是一些使用 PyArmor 的实例。

加密并打包 PyQt 应用
--------------------

文字统计工具 easy-han 使用 PyQt 开发，主要文件如下::

    config.json

    main.py
    ui_main.py
    readers/
        __init__.py
        msexcel.py

    tests/
    vnev/py36


加密打包脚本如下::

    cd /path/to/src
    pyarmor pack -e " --name easy-han --hidden-import comtypes --add-data 'config.json;.'" \
                 -x " --exclude vnev;tests" -s "easy-han.spec" main.py

    cd dist/easy-han
    ./easy-han

使用 `-e` 传入额外的参数去运行 `PyInstaller` ，要确认 `PyInstaller` 使
用这些选项可以正常打包::

    cd /path/to/src
    pyinstaller --name easy-han --hidden-import comtypes --add-data 'config.json;.' main.py

    cd dist/easy-han
    ./easy-han

使用 `-x` 传入额外的参数去加密脚本，因为 `tests` 和 `vnev` 下面也有很
多脚本，但是这些不需要加密，所以使用 `--exclude` 选项把它们排除。要确
认可以使用这些选项可以正常加密脚本::

    cd /path/to/src
    pyarmor obfuscate --exclude vnev;tests main.py

使用 `-s` 参数主要是因为 PyInstaller 使用 `--name` 修改了安装包的名称，
默认生成的 `.spec` 文件不再是主脚本名称，所以要告诉 `pack` 命令修改后
的 `.spec` 文件名称。

.. include:: _common_definitions.txt
