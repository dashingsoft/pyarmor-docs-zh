====================
 激活和注册 Pyarmor
====================

.. contents:: Contents
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor reg

第一次注册登记
==============

首先参考 :doc:`../licenses` 购买合适的许可证。购买完成之后，一个名称为 :file:`pyarmor-regcode-xxxx.txt` 的 :term:`激活文件` 将会很快发送到注册邮箱中，这个文件用于第一次的注册登记。

在第一次注册的时候，需要使用 :option:`-p` 指定许可证绑定的产品名称，如果没有使用该选项，那么产品名称被自动设定为 ``non-profits`` ，该许可证不能被应用于商用产品中。

注册登记需要网络连接。

内部使用的注册方式
------------------

如果该许可证只在内部使用，而不会在任何商业产品中使用，可以使用下面的方式注册::

    $ pyarmor reg pyarmor-regcode-xxxx.txt


在商用产品中使用的注册方式
--------------------------

假设许可证将被用来保护产品 ``Robot Studio`` ，那么使用下面的命令进行初始登记注册::

    $ pyarmor reg -p "Robot Studio" pyarmor-regcode-xxxx.txt

Pyarmor 会首先显示注册信息并请求确认，如果确认无误，输入 :kbd:`yes` 并 :kbd:`Enter` 继续下面的注册步骤，其他任何输入都会终止注册过程。

注册成功之后会同时生成一个名字类似 :file:`pyarmor-regfile-xxxx.zip` 的 :term:`注册文件` ，这个文件被用来在其他设备上进行注册。

:term:`激活文件` 在第一次注册成功之后就无法继续使用，后续在任何设备上注册请使用 :term:`注册文件` 。请妥善保管并备份注册文件，一旦丢失，Pyarmor 并不提供找回服务。

一旦注册成功之后，许可证绑定的产品名称就不能在进行修改。

绑定许可证到正在开发的产品
--------------------------

如果产品还在开发中，没有正式开始销售，允许初始化登记的时候设定产品的名称为 ``TBD`` 。例如::

    $ pyarmor reg -p "TBD" pyarmor-regcode-xxxx.txt

绑定的产品名称为 ``TBD`` 的许可证，必须在产品销售之前，使用下面的命令设置为正确的产品名称::

    $ pyarmor reg -p "Robot Studio" pyarmor-regcode-xxxx.txt

在其他设备上面注册
==================

一旦初始登记完成之后，就会得到一个 :term:`注册文件` ，通常名称为 :file:`pyarmor-regfile-xxxx.zip` 。

拷贝这个压缩文件到其他需要注册的设备，然后运行下面的命令进行注册::

    $ pyarmor reg pyarmor-regfile-xxxx.zip

查看注册信息::

    $ pyarmor -v

在 Docker 或者 CI pipeline 中注册
---------------------------------

在 Docker 或者 CI pipeline 中注册 Pyarmor 的基本方法同上，但是同时运行的 Pyarmor 的 Docker 数量有限制，不能超过 100 个。

**只允许在开发设备上安装和注册 Pyarmor。如果 Docker 镜像需要发送给客户，那么不允许在上面安装和注册 Pyarmor**

.. include:: ../_common_definitions.txt
