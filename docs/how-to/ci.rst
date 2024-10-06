.. highlight:: console

==================================
 在 CI/CD Pipeline 中使用 Pyarmor
==================================

.. versionchanged:: 8.6.0
    从 8.6.0 开始，专家版许可证不能直接应用在 CI/CD pipeline，需要使用专门的 CI 许可证

.. important::

   Pyarmor 8.6.0 对不同许可证在 CI/CD pipeline 的使用进行了重大调整

   在之前购买的 Pyarmor 8 的许可证如果不升级到 8.6.0，可以继续安装原来的方式在 CI/CD pipeline 中使用，最大可用的版本是 8.5.12, 使用文档请参考 `Pyarmor Doc 8.5.12`__

   如果需要升级 Pyarmor 到 8.6+，那么需要按照新的方式进行使用，其中最重要的变化是专家版许可证不能在用于 CI/CD pipeline，而是需要使用新的专门 CI 许可证进行代替

   __ https://pyarmor.readthedocs.io/zh/stable/

从 Pyarmor 8.6.0 开始，Pyarmor 专家版和集团版许可证不可以应用于 CI/CD pipeline

Pyarmor 试用版，基础版和专用的 CI 许可证可以直接应用于 CI/CD pipeline

首先生成一个有效期为 3 天的注册文件，然后在 CI/CD pipeline 中使用此文件进行注册

CI/CD pipeline 需要联网验证时间是否过期

在过期之后，需要重新生成注册文件

.. include:: ../_common_definitions.txt
