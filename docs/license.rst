.. _软件许可:

软件许可
========

PyArmor 是一个共享软件。试用版永不过期，试用版的限制是

* 最大可加密的脚本大小（编译成为 `.pyc` 之后）是 **32768** 个字节
* 在试用版中中生成的加密脚本不是私有的，也就是说，其他任何人也可以为这
  些加密脚本生成新的许可文件。
* 使用高级模式加密脚本的时候，每一个模块最多可以有大约 32 个函数（代码块）
* 任何人都可以使用本软件加密非商业用途的Python脚本，未经许可不得用于商
  业用途。

关于加密脚本的许可文件，可以参考 :ref:`加密脚本的许可文件`

生成私有的加密脚本和加密任意大小的脚本需要购买下列任意一种许可证。

PyArmor 有两种类型的许可证:

a. 个人用户许可，适用于产品的所有权为个人所有。个人用户购买一个许可证
   可以在自己所有的计算机和相关硬件设备上使用。

   个人用户许可证允许使用本软件加密任何属于自己的 Python 脚本，为加密
   脚本生成私有许可文件，发布加密后的脚本和必要的辅助文件到任何其他设
   备。

   个人用户许可证不允许加密产权属于法人（公司）的 Python 脚本。

b. 企业用户许可，适用于产品的所有权为法人（公司）所有。企业用户购买一
   个软件许可证可以在同一个产品系列的各个项目中使用。

   产品系列包括同一个产品升级之后的所有版本。

   企业用户许可证允许使用本软件在任何设备上，加密属于该产品系列的
   Python 脚本，为加密脚本生成私有许可文件，发布加密后的脚本和必要的辅
   助文件到任何其他设备。

   除非有许可人的许可，否则企业用户许可证不可以用于完全独立的其他产品
   系列。如果需要在其他产品系列中使用，必须为其他产品单独购买软件许可。

不管那一种许可方式，本软件都只可用于保护产品本身，不允许应用于产权不属
于被授权人的 Python 脚本。例如，开发平台工具 PyCharm 购买的 PyArmor 许
可证只能用于加密和保护 PyCharm 自身，而不能使用 PyArmor 的许可为其他使
用 PyCharm 进行开发的 Python 脚本提供加密功能。

购买
----

使用微信或者支付宝通过下面的链接购买许可

https://pyarmor.dashingsoft.com/cart/order.html

使用信用卡或者 PayPal 通过下面的连接购买许可

https://order.shareit.com/cart/add?vendorid=200089125&PRODUCT[300871197]=1

需要国内增值税发票的用户通过下面的链接购买许可

http://www.sharebank.com.cn/soft/softbuy.php?soid=52382

支付成功之后一个名为 "pyarmor-regcode-1.txt" 的注册文件会自动通过电子邮件发送过
去，把注册文件保存到磁盘，然后使用下面的命令进行注册

    pyarmor register pyarmor-regcode-1.txt

运行下面的命令查看注册信息

    pyarmor register

注册成功之后，请彻底删除使用试用版本生成的所有文件，然后重新进行加密。

.. note::

    如果你使用的是 PyArmor 6.5.2 之前的版本，使用任何文本编辑器打开注册文件
    "pyarmor-regcode-1.txt"，参考里面的说明进行注册

.. important::

   软件注册码永久有效，可以一直使用，但是注册码可能在某一个新版本之后无法使用。

升级说明
--------

在 **2017-10-10** 之前购买的许可证不在支持免费升级到最新版本，如果需要使用最新版
本的 PyArmor ，需要购买新的许可证。

技术支持
--------

问题报告请提交到 https://github.com/dashingsoft/pyarmor/issues ，有关安全的问题
请直接发送邮件到 jondy.zhao@gmail.com ，一般在 24 小时之内回复。

一般来说，所有问题可以归结为以下三类:

1. PyArmor 自身的 Bug
2. PyArmor 无法解决的已知问题
3. 错误的使用方法

这里列出了所有和第二类相关的问题，:ref:`加密脚本和原脚本的区别` 。例如，有些脚本
使用 ``inspect`` 去分析加密脚本的 ByteCode，或者想使用 `pdb` 去调试加密脚本，这
都是 PyArmor 无法解决的已知问题。

使用 PyArmor 需要必要的技能和知识，例如，如何使用命令行等，请使用之前确保这些知
识已经掌握 :ref:`必要的知识和技能`

当收到问题报告的时候，会根据类型进行相应的处理。对于第一类的问题，我会尽快解决并
发布新版本。但是对于第三类问题，有些是和 PyArmor 有关的，有些是和 Python 自身有
关的，这些都需要你去阅读相应的文档并了解使用方法，我只会给出提示，简单的示例和相
关的文档链接，而不会写一个具体到你的实例的操作过程或者把加密相关的所有操作替你完
成。

假如你购买了一套微软的 Excel，当你需要制作一个图表的时候，你必须自己学习 Excel
如何制作图表的功能，然后自己制作这个图表，而不是要求微软帮你把图表也制作好。

类似的，pyarmor 也是提供加密功能的工具，并且有详细的文档，所以你首先需要了解它的
功能，并学会使用它。例如，约束模式是 pyarmor 的一个高级功能，它需要你对 Python
有一定程度的了解，然后修改自己的脚本来适合它的要求。那么，你需要了解约束模式，学
习相关的 Python 知识，然后自己修改脚本。我不会去阅读你的脚本，然后帮助你修改和调
整，以使之适用于 pyarmor 的约束模式。

如果你计划加密第三方工具包，同样我不会为你加密第三方工具包，也不能确保它可以和
pyarmor 兼容。因为加密脚本和原来的脚本有一定的区别，如果这些区别正好被第三方包用
到，可能就无法正常工作。这里列出了详细的 :ref:`加密脚本和原脚本的区别`

Python 在各个领域得到广泛的应用，有很多包我甚至从来都没有用过。对于我来说，不会
去学习每一个包，然后确保其能够和 pyarmor 兼容。通常的处理方式是用户报告相关的异
常，根据异常的行号给出附近的源代码，我可以帮助分析哪些地方可能和 pyarmor 发生冲
突并给出相应的解决方案。

.. important::

   微信主要用于处理订单相关的事宜，不接受和处理技术方面的问题报告。

Q & A
-----

1. Single PyArmor license purchased can be used on various machines for
   obfuscation? or its valid only on one machine? Do we need to install license
   on single machine and distribute obfuscate code?

   | It can be used on various machines, but one license only for one product.

2. Single license can be used to obfuscate Python code that will run various
   platforms like windows, various Linux flavors?

   | For all the features of current version, it's yes. But in future versions,
   | I'm not sure one license could be used in all of platforms supported by
   | PyArmor.

3. How long the purchased license is valid for? is it life long?

   | It's life long. But I can't promise it will work for the future version of PyArmor.

4. Can we use the single license to obfuscate various versions of Python
   package/modules?

   | Yes, only if they're belong to one product.

5. Is there support provided in case of issues encountered?

   | Report issue in github or send email to me.

6. Does Pyarmor works on various Python versions?

   | Most of features work on Python27, and Python30~Python38, a few features
   | may only work for Python27, Python35 later.

7. Are there plans to maintain PyArmor to support future released Python
   versions?

   | Yes. The goal of PyArmor is let Python could be widely used in the
   | commercial softwares.

8. What is the mechanism in PyArmor to identify whether modules belong to same
   product? how it identifies product?

   | PyArmor could not identify it by itself, but I can check the obfuscated
   | scripts to find which registerred user distributes them. So I can find two
   | products are distributed by one same license.

9. If product undergoes revision ie. version changes, can same license be used
   or need new license?

   | Same license is OK.

10. What means a product serials under PyArmor EULA?

   | A product serial means a sale unit and its upgraded versions. For
   | example, AutoCAD 2003, 2010 could be taken as a product serials.

.. include:: _common_definitions.txt
