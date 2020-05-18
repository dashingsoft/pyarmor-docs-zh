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

支付成功之后注册文件会自动通过电子邮件发送过去。注册文件是一个压缩文件，
里面包含 3 个文件:

* README.txt
* license.lic (注册码)
* .pyarmor_capsule.zip (私有密钥箱)

当收到包含注册文件的邮件之后，保存附件为 `profile-regfile-1.zip` ，然
后使用下面的命令生效注册文件::

    pyarmor register pyarmor-regfile-1.zip

运行下面的命令查看注册信息

    pyarmor register

注册码生效之后，使用试用版本加密的脚本需要全部重新加密。

如果你使用的是 PyArmor 5.6 之前的版本，使用下面的方式注册：

1. 解压注册文件
2. 拷贝解压后的 "license.lic" 到 PyArmor 的安装目录下面
3. 拷贝解压后的 ".pyarmor_capsule.zip" 到用户的 HOME 目录

.. important::

   软件注册码永久有效，可以一直使用，但是不能转让。注册码也有可能在某
   一个新版本无法使用，虽然到现在为止，注册码可以应用于所有版本。

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

.. include:: _common_definitions.txt
