==========
 错误消息
==========

.. highlight:: none

.. program:: pyarmor gen

这里列出了加密时候和运行加密脚本的时候的错误信息列表，并且给出了导致错误发生的可能原因以及解决方案。

如果错误信息没有在这里找到，那么一般情况下这种错误不是 Pyarmor 引起的，可能是因为系统环境配置不正确，缺失系统包等原因造成的。这部分原因不需要 Pyarmor 进行任何修改，只要把环境配置正确，安装必要的包等就可以解决。对于这种类型的问题，请直接在百度或者其他任何搜索引擎，网站和论坛等查找解决方案。

加密时候的错误消息
==================

**加密常见错误信息**

下表列出的是运行命令 :command:`pyarmor` 时候常见的错误信息，部分可能的原因和解决方案

.. list-table:: 表-1. 加密时候错误信息表
   :name: pyarmor errors
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

**注册错误信息**

下表列出一般发生在注册 Pyarmor 时候的错误信息

.. list-table:: 表-1.1 注册时候错误信息表
   :name: register errors
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - HTTP Error 400: Bad Request
     - 请升级到 Pyarmor 8.2+，以显示正确的错误信息，然后采取相应的解决方案
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
     - 如果是在 CI/Docker 中使用 Pyarmor，请通过该订单的注册邮箱发送订单信息到 pyarmor@163.com 以解锁该订单
   * - update license token failed
     - 如果在 1 分钟之内运行注册命令超过 3 次，请等上 5 分钟之后在进行测试

       如何还存在问题，在浏览器打开网页 `http://pyarmor.dashingsoft.com//api/auth2/`

       如果页面返回 `NO:missing parameters` ，这说明网络没有问题，Pyarmor 许可证服务器也没有问题

       如果使用的是 v8.5.3 之前的版本，首先升级到 v8.5.3+，然后使用下面的命令检查 Python 是否可以访问服务器::

         $ python
         >>> from urllib.request import urlopen
         >>> res = urlopen('http://pyarmor.dashingsoft.com//api/auth2/')
         >>> print(res.read())
         b'NO:missing parameter'

       如果返回其他或者抛出异常，那么很有可能是防火墙设置问题，不允许 Python 访问网络，请参考防火墙文档正确进行配置


运行加密脚本的错误信息
======================

**Pyarmor 报告的错误信息**

这里列出的是运行加密脚本的时候 Pyarmor 报出的错误信息，部分可能的原因和解决方案。

如果该消息有错误代码，那么用户可以使用消息代码对这个消息进行国际化和本地化处理。没有错误代码的消息无法进行定制。

.. list-table:: 表-2. 运行加密脚本的错误信息表（Pyarmor）
   :name: runtime errors
   :header-rows: 1

   * - 错误代码
     - 错误信息
     - 原因和解决方案
   * -
     - error code out of range
     - Internal error
   * - error_1
     - this license key is expired
     -
   * - error_2
     - this license key is not for this machine
     -
   * - error_3
     - missing license key to run the script
     -
   * - error_4
     - unauthorized use of script
     -
   * - error_5
     - this Python version is not supported
     -
   * - error_6
     - the script doesn't work in this system
     -
   * - error_7
     - the format of obfuscated script is incorrect
     - 可能的原因:

       1. 加密脚本是由其他不兼容的 Pyarmor 生成的，请使用最新的 Pyarmor 进行加密
       2. 无法获得运行辅助包所在的路径，也会报这个错误
   * - error_8
     - the format of obfuscated function is incorrect
     -
   * -
     - RuntimeError: Resource temporarily unavailable
     - 设置了加密脚本的有效期，但是无法访问时间服务器导致的问题

       解决方案：

       1. 使用本地时间
       2. 用户使用 :term:`脚本补丁` 自己进行校验
       3. 升级到 Pyarmor 8.4.4+ ，并使用 HTTP 服务器进行检验，需要通过命令设置一个有效的 HTTP 服务器。例如 `pyarmor cfg nts=http://your.http-server.com/api/v2/`
   * -
     - Protection Exception
     - 如果使用了选项 :option:`--assert-call` 或者 :option:`assert-import` ，那么参考 :doc:`../tutorial/advanced` 中的 `过滤需要保护的函数和模块` ，根据错误堆栈的信息，忽略出错的函数或者模块。


**Python 报告的错误信息**

通常情况下，这种错误不是 Pyarmor 造成的。

解决这里的问题只需要参考 Python 的文档，把加密脚本看作是普通脚本，就可以解决问题，也可以直接在百度或者 Python 相关的论坛网站找到答案。

.. list-table:: 表-2.1 运行加密脚本的错误信息表（Python）
   :name: other runtime errors
   :header-rows: 1

   * - 错误信息
     - 原因和解决方案
   * - ImportError: attempted relative import with no known parent package
     - 1. ``from .pyarmor_runtime_000000 import __pyarmor__``

       解决方案：不要使用 :option:`-i` 或者 :option:`--prefix` 去加密脚本

.. include:: ../_common_definitions.txt
