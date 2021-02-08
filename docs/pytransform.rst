.. _运行时刻模块 pytransform:

运行时刻模块 `pytransform`
==========================

如果你意识到加密后的脚本对用户来说就是黑盒子，那么有很多事情都可以在
Python 脚本里来做。在这个时候，模块 :mod:`pytransform` 会提供很多有用
的函数和功能。

模块 :mod:`pytransform` 是和加密脚本一起发布，在运行加密脚本之前必须被
导入进来，所以，你可以在你的脚本中直接导入这个模块，使用里面的函数。

内容
----

.. exception:: PytransformError

   这个异常在 v6.0 之后已经不在使用。

   任何 PyArmor 的 api 失败都会抛出这个异常，传入的参数是错误发生的原因。

.. function:: get_expired_days()

   返回加密脚本的剩余的有效天数。

   >0: 剩余的有效天数

   -1: 永不过期

.. note::

   如果加密脚本已经过期，会直接抛出异常并退出。加密脚本里面的任何代码
   都不会被执行，所以一般情况下这个函数是不会返回 0 的。

.. function:: get_license_info()

   获取加密脚本许可证的相关信息。

   返回一个字典，可能的键名有：

   * ISSUER: 许可证的发布者
   * EXPIRED: 许可过期的日期
   * IFMAC：绑定的网卡 MAC 地址
   * HARDDISK： 绑定的硬盘序列号
   * IPV4：绑定的 IPv4 地址
   * DATA：自定义的数据，用于扩展认证方式
   * CODE：许可注册码

   如果许可证没有包含对应的键名，那么其值为 `None`

   `ISSUER` 是在版本 6.2.5 新增加的，使用试用版本生成的 `license.lic` ，它的值为
   `trial` ，而对于购买 pyarmor 的发布者，它的值是注册码 `pyarmor-vax-NNNNNN` 。
   需要注意的是如果 `license.lic` 是使用 pyarmor 6.0.1 之前的版本生成的，它的值
   是 `None` 。

   如果许可文件非法，例如已经过期，会抛出异常 :exc:`Exception`

.. function:: get_license_code()

   返回字符串，该字符串是生成许可文件时指定的注册码参数。

   如果认证文件非法或者无效，抛出异常 :exc:`Exception`

.. function:: get_user_data()

   返回字符串，该字符串是生成许可文件时指定的 ``-x`` 参数。 如果没有指
   定该参数，那么返回 `None`

   如果认证文件非法或者无效，抛出异常 :exc:`Exception`

.. function:: get_hd_info(hdtype, name=None)

   得到当前机器的硬件信息，通过 *hdtype* 传入需要获取的硬件类型（整型），可用的
   常量如下：

   * HT_HARDDISK 返回硬盘序列号
   * HT_IFMAC 返回网卡Mac地址
   * HT_IPV4 返回网卡的IPv4地址
   * HT_DOMAIN 返回目标设备的域名

   无法获取硬件信息会抛出异常 :exc:`Exception`

   在 Linux 下面，还可以使用参数 `name` 来指定网卡名称，或者硬盘的设备文件名，例如::

     get_hd_info(HT_IFMAC, name="eth2")
     get_hd_info(HT_HARDDISK, name="/dev/vda2")

   在 Windows 下面，可以使用参数 `name` 来获取全部网卡和全部硬盘的信息，例如::

     get_hd_info(HT_IFMAC, name="*")
     get_hd_info(HT_HARDDISK, name="*")
     get_hd_info(HT_HARDDISK, name="/0")    # First disk
     get_hd_info(HT_HARDDISK, name="/1")    # Second disk

   .. note:: 在 v6.5.3 接口参数进行了调整

       * 增加了关键字参数 `name`
       * 删除了关键字参数 `size`

.. attribute:: HT_HARDDISK, HT_IFMAC, HT_IPV4, HT_DOMAIN

   调用  :func:`get_hd_info` 时候 `hdtype` 的可以使用的常量

.. function:: assert_armored(*args)

   必须作为修饰函数来使用，用来检查传入的参数列表中的函数是经过加密的。例如::

    @assert_armored(foo.connect, foo.connect2)
    def start_server():
        foo.connect('root', 'root password')

   抛出异常 :exc:`Exception` 如果任何传入的一个函数不是 PyArmor 加密过的。

.. function:: check_armored(*args)

   返回 True 如果所有列出的函数都是被加密的

   返回 False 如果任何一个列出的函数没有被加密

   .. note:: 在 v6.6.2 中新增


示例
----

下面是一些示例，拷贝这些代码到需要加密的脚本里面，然后加密脚本，运行加密脚本查看效果。

显示加密脚本的剩余的有效天数

.. code-block:: python

   from pytransform import get_license_info, get_expired_days
   try:
       code = get_license_info()['CODE']
       left_days = get_expired_days()
       if left_days == -1:
           print('This license for %s is never expired' % code)
       else:
           print('This license for %s will be expired in %d days' % (code, left_days))
   except Exception as e:
       print(e)


更多内容，请参考 :ref:`使用插件扩展认证方式`

.. note::

   虽然在运行加密脚本的时候 :mod:`pytransform.py` 没有被加密，但是它同样被
   `PyArmor` 所保护。如果对它进行任何修改，运行加密脚本同样会抛出保护异常。

   参考 :ref:`对主脚本的特殊处理`
