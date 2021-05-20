.. _软件许可:

软件许可
========

PyArmor 是一个共享软件。试用版永不过期，试用版的限制是

* 试用版本可以加密的脚本大小有限制，超过限制的脚本无法进行加密。
* 在试用版中中生成的加密脚本不是私有的，也就是说，其他任何人也可以为这
  些加密脚本生成新的许可文件。
* 试用版本不能下载其他平台最新版本的动态库，以前版本的动态库依旧可以使用。
* 任何人都可以使用本软件加密非商业用途的Python脚本，未经许可不得用于商
  业用途。

关于加密脚本的许可文件，可以参考 :ref:`加密脚本的许可文件`

购买下列任意一种许可证可以解除以上所有的限制

PyArmor 有两种类型的许可证:

a. 个人用户许可，适用于产品的所有权为个人所有。个人用户购买一个许可证
   可以在自己所有的计算机和相关硬件设备上使用。购买这种类型的许可证的
   时候，注册名称填写个人的真实姓名，本产品只授权于注册名称对应的个人
   使用。

   个人用户许可证允许使用本软件加密任何属于自己的 Python 脚本，为加密
   脚本生成私有许可文件，发布加密后的脚本和必要的辅助文件到任何其他设
   备。

   个人用户许可证不允许加密产权属于法人（公司）的 Python 脚本。

b. 企业用户许可，适用于产品的所有权为法人（公司）所有。企业用户购买一
   个软件许可证可以在同一个产品系列的各个项目中使用。购买这种类型的许
   可证的时候，注册名称填写机构名称以及产品名称，例如，“西安德新软件的
   易科系统”，本软件只授权于注册名称对应的产品使用。

   产品系列包括同一个产品升级之后的所有版本。

   企业用户许可证允许使用本软件在任何设备上，加密属于该产品系列的
   Python 脚本，为加密脚本生成私有许可文件，发布加密后的脚本和必要的辅
   助文件到任何其他设备。

   除非有许可人的许可，否则企业用户许可证不可以用于完全独立的其他产品
   系列。如果需要在其他产品系列中使用，必须为其他产品单独购买软件许可。

不管那一种许可方式，本软件都只可用于保护产品本身，不允许应用于产权不属
于被授权人的 Python 脚本。

购买
----

使用微信或者支付宝通过下面的链接购买许可

https://pyarmor.dashingsoft.com/cart/order.html

使用信用卡或者 PayPal 通过下面的连接购买许可

https://order.shareit.com/cart/add?vendorid=200089125&PRODUCT[300871197]=1

需要国内增值税发票的用户通过下面的链接购买许可

http://www.sharebank.com.cn/soft/softbuy.php?soid=52382

对于类型为个人用户的许可，注册名称需要填写正确的姓名。

对于类型为企业用户的许可，除了注册名称需要填写正确的企业名称之外，还需要填写被授
权的产品名称，如果仅在企业内部使用，不会用于任何被销售的产品，可以填写“内部使用”。

支付成功之后一个名为 "pyarmor-regcode-xxxx.txt" 的注册文件会自动通过电子邮件发送
过去，把注册文件保存到磁盘，然后使用下面的命令进行注册

    pyarmor register pyarmor-regcode-xxxx.txt

运行下面的命令查看注册信息

    pyarmor register

注册成功之后，请彻底删除使用试用版本生成的所有文件，然后重新进行加密。

.. note::

    如果你使用的是 PyArmor 6.5.2 之前的版本，使用任何文本编辑器打开注册文件
    "pyarmor-regcode-xxxx.txt"，参考里面的说明进行注册

.. important::

   软件注册码永久有效，可以一直使用，但是注册码可能在某一个新版本之后无法使用。

升级说明
--------

在 **2017-10-10** 之前购买的许可证不在支持免费升级到最新版本，如果需要使用最新版
本的 PyArmor ，需要购买新的许可证。

技术支持
--------

如果使用过程中遇到任何问题，请首先参考这里的 `常见问题和解决方案
<https://pyarmor.readthedocs.io/en/latest/questions.html>`_ ，大部分情况下这里可
以帮助你快速解决问题。

如果没有找到解决方案，那么对于技术问题请点击这里 `报告问题
<https://github.com/dashingsoft/pyarmor/issues>`_ ，并按照模版填写问题报告。对于
商务和有关安全方面的问题，请发送邮件到 jondy.zhao@gmail.com

一般来说，所有问题可以归结为以下三类:

1. PyArmor 无法解决的已知问题
2. PyArmor 自身的 Bug
3. 错误的使用方法

对于第一类，是 PyArmor 无法解决的问题。例如，有些脚本使用 ``inspect`` 去分析加密
脚本的 ByteCode，或者想使用 ``pdb`` 去调试加密脚本，这都是 PyArmor 无法解决的已
知问题。这里列出了所有相关的问题 :ref:`加密脚本和原脚本的区别`

对于第二类的问题，我会尽快解决并发布新版本。

对于第三类问题，有些是和 PyArmor 有关的，有些是和 Python 自身有关的，这些都需要
你去阅读相应的文档并了解使用方法，我只会给出提示，简单的示例和相关的文档链接，而
不会写一个具体到你的实例的操作过程或者把加密相关的所有操作替你完成。

假如你购买了一套微软的 Excel，当你需要制作一个图表的时候，你必须自己学习 Excel
如何制作图表的功能，然后自己制作这个图表，而不是要求微软帮你把图表也制作好。

类似的，pyarmor 也是提供加密功能的工具，并且有详细的文档，所以你首先需要了解它的
功能，并学会使用它。例如，约束模式是 pyarmor 的一个高级功能，它需要你对 Python
有一定程度的了解，然后修改自己的脚本来适合它的要求。那么，你需要了解约束模式，学
习相关的 Python 知识，然后自己修改脚本。我不会去阅读你的脚本，然后帮助你修改和调
整，以使之适用于 pyarmor 的约束模式。

如果你计划加密或者使用第三方工具包，同样我不会为你加密第三方工具包，也不能确保它
可以和 pyarmor 兼容。因为加密脚本和原来的脚本有一定的区别，如果这些区别正好被第
三方包用到，可能就无法正常工作。这里列出了详细的 :ref:`加密脚本和原脚本的区别`

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
