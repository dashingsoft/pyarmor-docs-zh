============
 性能和安全
============

.. highlight:: console

.. program:: pyarmor gen

**关于安全性**

Pyarmor 的核心功能是保护 Python 脚本无法被反编译，通过多种不可逆加密模式的实现，已经能够实现 Python 脚本无法使用任何方式完全反编译出来。

如果你看到有人宣称能够的破解 Pyarmor，请首先参考 :doc:`../how-to/security` ，使用你所可用的最高安全选项去加密一个简单的参考脚本，然后尝试使用网上所说的方法和工具进行破解。如果能够被破解，再把Python 版本，Pyarmor 的版本，运行的平台，加密使用的选项，参考脚本以及破解的方法发送到 |Contact|

Pyarmor 并没有提供内存数据保护和很强的反调试保护，即便没有这些保护，Pyarmor 也能保证加密脚本无法被恢复成为原来的脚本。但是对于使用各种逆向工程方法直接修改内存运行数据，以及运行时刻的内存数据，并没有提供完整的保护机制。如何对这方面的要求很高，那就需要结合 Pyarmor 提供的功能以及使用其他反调试工具和技术一起来进行保护，详细说明请参考 :doc:`../how-to/protection`

**关于性能**

Pyarmor 提供了大量的选项来平衡性能和安全性，用户可以根据需要选择不同的选项。这里列出了所有相关的选项以及其对安全性和性能方面的影响。

需要注意的是不同的脚本，以及不同的使用场景，相同的选项对性能的影响可能有很大的不同。这里的所有性能测试数据都是基于同一个简单的测试脚本，如果对性能有很高的要求，请使用相应的场景和脚本进行测试，这里的结果不一定有参考价值。

在运行任何 Python 脚本之前，除非特别说明，都要清除脚本对应的 ``__pycache__`` 。这个目录存放脚本对应 ``.pyc`` ，如果这个目录存在，就意味着执行时间里面没有包含编译脚本的时间，但是编译脚本（.py->.pyc）是特别花费时间的，和函数的执行时间相比，不是一个数量级别。

测试脚本 ``benchmark.py`` 的内容如下

.. code-block:: python

    import sys


    class BenTest(object):

        def __init__(self):
            self.a = 1
            self.b = "b"
            self.c = []
            self.d = {}


    def foo():
        ret = []
        for i in range(100000):
            ret.extend(sys.version_info[:2])
            ret.append(BenTest())
        return len(ret)


主脚本 ``testben.py`` 的内容如下

.. code-block:: python

    import benchmark
    import sys
    import time


    def metric(func):
        if not hasattr(time, 'process_time'):
            time.process_time = time.clock

        def wrap(*args, **kwargs):
            t1 = time.process_time()
            result = func(*args, **kwargs)
            t2 = time.process_time()
            print('%-16s: %10.3f ms' % (func.__name__, ((t2 - t1) * 1000)))
            return result
        return wrap


    def test_import():
        t1 = time.process_time()
        import benchmark2 as m2
        t2 = time.process_time()
        print('%-16s: %10.3f ms' % ('test_import', ((t2 - t1) * 1000)))
        return m2


    @metric
    def test_foo():
        benchmark.foo()


    if __name__ == '__main__':
        print('Python %s.%s' % sys.version_info[:2])
        test_import()
        test_foo()

**默认加密脚本的性能**

首先比较不同 Python 版本下的加密和不加密脚本的性能

使用下面的脚本，分别运行没有加密和加密的脚本两次，其中第二次运行的主要目的是测试存在 ``__pycache__`` 情况下的模块导入时间::

    $ rm -rf dist __pycache__

    $ cp benchmark.py benchmark2.py
    $ python testben.py

    Python 3.7
    test_import     :   1.303 ms
    test_foo        : 250.360 ms

    $ python testben.py

    Python 3.7
    test_import     :   0.290 ms
    test_foo        : 252.273 ms

    $ pyarmor gen testben.py benchmark.py benchmark2.py
    $ python dist/testben.py

    Python 3.7
    test_import     :   0.907 ms
    test_foo        : 311.076 ms

    $ python dist/testben.py

    Python 3.7
    test_import     :   0.454 ms
    test_foo        : 359.138 ms

.. table:: 表-1. 不同 Python 版本的加密脚本性能测试
   :widths: auto

   ==============  =========  =========  =========  =========  =========  =========
   时长(毫秒)       导入模块时间            导入模块时间2           执行函数时间
   --------------  --------------------  --------------------  --------------------
   Python 版本      未加密      加密       未加密      加密       未加密      加密
   ==============  =========  =========  =========  =========  =========  =========
   3.7             1.303      0.907      0.290      0.454      252.2      311.0
   3.8             1.305      0.790      0.286      0.338      272.232    295.973
   3.9             1.198      1.681      0.265      0.449      267.561    331.668
   3.10            1.070      1.026      0.408      0.300      281.603    322.608
   3.11            1.510      0.832      0.464      0.616      164.104    289.866
   ==============  =========  =========  =========  =========  =========  =========

可以看的出来，和之前的版本相比，关于执行函数的时间 Python 3.11 比加密后的脚本快了很多，可能和它的指令缓存和性能优化有关系。

**RFT 模式性能**

RFT 模式是直接把源代码的函数，类，方法和变量进行重命名，所以不应该会影响性能。这里我们比较的是加密后的脚本和 RFT 模式加密的脚本性能数据。下表中的数据是使用下面的测试脚本获得的::

    $ rm -rf dist
    $ pyarmor gen testben.py benchmark.py benchmark2.py
    $ python dist/testben.py

    $ rm -rf dist
    $ pyarmor gen --enable-rft testben.py benchmark.py benchmark2.py
    $ python dist/testben.py

.. table:: 表-2. 默认 RFT 模式性能测试
   :widths: auto

   ==============  =========  =========  =========  =========  ==================
   时长(毫秒)       导入模块时间            执行函数时间            备注
   --------------  --------------------  --------------------  ------------------
   Python 版本      加密      RFT 模式    加密       RFT 模式
   ==============  =========  =========  =========  =========  ==================
   3.7             1.083      1.317      334.313    324.023
   3.8             0.774      1.109      239.217    241.697
   3.9             0.775      0.809      304.838    301.789
   3.10            2.182      1.049      310.046    339.414
   3.11            0.882      0.984      258.309    264.070
   ==============  =========  =========  =========  =========  ==================

接下来，我们组合 RFT 模式和 :option:`--obf-code` 为 ``0`` 对脚本进行加密，然后和没有加密脚本的比较一下性能。下表中的数据是使用下面的测试脚本获得的::

    $ rm -rf dist __pycache__
    $ python testben.py

    $ pyarmor gen --enable-rft --obf-code=0 testben.py benchmark.py benchmark2.py
    $ python testben.py

.. table:: 表-2.1 组合 RFT 模式性能测试
   :widths: auto

   ==============  =========  =========  =========  =========  ==================
   时长(毫秒)       导入模块时间            执行函数时间            备注
   --------------  --------------------  --------------------  ------------------
   Python 版本     未加密     组合模式    未加密     组合模式
   ==============  =========  =========  =========  =========  ==================
   3.7             0.757      1.844      307.325    272.672
   3.8             0.791      0.747      276.865    243.436
   3.9             1.276      0.986      246.407    236.138
   3.10            2.563      1.142      256.583    260.196
   3.11            0.952      0.938      185.435    154.390
   ==============  =========  =========  =========  =========  ==================

**BCC 模式性能**

BCC 模式是把部分函数转换成为 C 函数，应该模块装载时间略长一些，函数执行的时间能稍微快一些。下表中的数据是使用下面的测试脚本获得的::

    $ rm -rf dist __pycache__
    $ python testben.py
    $ python testben.py

    $ pyarmor gen --enable-bcc testben.py benchmark.py benchmark2.py
    $ python dist/testben.py
    $ python dist/testben.py

.. table:: 表-3. 不同 Python 版本的 BCC 模式性能测试
   :widths: auto

   ==============  =========  =========  =========  =========  =========  =========
   时长(毫秒)       导入模块时间            导入模块时间2           执行函数时间
   --------------  --------------------  --------------------  --------------------
   Python 版本      未加密      加密       未加密      加密       未加密      加密
   ==============  =========  =========  =========  =========  =========  =========
   3.7             1.086      1.177      0.342      0.391      344.640    271.426
   3.8             1.099      1.397      0.351      0.400      291.244    251.520
   3.9             1.229      1.076      0.538      0.362      306.594    254.458
   3.10            1.267      0.999      0.255      0.796      302.398    247.154
   3.11            1.146      1.056      0.273      0.536      206.311    189.582
   ==============  =========  =========  =========  =========  =========  =========

**不同选项的性能**

使用不同选项的测试，为了便于比较，每一个选项尽可能的都是单独使用，除非特别情况必须组合使用。

例如，选项 :option:`--no-wrap` 的测试如下::

    $ rm -rf dist __pycache__
    $ pyarmor testben.py

    $ pyarmor gen --no-wrap testben.py benchmark.py benchmark2.py
    $ pyarmor dist/testben.py

    Python 3.7
    test_import     :      0.971 ms
    test_foo        :    306.261 ms

.. list-table:: 表-4. 不同选项对性能和安全性的影响
   :header-rows: 1

   * - 选项
     - 性能影响
     - 安全性
   * - :option:`--no-wrap`
     - 增加性能
     - 降低安全性
   * - :option:`--obf-module` ``0``
     - 对性能影响不大
     - 轻微降低安全性
   * - :option:`--obf-code` ``0``
     - 显著增加性能
     - 显著降低安全性
   * - :option:`--obf-code` ``2``
     - 降低性能
     - 显著增加安全性
   * - :option:`--enable-rft`
     - 基本不影响性能
     - 增强安全性
   * - :option:`--enable-themida`
     - 显著降低性能
     - 显著提高安全性，能有效防止反调试工具
   * - :option:`--mix-str`
     - 降低性能
     - 提高安全性，主要是保护字符串常量
   * - :option:`--assert-call`
     - 降低性能
     - 提高安全性，主要是防止注入和Monkey Trick
   * - :option:`--assert-import`
     - 轻微降低性能
     - 提高安全性，主要是防止注入和Monkey Trice
   * - :option:`--private`
     - 降低性能
     - 提高安全性，主要是防止模块属性被外部脚本读取
   * - :option:`--restrict`
     - 降低性能
     - 提高安全性，主要是防止模块属性被外部脚本读取

.. include:: ../_common_definitions.txt
