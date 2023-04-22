============
 定制和扩展
============

.. contents:: 内容
   :depth: 2
   :local:
   :backlinks: top

.. highlight:: console

.. program:: pyarmor gen

Pyarmor 提供了 :term:`加密插件` 和 :term:`运行插件` 分别来扩展功能。

其中 :term:`加密插件` 用来增加

Using plugins
=============

.. versionadded:: 8.x
                  This feature is still not implemented

Plugin is used to do some pre-build or post-build work in generating obfuscated scripts.

Use cases:

- Copy data files to output
- In MacOS, codesign binary extension :mod:`pyarmor_runtime`
- For multiple platforms, rename binary extension :mod:`pyarmor_runtime` suffix to avoid name confilcts

First create any script in :term:`local configuration path`. For example, create :file:`.pyarmor/myplugin.py`

.. code-block:: python

    from pyarmor.cli.plugin import BuildPlugin, KeyPlugin, RuntimePlugin

    __all__ = ['DataFilePlugin', 'NotePlugin', 'DarwinPlugin', 'SuffixPlugin']

    class DataFilePlugin(BuildPlugin):

        def process(self, inputs, output):
            print('context:', self.ctx)
            print('inputs are', inputs)
            print('output path', output)

    class NotePlugin(KeyPlugin):

        def process(self, filename, **options):
            lines = ['# %s: %s' % (k, v) for k, v in options.items()]
            notes = '\n'.join(lines).encode()
            with open(filename, 'rb') as f:
                keydata = f.read()
            with open(filename, 'wb') as f:
                f.write(notes + b'\n' + keydata)

    class DarwinPlugin(RuntimePlugin):

        def process(self, platname, filename):
            print('context:', self.ctx)
            if platname.startswith('darwin.'):
                check_output('codesign -f -s - %s' % filename)

Then use all of them::

    $ pyarmor cfg plugins = "myplugin"

Or part of them::

    $ pyarmor cfg plugins = "myplugin.DarwinPlugin myplugin.NotePlugin"

Using hooks
===========

.. versionadded:: 8.x
                  This feature is still not implemented

Hook is used when running obfuscated scripts, it mainly does some special protection work.

A hook is a Python script, generally run in main script, but it also could be called

- when loading the runtime package :mod:`pyarmor_runtime`
- when imporing each obfuscated module
- periodically only when runtime key is in period mode

Use cases:

- read user data in runtime key, and verify it by user
- set runtime language in boot
- copy runtime key when packing script to one file
- call some extra anti-debug routines defined by user

If same full module name script in the ``.pyarmor/hooks``, it will be embedded into this module. For example, ``.pyarmor/hooks/alec.main.py`` will be inserted into module ``src/alec/main.py``. For package ``__init__.py``, it's hook file name is package name.

There is a special hook script ``.pyarmor/hooks/pyarmor_runtime.py`` will be embedded into extension `pyarmor_runtime` init function.


Create hook script in the `.pyarmor/hooks/foo.py`

.. code-block:: python

    from http.client import HTTPSConnection
    expire = __pyarmor__(1, None, b'keyinfo', 1)
    host = 'worldtimeapi.org'
    path = '/api/timezone/Europe/Paris'
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
         host = 'timeapi.io'
         path = '/api/Time/current/zone?timeZone=Europe/Amsterdam'
         ...

Then generate script with local time::

    $ pyarmor gen -e .30 foo.py

.. include:: ../_common_definitions.txt
