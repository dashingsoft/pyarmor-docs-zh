==========
 常见问题
==========

.. _asking questions:

如何在 Github 上提问
====================

TBD

是否可以对大文件提供试用许可证
==============================

不提供。

注册相关问题
============


* 由于我们有自己的 CICD pipline，所以我们希望在打项目镜像时在镜像内对代码进行加
  密，那么这是否意味着我们在每次打镜像时都需要通过 “pyarmor register
  pyarmor-regcode-xxxx.txt” 来对pyarmor进行注册？我们的 CICD 会触发的比较频繁，
  那么如果比较频繁的进行 register 操作的话是否会影响pyarmor的使用?

  回答::

  > 激活文件 yarmor-regcode-xxxx.txt 一般只用于第一次的注册登记，使用超过十次之
  > 后就无法在进行注册。在 CICD 中新镜像使用的时候需要注册，但是不能使用这个文件，
  > 可以使用第一次注册登记时候生成的一个 .zip 格式的注册文件。

  具体使用方法请参考 :doc:`how-to/register` 里面的注册许可证中的说明

.. include:: _common_definitions.txt
