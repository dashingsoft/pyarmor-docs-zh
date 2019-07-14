.. _加密脚本和原脚本的区别:

加密脚本和原脚本的区别
======================

加密脚本和原来的脚本相比，存在下列一些的不同:

* 运行加密脚本的 Python 主版本和加密脚本使用的 Python 主版本应该要一致，
  因为加密的脚本实际上已经是 `.pyc` 文件，如果主版本不一致，有些指令无
  法识别或者会出错。尤其是 Python3.6，在这个版本引入了新的指令系统，所
  以和 Python3.5 以及之前的版本完全不同。

* 执行加密角本的 Python 不能是调试版，准确的说，不能是设置了
  Py_TRACE_REFS 或者 Py_DEBUG 生成的 Python

* 使用 ``sys.settrace``, ``sys.setprofile``, ``threading.settrace`` 和
  ``threading.setprofile`` 设置的回调函数在加密脚本中将被忽略

* 代码块的属性 ``__file__`` 在加密脚本是 ``<frozen name>`` ，而不是文件
  名称，在异常信息中会看到文件名的显示是 ``<frozen name>``

  需要注意的是模块的属性 ``__file__`` 还和原来的一样，还是文件名称。加
  密下面的脚本并运行，就可以看到输出结果的不同::

      def hello(msg):
          print(msg)

      # The output will be 'foo.py'
      print(__file__)

      # The output will be '<frozen foo>'
      print(hello.__file__)

第三方解释器的支持
------------------

对于第三方的解释器（例如 Jython 等）以及通过嵌入 Python C/C++ 代码调用
加密脚本，需要满足下列条件:

* 第三方解释器或者嵌入的 Python 代码必须装载 Python 官方的动态库，动态
  库的源代码在 https://github.com/python/cpython ，并且核心代码不能被
  修改，修改后的代码可能会导致加密脚本无法执行。

* 在 Linux 下面 装载 Python 动态库 `libpythonXY.so` 的时候 `dlopen` 必
  须设置 `RTLD_GLOBAL` ，否则加密脚本无法运行。

.. note::

   Boost::python，默认装载 Python 动态库是没有设置 `RTLD_GLOAL` 的，运
   行加密脚本的时候会报错 "No PyCode_Type found" 。解决方法就是在初始
   化的调用方法 `sys.setdlopenflags(os.RTLD_GLOBAL)` ，这样就可以共享
   动态库输出的函数和变量。

* 模块 `ctypes` 必须存在并且 `ctypes.pythonapi._handle` 必须被设置为
  Python 动态库的句柄，PyArmor 会通过该句柄获取 Python C API 的地址。


.. include:: _common_definitions.txt
