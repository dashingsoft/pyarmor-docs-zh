============================
 Pyarmor |version| 用户文档
============================

:版本: |release|
:主页: |Homepage|
:联系方式: |Contact|
:作者: |Author|

Pyarmor 是什么
==============

|Pyarmor| 是一个用于加密和保护 |Python| 脚本的工具。它能够在运行时刻保护|Python|
脚本代码不被泄露，设置加密后脚本的使用期限，绑定加密脚本到硬盘、网卡等硬件设备。

How the documentation is organized
==================================

|Pyarmor| has a lot of documentation. A high-level overview of how it's
organized will help you know where to look for certain things:

* :doc:`Part 1: Tutorials <part-1>` takes you by the hand through a series
  of steps to obfuscate |Python| scripts and packages. Start here if you're
  new to |Pyarmor|. Also look at the :doc:`tutorial/getting-started`

* :doc:`Part 2: How To <part-2>` guides are recipes. They guide you through
  the steps involved in addressing key problems and use-cases. They are more
  advanced than tutorials and assume some knowledge of how |Python| works.

* :doc:`Part 3: References <part-3>` guides contain key concepts, man page,
  configurations and other aspects of |Pyarmor| machinery.

* :doc:`Part 4: Topics <part-4>` guides insight into key topics and provide
  useful background information and explanation. They describe how it works and
  how to use it but assume that you have a basic understanding of key concepts.

* :doc:`Part 5: Licneses <licenses>` describes EULA of |Pyarmor|, the different
  |Pyarmor| licenses and how to purchase |Pyarmor| license.

Getting help
============

Having trouble? We'd like to help!

Try the :doc:`FAQ <questions>` – it's got answers to many common questions.

Looking for specific information? Try the :ref:`genindex`, or :ref:`the
detailed table of contents <mastertoc>`.

Not found anything? See :ref:`asking questions in github <asking questions>`.

Report bugs with Pyarmor_ in issues_

如何阅读本手册
==============

对于第一次使用 `Pyarmor`_ 的用户来说，先把 :doc:`tutorial` 全部仔细看一下，
然后在浏览一下 :doc:`usage` ，一般就可以完成对脚本的基本保护。

接下来看一下 :doc:`advanced` 下的各个标题，这里面有很多实际的应用场景。

|Pyarmor| 是一个命令行工具，它能够提供的功能都是通过不同的命令行选项来实现
的，:doc:`man` 则详细说明了 |Pyarmor| 各个命令和选项的功能和作用。

:doc:`insight-into` 则详细说明了 `Pyarmor`_ 组成和实现原理，有助于解决加密复
杂应用时候出现的问题，知其所以然才能更好的使用 `Pyarmor`_ 。

如果加密脚本的性能不能满足需要，那就需要看看 :doc:`performance` ，看看不同的
选项是如何影响性能，以及不同性能下面的安全性。

:doc:`license` 描述了 `Pyarmor`_ 的最终用户使用协议， `Pyarmor`_ 的许可模式，
各种许可模式的功能和限制，如何购买许可证以及关于许可证的一些常见问题。

:doc:`questions` 的开始部分详细说明了对于常见问题的通用解决方法，大部分的用
户问题按照通用方法的步骤，都可以得到解决。后面的章节则分类列出了用户经常提问
的一些问题和解决方案，使用过程中出现问题来这里通常是最快的解决方式。

目录
====

.. toctree::
   :numbered:
   :maxdepth: 3
   :name: mastertoc

   part-1
   part-2
   part-3
   part-4
   licenses
   questions

索引表
======

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

.. include:: _common_definitions.txt
