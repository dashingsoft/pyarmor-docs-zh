.. _pyarmor 的安全性:

PyArmor 的安全性
================

PyArmor 使用分片式技术来保护 Python 脚本。所谓分片保护，就是单独加密
每一个函数， 在运行脚本的时候，只有当前调用的函数被解密，其他函数都没有
解密。而一旦函数执行完成，就又会重新加密，这是 PyArmor 的特点之一。

例如，下面这个脚本 `foo.py`::

  def hello():
      print('Hello world!')

  def sum(a, b):
      return a + b

  if __name == '__main__':
      hello()
      print('1 + 1 = %d' % sum(1, 1))

PyArmor 会首先加密函数 `hello` 和 `sum` ，然后在加密整个模块，进行两次
加密。当运行加密的 `hello` 的时候， `sum` 依旧是加密的。`hello` 执行完
成之后，会被重新加密，然后才开始解密并执行 `sum` 。

.. _交叉保护机制:

交叉保护机制
------------

PyArmor 的核心代码使用 `c` 来编写，所有的加密和解密算法都在动态链接库中
实现。 首先 `_pytransform` 自身会使用 `JIT` 技术，即动态生成代码的方式
来保护自己，加密后的 Python 脚本由动态库 `_pytransform` 来保护，反过来，
在加密的 Python 的脚本里面，也会来校验动态库，确保其没有进行任何修改。
这就是交叉保护的原理，Python 代码和 `c` 代码相互进行校验和保护，大大提
高了安全性。

动态库保护的核心有两点:

1. 用户不能通过修改代码段指令来获得没有授权的使用。例如，将指令 `JZ` 修
   改为 `JNZ` ，从而使得认证失败可以继续执行
2. 加密 Python 脚本使用的键值不能通过反向跟踪的方式获取到

那么， `JIT` 是如何来做到的呢？

PyArmor 定义了一套自己的指令系统（基于 GNU lightning)，然后把核心函数，
主要是获取键值的算法，加解密的过程等，使用自己的指令系统生成数据代码。
数据代码存放在一个单独的 `c` 文件中，内容如下::

    t_instruction protect_set_key_iv = {
        // function 1
        0x80001,
        0x50020,
        ...

        // function 2
        0x80001,
        0xA0F80,
        ...
    }

    t_instruction protect_decrypt_buffer = {
        // function 1
        0x80021,
        0x52029,
        ...

        // function 2
        0x80001,
        0xC0901,
        ...
    }

这是两个受保护的函数，每一个受保护的函数里面会有很多小函数段。随后编译
动态库，计算代码段的校验和，使用这个真实的代码段的校验和替换相关的指令，
并对数据代码进行混淆，修改后的文件如下::

    t_instruction protect_set_key_iv = {
        // function 1, 不混淆
        0x80001,
        0x50020,
        ...

        // function 2，混淆下面的数据指令
        0xXXXXX,
        0xXXXXX,
        ...
    }

    t_instruction protect_decrypt_buffer = {
        // function 1, 不混淆
        0x80021,
        0x52029,
        ...

        // function 2，混淆下面的数据指令
        0xXXXXX,
        0xXXXXX,
        ...
    }

使用修改后的文件重新编译生成动态库，这个动态库会发布给客户。

当加密脚本运行的时候，每一次调用被保护的函数的时候，就会进入 `JIT` 动态
代码保护例程:

1. 读取 `functiion 1` 的数据代码，动态生成 `function 1`
2. 执行 `function 1`::

    检查代码段的校验和，如果不一致，退出
    检查当前是否有调试器，如果发现，退出
    检查执行时间是否太长，如果执行时间太长，退出
    如果可能的话，清除硬件断点寄存器
    恢复下一个函数 `function 2` 的数据代码

3. 读取 `functiion 2` 的数据代码，动态生成 `function 2`
4. 重复步骤 2 的操作

这样循环有限次之后，真正受保护的代码才被执行。总之，主要达到的目的是开
始执行受保护的代码的时候，不能被调试器中断。

为了在 Python 端保护动态库没有被进行任何修改，在加密主脚本的时候，会插
入额外的一段代码来检查和保护动态链接库，详细工作原理参考 :ref:`Special
Handling of Entry Script`

当然，为了提高安全性，你也可以使用第三方保护工具，例如 ASProtect_,
VMProtect_ 等来保护动态库。

.. include:: _common_definitions.txt
