======================
 最高安全性和最快性能
======================

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

如何生成最高安全性的脚本
========================

下面的选项都可以用来增加安全性

* :option:`--enable-rft` 几乎不影响性能，是推荐选项
* :option:`--enable-bcc` 可能会显著增加模块导入时间
* :option:`--enable-jit` 可以防止静态反编译
* :option:`--enable-themida` 可以防止动态调试器，但是对性能影响比较大，并且仅在 Windwos 可用
* :option:`--mix-str` 保护脚本的所有字符串常量
* `pyarmor cfg mix_argnames=1` 保护函数参数，但是可能导致 annotations 不可用

下面的选项可以隐藏加密模块的属性，外部脚本无法直接导入和使用加密脚本

* 加密脚本使用 :option:`--private` ，加密包使用 :option:`--restrict`

下面的选项可以防止各种注入攻击和替换加密脚本中函数等攻击

* :option:`--assert-call`
* :option:`--assert-import`

如何生成最快的加密脚本
======================

使用下面的选项可以生成性能最高的加密脚本，其安全性相当于 `.pyc`

* :option:`--obf-code` ``0``
* :option:`--obf-module` ``0``
* `pyarmor cfg restrict_module=0`

如果需要提高安全性，但是对性能又不要影响太大，最后的选择是同时启用 :term:`RFT 模式`

* :option:`--enable-rft`

如果有敏感字符串，那么使用 :option:`--mix-str` 同时设置过滤条件，仅仅加密这些敏感字符串。如果没有过滤条件，所有的字符串常量都会加密，可能对性能会造成一点影响

* `pyarmor cfg mix.str:includes "/regular expression/"`
* :option:`--mix-str`

不同类型应用的推荐选项
======================

**对于服务网络请求类型的应用**

   如果 :term:`RFT 模式` 已经足够安全（通过检查转换后的脚本来确定是否足够安全），那么使用下面的选项进行加密

   * :option:`--enable-rft`
   * :option:`--obf-code` ``0``
   * :option:`--obf-module` ``0``
   * :option:`--mix-str` 和过滤条件，如果不设置过滤条件能满足性能要求，也可以不设置

   如果单独的 :term:`RFT 模式` 不能满足安全需求，那么使用下面的选项

   * :option:`--enable-rft`
   * :option:`--no-wrap`
   * :option:`--mix-str` 和过滤条件

**对于其他大多数的应用**

   如果 :term:`RFT 模式` 和 :term:`BCC 模式` 可用，那么使用下面的选项

   * :option:`--enable-rft`
   * :option:`--enable-bcc`
   * :option:`--mix-str` 和过滤条件
   * :option:`--assert-import`

   如果不可用，使用下面的选项

   * :option:`--enable-jit`
   * :option:`--private` （加密脚本），或者 :option:`--restrict` （加密包）
   * :option:`--mix-str` 和过滤条件
   * :option:`--assert-import`

   如果需要避免注入攻击，使用下面的选项

   * :option:`--assert-call` ，同时检查跟踪日志，确保关键函数被保护，必要的时候使用运行 :term:`脚本补丁` 对这些函数进行保护

   如果性能允许的话，启用选项 :option:`--enable-themida` ，这个选项还是能很大程度的防止调试器的攻击，但就是对性能影响大一些

.. include:: ../_common_definitions.txt
