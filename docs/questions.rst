==========
 常见问题
==========

.. _asking questions:

如何在 Github 上提问
====================

Pyarmor 是一个命令行工具，提供了丰富的选项来应对不同的情况。对于简单的应用，使用默认选项就可以工作，不需要学习如何使用 Pyarmor。但是对于一些高级的功能，或者比较复杂的应用，尤其是需要使用大型的第三方包来调用加密脚本，用户需要一定的时间去学习和使用各种选项，并组合这些选项去解决加密过程中问题。

当加密脚本出现问题的时候，这不一定是 Pyarmor 的 Bug，很可能是这个脚本使用默认的选项和配置无法工作。这种情况下需要分析问题并找到 Pyarmor 解决这个问题的选项和设置，这就需要用户参考 Pyarmor 的命令手册去找到合适的选项。

**Pyarmor 开发组不会去花费时间去了解客户的应用程序，然后告诉用户需要使用什么选项去解决加密过程中遇到的问题，用户需要自己学习 Pyarmor 的各个选项如何使用，并根据自己的需要选择正确的选项**

Pyarmor 有完整的文档系统，所有的功能和选项都已经在文档中描述，Pyarmor 开发组能告诉你的，这些文档中都已经有说明。

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

- 浏览一下本页后面的常见问题

- 运行没有加密的脚本，确保没有加密之前的脚本能够正确的运行

- 如果加密的是复杂的应用，请首先测试一个简单的脚本，确认使用同样的加密选项的简单脚本能够工作

- 启用调试模式，跟踪日志，在控制台输出更多信息，仔细检查日志信息，确认问题所在

- 如果使用的不是最新版本的 Pyarmor，请升级到最新版本

- 搜索一下 `问题报告`_

- 搜索一下 `讨论区`_

如果你不想花费时间去阅读文档，请在 `讨论区`_ 提问，但是 Pyarmor 开发组并不保证一定回答这部分的所有问题。

如果你发现 Pyarmor 中存在 Bug，请提交到 `问题报告`_

Pyarmor 开发组欢迎真正的 Bug 报告，并且会尽快解决这些 Bug。报告 Bug 请提供必要的信息以便 Pyarmor 开发组能够重现问题，这样有助于快速解决问题，无法被 Pyarmor 开发组 重现的问题解决起来需要更长的周期。运行 :ref:`pyarmor gen` 命令的时候，控制台的前四行包含加密使用的选项，Pyarmor 的版本信息，Python 的版本信息，以及当前的运行平台，请务必以文本的形式拷贝这四行到 Bug 报告中。对于无法重现的 Bug 报告，Pyarmor 开发组会把它转入到讨论区，以后可以重现的时候，在转回到 Bug 并继续处理。为了方便处理 Bug，请尽量不要使用截图，而是直接拷贝文本。例如，一般报告应该以这样的方式开始::

   **加密使用的选项和命令**
   ```
   $ pyarmor cfg wrap_mode 1
   $ pyarmor gen --assert-call foo.py

   INFO     Python 3.9.0
   INFO     Pyarmor 8.1.6 (pro), 005068, btarmor
   INFO     Platform darwin.x86_64
   ```

热点问题
========

**看到网上有破解 Pyarmor 的工具？请问 Pyarmor 能够防止这些破解工具吗？**

  Pyarmor 开发组一般不会关注和了解这些破解工具，而是致力于研究 CPython 的源代码，以改进算法来提高安全性，所以没有办法直接回答这类问题。

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

Apple 上的 Segment fault
========================

首先升级 Pyarmor 到 8.3.0 之后的版本，这个版本解决了使用非系统 Python 导致的崩溃问题

如果已经是最新版本，那么检查预编译的扩展模块 ``pytransform3.so`` 和 ``pyarmor_runtime.so``

- 确保它们被正确签名， ``codesign -v /path/to/xxx.so``
- 检查所有的依赖库， ``otool -L /path/to/xxx.so`` ，确保所有的依赖库都存在

如果是 Pyarmor 8.3.0 之前的版本，主要有下面的几种原因

1. 通常情况下，这是因为不正确的 code signature

如果是加密或者注册的时候发生了崩溃，请尝试对扩展模块 ``pytransform3.so`` 重新签名::

    $ codesign -s - -f /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so

如果是运行加密脚本的时候发生了崩溃，请尝试对扩展模块 ``pyarmor_runtime.so`` 重新签名::

    $ codesign -s - -f dist/pyarmor_runtime_000000/pyarmor_runtime.so

请参阅 Apple 官方文档 `Using the latest code signature format`__

2. 另外一种原因是 Python 的依赖库无法找到

使用非标准方式安装的 Python 也可能会导致崩溃，需要使用 otool 和 install_name_tool 解决依赖库问题

因为预编译的扩展模块需要一些依赖库，如果依赖库的位置不对，就可能直接崩溃。使用 ``otool -L`` 可以查看依赖库::

    $ otool -L /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so

    /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so:
	pytransform3.so (compatibility version 0.0.0, current version 1.0.0)
	@rpath/lib/libpython3.9.dylib (compatibility version 3.9.0, current version 3.9.0)
        ...

除了系统库之外，主要就是 Python 动态库 ``@rpath/lib/libpython3.9.dylib`` ，其中默认配置的 ``rpath`` 为::

    $ install_name_tool -id pytrnsform3.so \
            -change $deplib @rpath/lib/libpython$ver.dylib \
            -add_rpath @executable_path/.. \
            -add_rpath @loader_path/.. \
            -add_rpath /System/Library/Frameworks/Python.framework/Versions/$ver \
            -add_rpath /Library/Frameworks/Python.framework/Versions/$ver \
            build/$host/libs/cp$ver/$name.so

也可以使用下面的命令查看 ``rpath``::

    $ otool -l /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so

检查当前系统是否存在 ``@rpath/lib/libpython3.9.dylib`` ，如果不存在这个文件的话，需要使用 ``install_name_tool`` 适配当前的 Python 安装环境，假设 Python 动态库是 ``/usr/local/Python.framework/Versions/3.9/Python``::

    $ install_name_tool -change @rpath/lib/libpython3.9.dylib /usr/local/Python.framework/Versions/3.9/Python \
            /path/to/lib/pythonX.Y/site-packages/pyarmor/cli/core/pytransform3.so

对于 ``dist/pyarmor_runtime_000000/pyarmor_runtime.so`` 也是同样的，必须保证依赖库都存在，否则需要进行适配运行环境。

如何找到当前 Python 解释器对应的动态库，请自行搜索答案。注意有些预编译的 Python 没有使用动态库，那么是无法运行加密脚本的，需要重新编译支持动态库的版本。

请参阅 Apple 官方文档 `Run-Path Dependent Libraries`__

3. 如果系统安装了多个 Python，确保链接到正确的动态库

例如，除了系统 Python3.9 之外，还使用 anaconda3 安装了 Python 3.9 在 ``/Users/my_username/anaconda3/bin/python``

当使用 ``/Users/my_username/anaconda3/bin/python`` 运行加密脚本的时候，首先会装载动态库 ``dist/pyarmor_runtime_000000/pyarmor_runtime.so`` ，这个动态库依赖 Python 系统库，按照配置会首先查找 ``/Users/my_username/anaconda3/bin/python/../lib/libpython3.9.dylib`` ，如果这个文件不存在，那么会继续查找 ``/Library/Frameworks/Python.framework/Versions/3.9/lib/libpython3.9.dylib`` ，如果这个文件存在，那么会使用系统的 Python 动态库文件，这样可能就会造成崩溃问题。

这种情况下就需要使用 install_name_tool 修改 ``dist/pyarmor_runtime_000000/pyarmor_runtime.so`` 适配当前的运行环境，使之能够装载 anaconda3 对应的 Python 动态库。Pyarmor 默认适配的是系统 Python ，并且不可能自动适配所有的运行环境，需要用户根据部署环境进行适配。

4. 权限设置问题

Pyarmor 使用了 JIT 技术来提高安全性，在 Apple M1，这可能需要对 Python 进行某些设置。使用下面的命令检查 Python 的 entitlements 并进行必要的设置::

    $ codesign -d --entitlements - $(which python)

请参阅 Apple 官方文档 `Allow Execution of JIT-compiled Code Entitlement`__

5. 最后看一下 Apple 的 segment fault 日志，根据提示的错误信息搜索网络找解决方案

__ https://developer.apple.com/documentation/xcode/using-the-latest-code-signature-format/
__ https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/RunpathDependentLibraries.html
__ https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_allow-jit

注册问题
========

**ERROR request license token failed (104)**

  首先请确认网络可用，其次检查防火墙设置，如果可能的话，暂时关闭防火墙进行测试。

  在 Windows 下面，防火墙要允许动态库 ``pytransform3.pyd`` 访问 `pyarmor.dashingsoft.com` 的端口 ``80`` ，在其他系统，防火墙要允许 ``pytransform3.so`` 访问 `pyarmor.dashingsoft.com`  的端口 ``80`` ，具体防火墙的规则设置请参阅防火墙的文档。

**集团版许可证报错： ERROR request license token failed**

  首先升级 Pyarmor 到 8.3.5+

  其次使用调试选项 ``-d`` 在离线机器上面运行注册命令，例如::

      $ pyarmor -d reg pyarmor-device-regfile-6000.4.zip

  检查控制台的输出日志，确保离线许可证的设备包含当前设备。例如::

      DEBUG    group license for machines: ['tokens/gb04eb35da4f5378185c8663522e0a5e3']
      DEBUG    got machine id: gb04eb35da4f5378185c8663522e0a5e3

  如果设备不匹配，请使用 Pyarmor 8.3.5 以后的版本重新为当前设备生成离线注册文件。

  对于虚拟设备，请确保每次启动之后的设备 id 均保持一致。

使用许可相关问题
================

**我的软件每年大概能分发1W+，一个集团版许可证能绑定1万个离线设备吗? 这些设备到期后，许可证的数量能回收吗**

这里明确一些基本概念

- 运行加密脚本的机器不需要安装 Pyarmor，也不需要 Pyarmor 许可证
- Pyarmor 许可证主要应用于加密脚本的机器
- 集团版许可证只供企业用户使用，一般情况下个人用户推荐购买专家版许可证
- 关于 Pyarmor 许可证的详细说明文档，请参阅 :doc:`licenses`

**购买后能够多设备使用吗？ 如果可以多设备使用，最大的设备数上限是多少?**

可以，但是必须应用于同一个产品。

对于基础版和专家版许可证，二十四小时之内最多可以有 100 台不同设备使用 Pyarmor ，每一个 Docker 容器也会被认为是一台设备。

对于集团版许可证，最多可以在 100 台离线设备使用，但是可以在离线设备上运行数量不受限制的 Docker 容器

**请问如何选择不同的许可证类型?**

1. 如果使用的是 Python 2.7 或者 Python 3.6 之前的版本，只能选择老版本的许可证。

2. 如果需要离线加密，只能选择集团版许可证。

3. 如果在 24 小时内需要运行超过 100 个 Docker 容器进行加密，那么只能选择集团版许可证

4. 推荐使用专家版许可证

**如果现在购买基础版后续是否可以补差价升级专家版或者集团版本?**

目前还不支持各种版本的补差价升级。

**购买时为不含税,后续是否可以通过补齐同版本差价进行开发票?**

目前不支持补齐差价开发票。

**我们采购流程需要双方签一个合同，请问是否支持呢？**

Pyarmor 是一个工具产品，不额外签订其他合同， `Pyarmor 最终用户许可协议`_ 就是合同文件。

**你们提到一个许可证授权给有且只有一个产品使用，表示的是我只能用来加密一个脚本吗？**

  你可以把“一个产品”替换成为“一种产品”来进行理解，一种产品指的是独立销售的软件所有组成部分，包括开发需要的各种设备，以及提供支持的服务器，云服务器等。一种产品也包括产品的当前版本，历史版本，以及将来的升级版本。一种产品也包括基础功能相同，组合不同特殊功能而形成的不同版本的产品，这种产品的特征是不同版本对外销售名称一样，只是通过辅助名称等来进行区分。

  详细说明和例子请参考 :doc:`licenses`

**是否可以对大文件提供试用许可证**

  不提供。

  Pyarmor 是一个小工具，大部分的功能在试用版中均可以得到验证。对于一些无法试用的高级功能，主要是 RFT 模式和 BCC 模式，这些模式都可以通过配置忽略把不支持的部分脚本或者函数进行忽略。也就是说，只要通过额外的配置，它们总是可以工作的。

**我们有自己的 CICD，需要在镜像内对代码进行加密，那么这是否意味着我们在每次打镜像时都需要通过 “pyarmor register pyarmor-regcode-xxxx.txt” 来对pyarmor进行注册？我们的 CICD 会触发的比较频繁，那么如果比较频繁的进行 register 操作的话是否会影响pyarmor的使用?**

  激活文件 yarmor-regcode-xxxx.txt 一般只用于第一次的注册登记，在 CICD 中新镜像使用的时候需要注册，但是不能使用这个文件，可以使用第一次注册登记时候生成的一个 .zip 格式的注册文件。

  具体使用方法请参考 :doc:`how-to/register` 里面的注册许可证中的说明

**我们之前是使用的Pyarmor 6.8.1版本进行的前期测试和验证，现在想要正式购买授权，但目前最新版本已经是8.0+，那这个之前的6.8.1版本的授权该如何购买**

  如果确定是使用 Pyarmor 8 之前的版本，在 `购买页面`__ 可以选择购买老版本许可证。

  直接支持老版本的许可证的最终版本是 Pyarmor 7.7.4

  如果要在 Pyarmor 8.0+ 中使用老版本的许可证，需要使用命令 pyarmor-7

  __ https://pyarmor.dashingsoft.com/cart/order.html

.. include:: _common_definitions.txt
