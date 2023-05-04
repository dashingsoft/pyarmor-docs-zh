============
 定制和扩展
============

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 使用下面的方式进行定制和扩展

- 使用命令 :ref:`pyarmor cfg` 修改默认配置
- 使用 :term:`加密插件` 对加密过程和输出文件进行扩展和定制
- 使用 :term:`脚本补丁` 对运行时刻的加密脚本进行扩展和定制

设置运行辅助包的名称
====================

.. versionadded:: 8.2 [#]_

默认情况下运行辅助包的名称是 ``pyarmor_runtime_xxxxxx``

这个名称可以被配置成为任何合法的包名称。例如设定名称为 ``my_runtime``::

    pyarmor cfg package_name_format "my_runtime"

.. [#] 试用版本不可以修改运行辅助包的名称，修改后的加密脚本无法运行

自定义需要保护的函数和模块
==========================

.. versionadded:: 8.2

Pyarmor 8.2 新增加一个配置项 ``auto_mode`` 用来实现自定义需要保护的函数和模块，它的默认值为 ``and`` ，这时候过滤方式和以前的版本是一样的。 ``and`` 的含义是所有的操作对象除了是自动识别之外，还必须满足 ``includes`` 和 ``excludes`` 条件。

如果修改其值为 ``or`` ，则表示除了自动识别的函数和模块之外，还需要保护 ``includes`` 里面的函数。例如，下面的命令，，除了保护自动识别的函数之外，还额外保护函数 ``foo`` 和 ``koo``::

    $ pyarmor cfg ast.call:auto_mode "or"
    $ pyarmor cfg ast.call:includes "foo koo"

    $ pyarmor gen --assert-call foo.py

下面的命令可以用来保护没有使用 ``import`` 语句直接导入的加密模块 ``joker.card``::

    $ pyarmor cfg ast.import:auto_mode "or"
    $ pyarmor cfg ast.import:includes "joker.card"

    $ pyarmor gen --assert-import joker/

使用加密插件修正运行辅助包的依赖项
==================================

.. versionadded:: 8.2

在使用 Dawin 的设备中，如果 Python 没有安装在标准路径，那么运行加密脚本的时候可能会因为找不到依赖的 Python 动态库而出现装载错误。

如果需要运行加密脚本的环境在这种设备，那么在加密脚本的时候需要修正运行辅助包的依赖库位置。

首先查看一些运行辅助包中动态库 ``pyarmor_runtime.so`` 的依赖项::

    $ otool -L dist/pyarmor_runtime_000000/pyarmor_runtime.so

    dist/pyarmor_runtime_000000/pyarmor_runtime.so:

	pyarmor_runtime.so (compatibility version 0.0.0, current version 1.0.0)
        ...
	@rpath/lib/libpython3.9.dylib (compatibility version 3.9.0, current version 3.9.0)
        ...

如果 :term:`客户设备` 上面没有 ``@rpath/lib/libpython3.9.dylib`` ，而是 ``@rpath/lib/libpython3.9.so`` ，那么加密脚本无法被装载。

这时候可以通过插件来修正这个问题，首先创建一个插件脚本 :file:`.pyarmor/conda.py`:

.. code-block:: python

    __all__ = ['CondaPlugin']

    class CondaPlugin:

        def _fixup(self, target):
            from subprocess import check_call
            check_call('install_name_tool -change @rpath/lib/libpython3.9.dylib @rpath/lib/libpython3.9.so %s' % target)
            check_call('codesign -f -s - %s' % target)

        @staticmethod
        def post_runtime(ctx, source, target, platform):
            if platform.startswith('darwin.'):
                print('using install_name_tool to fix %s' % target)
                self._fixup(target)

启用这个插件脚本，然后重新加密脚本::

    $ pyarmor cfg plugins + "conda"
    $ pyarmor gen foo.py

请根据具体的环境修改上面的插件脚本以满足需要。

.. seealso:: :ref:`plugins`

使用脚本补丁绑定脚本到 Docker
=============================

.. versionadded:: 8.2

假设我们要把脚本 ``app.py`` 绑定运行在两个 Docker 上面，它们的 id 分别是 ``docker-a1`` ， ``docker-b2``

那么，首先创建一个 :term:`脚本补丁` ``.pyarmor/hooks/app.py``

.. code-block:: python

    def _pyarmor_check_docker():
        cid = None
        with open("/proc/self/cgroup") as f:
            for line in f:
                if line.split(':', 2)[1] == 'name=systemd':
                    cid = line.strip().split('/')[-1]
                    break

        docker_ids = __pyarmor__(0, None, b'keyinfo', 1).decode('utf-8')
        if cid is None or cid not in docker_ids.split(','):
            raise RuntimeError('license is not for this machine')

    _pyarmor_check_docker()

然后加密脚本，同时把 Docker 的信息存储到 :term:`运行密钥` 中::

    $ pyarmor gen --bind-data "docker-a1,docker-b2" app.py

运行加密脚本以验证其效果，可以增加一些 print 语句在脚本补丁中进行调试。

.. seealso:: :ref:`hooks` :func:`__pyarmor__`

使用其他网络时间服务来检查脚本有效期
====================================

.. versionadded:: 8.2

默认情况下 Pyarmor 是请求 NTP 服务器来验证加密脚本的有效期，如果 NTP 端口没有开放，也可以通过 :term:`脚本补丁` 使用其他网络时间服务器来进行验证。

首先创建脚本补丁 ``.pyarmor/hooks/foo.py``

.. code-block:: python

    def _pyarmor_check_worldtime(host, path):
        from http.client import HTTPSConnection
        expired = __pyarmor__(1, None, b'keyinfo', 1)
        conn = HTTPSConnection(host)
        conn.request("GET", path)
        res = conn.getresponse()
        if res.code == 200:
            data = res.read()
            s = data.find(b'"unixtime":')
            n = data.find(b',', s)
            current = int(data[s+11:n])
            if current > expire:
                raise RuntimeError('license is expired')
         else:
             raise RuntimeError('got network time failed')
    _pyarmor_check_worldtime('worldtimeapi.org', '/api/timezone/Europe/Paris')

然后加密脚本，有效期的设置使用本地时间::

    $ pyarmor gen -e .30 foo.py

这样就可以使用定制的代码检查网络时间。

.. seealso:: :ref:`hooks` :func:`__pyarmor__`

.. include:: ../_common_definitions.txt
