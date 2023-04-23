==========
 错误消息
==========

.. highlight:: none

.. program:: pyarmor gen

这里列出了加密时候和运行加密脚本的时候的错误信息列表，并且给出了导致错误发生的可能原因以及解决方案。

如果错误信息没有在这里找到，那么一般情况下这种错误不是 Pyarmor 引起的，可能是因为系统环境配置不正确，缺失系统包等原因造成的。这部分原因不需要 Pyarmor 进行任何修改，只要把环境配置正确，安装必要的包等就可以解决。对于这种类型的问题，请直接在百度或者其他任何搜索引擎，网站和论坛等查找解决方案。

加密时候的错误消息
==================

表-1 列出的是运行命令 :command:`pyarmor` 时候常见的错误信息，部分可能的原因和解决方案

.. list-table:: 表-1. 加密时候错误信息表
   :name: pyarmor errors
   :width: 100
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - out of license
     - 使用试用版或者当前许可证没有的功能

       解决方案请参考 :doc:`../licenses`
   * - not machine id
     - 当前设备的硬件信息发生了改变可能会造成这个错误

       重现在当前设备注册 Pyarmor 可以解决这个问题
   * - query machine id failed
     - 如果无法获取到当前设备的相关硬件信息，会报这个错误
   * - relative import "%s" overflow
     - 尝试直接加密一个脚本，但是脚本里面使用了相对导入的语句

       解决方案: 直接加密脚本所在的包 (目录)，而不是单独加密一个文件

表-1.1 列出一般发生在注册 Pyarmor 时候发生的错误信息

.. list-table:: 表-1.1 注册时候错误信息表
   :name: register errors
   :width: 100
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - HTTP Error 401: Unauthorized
     - 在新版本的许可证下使用 pyarmor-7 命令

       没有解决方案，pyarmor-7 只支持老版本的许可证
   * - HTTP Error 503: Service Temporarily Unavailable
     - 在 1 分钟之内使用了多次注册命令

       许可证服务器一分钟之内至多允许同一个 IP 的三次请求，过多的请求将返回 503 错误
   * - unknown license type OLD
     - 在 Pyarmor 8 中使用老版本的许可证

       请参阅这里的升级说明 :doc:`../licenses`

       解决方案：使用命令 ``pyarmor-7``
   * - This code has been used too many times
     -

运行加密脚本的错误信息
======================

这里列出的是运行加密脚本的时候常见的错误信息，部分可能的原因和解决方案。

表-2 里面的错误信息都是 Pyarmor 报告的

.. list-table:: 表-2. 运行加密脚本的错误信息表（Pyarmor）
   :name: runtime errors
   :width: 100
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - error code out of range
     -
   * - this license key is expired
     -
   * - this license key is not for this machine
     -
   * - missing license key to run the script
     -
   * - unauthorized use of script
     -
   * - this Python version is not supported
     -
   * - the script doesn't work in this system
     -
   * - the format of obfuscated script is incorrect
     - 可能的原因:

       1. 加密脚本是由其他不兼容的 Pyarmor 生成的，请使用最新的 Pyarmor 进行加密
       2. 无法获得运行辅助包所在的路径，也会报这个错误
   * - the format of obfuscated function is incorrect
     -
   * - RuntimeError: Resource temporarily unavailable
     - 设置了加密脚本的有效期，但是无法访问时间服务器导致的问题

       这个问题无法解决，要么使用本地时间，要么用户使用 :term:`脚本补丁` 自己进行校验

表-2.1 中的错误信息都是 Python 解释器报告的，通常情况下，这种错误不是 Pyarmor 造成的。

解决这里的问题只需要参考 Python 的文档，把加密脚本看作是普通脚本，就可以解决问题，也可以直接在百度或者 Python 相关的论坛网站找到答案。

.. list-table:: 表-2.1 运行加密脚本的错误信息表（Python）
   :name: other runtime errors
   :width: 100
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - ImportError: attempted relative import with no known parent package
     - 1. ``from .pyarmor_runtime_000000 import __pyarmor__``

       解决方案：不要使用 :option:`-i` 或者 :option:`--prefix` 去加密脚本

.. include:: ../_common_definitions.txt
