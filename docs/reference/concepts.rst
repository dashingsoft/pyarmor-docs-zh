==========
 概念定义
==========

.. glossary::

  BCC 模式

      一种加密的方法，可以把脚本中的函数转换成为 :term:`C` 函数，然后使用优化选项编译生成机器指令。使用这种方式加密的函数是不可逆的，无法恢复成为原来的 Python 函数。

  本地配置

      一般在当前目录下会创建一个目录 ``.pyarmor`` ，用来存放本地配置，默认本地配置的文件是 ``.pyarmor/config``

      相同选项的本地配置会覆盖 :term:`全局配置`

      参考 :ref:`pyarmor cfg`

  C

      一种最古老的编程语言，现在依然生命力旺盛。

  加密插件

      一个 :term:`Python` 脚本， 在加密过程被调用，可以对输出的文件进行一些额外的操作。

      插件的类型有三种：

      - KeyPlugin
      - BuildPlugin
      - RuntimePlugin

  根目录

      用来存放 Pyarmor 注册信息，全局配置文件等的路径，默认情况下是当前登陆用户的根目录下面的子目录 :file:`~/.pyarmor`

      使用 sudo 命令可能会改变 Pyarmor 的根目录

  激活文件

      一个文本文件，购买任何 :term:`Pyarmor 许可证` 之后，都会有一个相应的激活文件发送到注册邮箱。

      激活文件主要用于第一次注册 :term:`Pyarmor 许可证` ，激活文件一旦完成初始登记，就无法在继续使用。

      初始登记成功会同时生成相应的 :term:`注册文件` ，后面在任何设备上进行注册都需要使用 :term:`注册文件` 。

  JIT

      JUST-IN-TIME 的缩写，是一种在运行时刻生成机器指令的技术，可以有效防止静态反编译工具对代码进行分析

  开发机器

      是指安装和运行 Pyarmor 的设备，在开发机器上对脚本进行加密

      不是所有的平台都可以运行 Pyarmor，所有支持的运行环境请参考 :doc:`environments`

  客户设备

      是指运行加密脚本的设备

      客户设备上面不需要安装 Pyarmor

  扩展模块

      一个使用 :term:`C` 或者 C++ 语言编写的 :term:`Python 模块`

  模块私有配置

      每一个加密模块可以有自己的私有配置，一般存放在 :term:`本地配置` 的目录下面，和模块同名的 ``module.ruler`` 文件

      相同选项的私有配置会覆盖 :term:`本地配置`

  Pyarmor

      Pyarmor 是一个用来加密 Python 脚本的工具。

      Pyarmor 的组成部分

      - :term:`Pyarmor 项目`
      - :term:`pyarmor 包`

  Pyarmor 包

      一个 :term:`Python 包` ，它包含下列包

      * :mod:`pyarmor`
      * :mod:`pyarmor.cli`
      * :mod:`pyarmor.cli.core`
      * :mod:`pyarmor.cli.runtime`

  Pyarmor 基础版许可证

      一种 :term:`Pyarmor 许可证` 类型

  Pyarmor 集团版许可证

      一种 :term:`Pyarmor 许可证` 类型

  Pyarmor 用户

      使用 :term:`Pyarmor` 的组织机构或者开发人员

  Pyarmor 项目

      项目文件存放在 |Home|

      这里有 Pyarmor 开源部分的代码， 提交的 `问题报告`_ 和最新的文档。

  Pyarmor 许可证

      由 Pyarmor 开发团队颁发，用于解锁 Pyarmor 试用版本中功能限制

      请参考 :doc:`Pyarmor 许可模式和许可证 <../licenses>`

  Pyarmor 专家版许可证

      一种 :term:`Pyarmor 许可证` 类型

  Python

      一种编程语言，官网地址 Python_

  Python 脚本

      一个包含 Python 源代码的文件

      外部概念 https://docs.python.org/3.11/glossary.html#term-module

  Python 模块

      要么是一个 :term:`Python 脚本` ，要么是一个 :term:`扩展模块`

  Python 包

      外部概念 https://docs.python.org/3.11/glossary.html#term-package

  全局配置

      存放 Pyarmor 运行配置的全局文件，全局配置文件必须存放在 Pyarmor 的 :term:`根目录` ，默认的文件名称是 ``~/.pyarmor/config/global``

      参考 :ref:`pyarmor cfg`

  RFT 模式

      一种不可逆的加密模式，可以重命名 Python 脚本中的函数，类，方法，变量和参数

  外部密钥

      一个用于存储 :term:`运行密钥` 的文件，通常的名称为 ``pyarmor.rkey``

      外部密钥必须存放在下面的任何一个路径，才能被发现:

      - :term:`运行辅助包` 所在的路径
      - 环境变量 :envvar:`PYARMOR_RKEY` 指定的路径
      - :attr:`sys._MEIPASS` 指定的路径
      - 当前路径

  运行插件

      一个 Python 脚本，可以被嵌入到加密脚本中，执行一些额外的检查。

  运行辅助包

      一个 :term:`Python 包` ，名称一般为 ``pyarmor_runtime_xxxxxx``

      一般生成加密脚本的同时，也会生成相应的运行辅助包，运行加密脚本需要有相应的运行辅助包。

  运行辅助文件

      是指运行加密脚本需要的所有其他文件

      通常情况下它等价于 :term:`运行辅助包` ，如果使用了 :term:`外部密钥` ，那么也包含外部密钥

  运行密钥

      保存加密脚本的运行设置和相关约束限制，包括脚本的有效期，脚本绑定的设备信息，也包括控制加密脚本行为的其他标志和设置。

      通常情况下运行密钥被嵌入到 :term:`运行辅助包` 里面，但是也可以是一个独立文件形式的 :term:`外部密钥`

  运行平台

      Pyarmor 定义的标准名称，用来标示运行 Pyarmor 和加密脚本的操作系统和 CPU 架构

      下面列出了所有定义的运行平台:

        * Windows
            - windows.x86_64
            - windows.x86
        * Many Linuxs
            - linux.x86_64
            - linux.x86
            - linux.aarch64
            - linux.armv7
        * Apple Intel and Silicon
            - darwin.x86_64
            - darwin.aarch64 or darwin.arm64

  注册文件

      一个 ``.zip`` 格式的压缩文件，主要用于除了初始登记之后的所有注册。

      第一次初始登记使用 :term:`激活文件` ，激活成功之后生成注册文件，之后在任何设备上都要使用注册文件进行注册。


.. module:: pyarmor
    :synopsis: 一个用来加密 Python 脚本的命令行工具

.. module:: pyarmor.cli
.. module:: pyarmor.cli.core
    :synopsis: 一个平台相关的二进制 Wheel 包，提供 Pyarmor 需要的核心扩展模块

.. module:: pyarmor.cli.runtime
    :synopsis: 支持 Pyarmor 跨平台加密的包，提供跨平台所需要的预编译动态库

.. include:: ../_common_definitions.txt
