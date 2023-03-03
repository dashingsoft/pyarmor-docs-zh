====================
 加密时候的错误消息
====================

这里列出的是运行命令 :command:`pyarmor` 时候常见的错误信息，部分可能的原因和解决方案

* out of license

  使用试用版或者当前许可证没有的功能

  解决方案请参考 :doc:`../licenses`

========================
 运行加密脚本的错误信息
========================

这里列出的是运行加密脚本的时候常见的错误信息，部分可能的原因和解决方案

* error code out of range

* this license key is expired

* this license key is not for this machine

* missing license key to run the script

* unauthorized use of script

* this Python version is not supported

* the script doesn't work in this system

* the format of obfuscated script is incorrect

  可能的原因:

  - 加密脚本是由其他不兼容的 Pyarmor 生成的，请使用最新的 Pyarmor 进行加密
  - 无法获得运行辅助包所在的路径，也会报这个错误


* the foramt of obfuscated function is incorrect

.. include:: ../_common_definitions.txt
