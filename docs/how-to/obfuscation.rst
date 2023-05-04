.. highlight:: console

..
  ========================
   Obfuscating django app
  ========================

  TODO:

  ===========================
   Building obfuscated wheel
  ===========================

  TODO:

  ============================
   打包使用外部密钥的加密脚本
  ============================

  编写中...

==========================
 打包脚本的时候保护系统库
==========================

.. versionadded:: 8.2

Pyarmor 对使用 PyInstaller 打包进行发布的方式提供了特别的保护，这种模式下面可以对系统库进行加密和保护，确保其不能被替换。下面是必须使用的选项::

    $ pyarmor cfg assert.call:auto_mode="or" assert.call:includes = "*"
    $ pyarmor cfg assert.call:auto_mode="or" assert.call:includes = "*"

    $ pyarmor gen --assert-call --assert-import --restrict --pack dist/foo/foo foo.py

其他选项可以根据需要进行配置。

.. seealso:: :doc:`protection`

.. include:: ../_common_definitions.txt
