==========
 常见问题
==========

.. _asking questions:

如何在 Github 上提问
====================

Pyarmor 是一个命令行工具，提供了丰富的选项来应对不同的情况。对于简单的应用，使用默认选项就可以工作，不需要学习如何使用 Pyarmor。但是对于一些高级的功能，或者比较复杂的应用，尤其是需要使用大型的第三方包来调用加密脚本，用户需要一定的时间去学习和使用各种选项，并组合这些选项去解决加密过程中问题。

当加密脚本出现问题的时候，这不一定是 Pyarmor 的 Bug，很可能是这个脚本使用默认的选项和配置无法工作。这种情况下需要分析问题并找到 Pyarmor 解决这个问题的选项和设置，这就需要用户参考 Pyarmor 的命令手册去找到合适的选项。

**Pyarmor Team 不会告诉用户需要使用什么选项去解决加密过程中遇到的问题，用户需要自己学习 Pyarmor 的各个选项如何使用，并根据自己的需要选择正确的选项**

Pyarmor 有完整的文档系统，所有的功能和选项都已经在文档中描述，Pyarmor Team 能告诉你的，这些文档中都已经有说明。

在提问之前，请首先尝试下面的解决方案，这应该能够解决大部分的问题，并且避免了提交重复的问题:

- 如果使用的是命令 :command:`pyarmor-7` 或者 Pyarmor < 8.0，请参阅 `Pyarmor 7.x 在线文档`_

- 浏览一下 :ref:`文档总目录 <mastertoc>`

- 如果还没有读过 :doc:`tutorial/getting-started` ，首先把入门教程读一下，特别是最后面的一章，文档的组织结构和内容说明

- 参阅 :doc:`常见错误信息 <reference/errors>` 里面的错误信息和解决方案

- 如果是使用 pack 方面的问题，请参阅 :doc:`topic/repack`

- 如果是使用 :term:`RFT 模式` 的问题，请参阅 :ref:`using rftmode`

- 如果是使用 :term:`BCC 模式` 的问题，请参阅 :ref:`using bccmode`

- 如果是第三库无法正常调用加密脚本，请参阅 :doc:`how-to/third-party`

- 如果是和安全性能相关的问题，请参阅 :doc:`topic/performance`

- 浏览一下本页后面的内容

- 运行没有加密的脚本，确保没有加密之前的脚本能够正确的运行

- 如果加密的是复杂的应用，请首先测试一个简单的脚本，确认它使用同样的加密选项能够工作

- 启用调试模式，跟踪日志，在控制台输出更多信息，仔细检查日志信息，确认问题所在

- 如果使用的不是最新版本的 Pyarmor，请升级到最新版本

- 搜索一下 `问题报告`_

- 搜索一下 `讨论区`_

如果你不想花费时间去阅读文档，请在 `讨论区`_ 提问，但是 Pyarmor Team 并不保证一定回答这部分的所有问题。

如果你发现 Pyarmor 中存在 Bug，请提交到 `问题报告`_

Pyarmor Team 欢迎真正的 Bug 报告，并且会尽快解决这些 Bug。报告 Bug 请提供必要的信息以便 Pyarmor Team 能够重现问题，这样有助于快速解决问题，无法被 Pyarmor Team 重现的问题解决起来需要更长的周期。运行 :ref:`pyarmor gen` 命令的时候，控制台的前四行包含加密使用的选项，Pyarmor 的版本信息，Python 的版本信息，以及当前的运行平台，请务必以文本的形式拷贝这四行到 Bug 报告中。对于无法重现的 Bug 报告，Pyarmor Team 会把它转入到讨论区，以后可以重现的时候，在转回到 Bug 并继续处理。为了方便处理 Bug，请尽量不要使用截图，而是直接拷贝文本。例如，一般报告应该以这样的方式开始::

   **加密使用的选项和命令**
   ```
   $ pyarmor cfg wrap_mode 1
   $ pyarmor gen --assert-call foo.py

   INFO     Python 3.9.0
   INFO     Pyarmor 8.1.6 (pro), 005068, boot armor
   INFO     Platform darwin.x86_64
   ```

热点问题
========

**看到网上有破解 Pyarmor 的工具？请问 Pyarmor 能够防止这些破解工具吗？**

  Pyarmor 一般不会关注和了解这些破解工具，而是致力于研究 CPython 的源代码，以改进算法来提高安全性，所以没有办法直接回答这类问题。

  可以确定的是，Pyarmor 提供了多种不可逆的加密模式，从原理上来说，是不可能把加密脚本恢复成为原来的脚本。

  请参考 :doc:`how-to/security` ，使用你可用的最高安全选项去加密一个简单的参考脚本，例如

  .. code-block:: python

     import sys

     def fib(n):
         a, b = 0, 1
         while a < n:
             print(a, end=' ')
             a, b = b, a+b
         print()

     print('python version:', sys.version_info[:2])
     print('this is fib(10)', fib(10))

  然后尝试使用网上所说的方法和工具进行破解。

  如果能够被破解，请把Python 版本，Pyarmor 的版本，运行的平台，加密使用的选项，参考脚本以及破解的方法发送到 |Contact|

  请不要在 Pyarmor 上发布任何破解工具的链接。

  Pyarmor 的核心功能是防止加密脚本被还原，对使用各种逆向工程方法的内存拷贝，直接修改内存绕过加密脚本的约束设置并没有特别的保护。关于如何对运行数据进行保护的解决方案，请参考 :doc:`how-to/protection`

Apple M1 上的 Segment fault
===========================

通常情况下，这是因为不正确的 code signature

如果是加密或者注册的时候发生了崩溃，请尝试对扩展模块 ``pytransform3.so`` 重新签名::

    $ codesign -s - -f /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so

如果是运行加密脚本的时候发生了崩溃，请尝试对扩展模块 ``pyarmor_runtime.so`` 重新签名::

    $ codesign -s - -f dist/pyarmor_runtime_000000/pyarmor_runtime.so

请参阅 `Using the latest code signature format`__

__ https://developer.apple.com/documentation/xcode/using-the-latest-code-signature-format/

使用许可相关问题
================

**我们采购流程需要双方签一个合同，请问是否支持呢？**

Pyarmor 是一个工具产品，不额外签订其他合同， `Pyarmor 最终用户许可协议`_ 就是合同文件。

**你们提到一个许可证授权给有且只有一个产品使用，表示的是我只能用来加密一个脚本吗？**

  你可以把“一个产品”替换成为“一种产品”来进行理解，一种产品指的是独立销售的软件所有组成部分，包括开发需要的各种设备，以及提供支持的服务器，云服务器等。一种产品也包括产品的当前版本，历史版本，以及将来的升级版本。一种产品也包括基础功能相同，组合不同特殊功能而形成的不同版本的产品，这种产品的特征是不同版本对外销售名称一样，只是通过辅助名称等来进行区分。

  详细说明和例子请参考 :doc:`licenses`

**是否可以对大文件提供试用许可证**

  不提供。

  Pyarmor 是一个小工具，大部分的功能在试用版中均可以得到验证。对于一些无法试用的高级功能，主要是 RFT 模式和 BCC 模式，这些模式都可以通过配置忽略把不支持的部分脚本或者函数进行忽略。也就是说，只要通过额外的配置，它们总是可以工作的。

**我们有自己的 CICD，需要在镜像内对代码进行加密，那么这是否意味着我们在每次打镜像时都需要通过 “pyarmor register pyarmor-regcode-xxxx.txt” 来对pyarmor进行注册？我们的 CICD 会触发的比较频繁，那么如果比较频繁的进行 register 操作的话是否会影响pyarmor的使用?**

  激活文件 yarmor-regcode-xxxx.txt 一般只用于第一次的注册登记，使用超过十次之后就无法在进行注册。在 CICD 中新镜像使用的时候需要注册，但是不能使用这个文件，可以使用第一次注册登记时候生成的一个 .zip 格式的注册文件。

  具体使用方法请参考 :doc:`how-to/register` 里面的注册许可证中的说明

.. include:: _common_definitions.txt
