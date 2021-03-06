.. _python:

+++++++++++++++++++++++++++++++
The Python programming language
+++++++++++++++++++++++++++++++

.. image:: python.png
   :alt: Python logo
   :align: right
   :target: http://www.python.org/

* `Homepage of the Python project <https://www.python.org/>`_
* :ref:`Unicode in Python 2 and Python 3 <python-unicode>`


Python 2 or Python 3?
=====================

* `pythonclock.org <https://pythonclock.org/>`_: "Python 2.7 will retire in..."
* `Python 3 Statement <https://python3statement.github.io/>`_

My article `Why should OpenStack move to Python 3 right now?
<http://techs.enovance.com/6521/openstack_python3>`_ explains why you should
move to Python 3 right now.

Python 2 will reach its end of life in 2020: see `Python 2.7 Release Schedule
<https://www.python.org/dev/peps/pep-0373/>`_. At Pycon US 2014, the support
`was extended from 2015 to 2020
<https://hg.python.org/peps/rev/76d43e52d978>`_ to give more time to companies
to port their applications to Python 3.


Port Python 2 code to Python 3
==============================

* `six module <http://pythonhosted.org/six/>`_
* `2to6 <https://github.com/limodou/2to6>`_
* `Language differences and workarounds <http://python3porting.com/differences.html>`_ (python3porting book)
* `Porting Python 2 Code to Python 3 <http://docs.python.org/dev/howto/pyporting.html>`_ (docs.python.org)
* `python-porting mailing list <http://mail.python.org/mailman/listinfo/python-porting>`_
* `getpython3.com <http://getpython3.com/>`_
* `Porting code to Python 3 <http://wiki.python.org/moin/PortingToPy3k/>`_ (Python.org wiki)
* `python-incompatibility <http://code.google.com/p/python-incompatibility/>`_
* `2to3c <https://fedorahosted.org/2to3c/>`_
* `py3to2 <https://pypi.python.org/pypi/py3to2>`_
* `Python 3 Wall of Superpowers <https://python3wos.appspot.com/>`_
* `Can I Use Python 3? <https://github.com/brettcannon/caniusepython3>`_


Python packaging
================

* `Install pip
  <http://www.pip-installer.org/en/latest/installing.html>`_
* `pip documentation
  <http://www.pip-installer.org/>`_
* `Python Packaging User Guide
  <http://python-packaging-user-guide.readthedocs.org/>`_
* `Python Wheels
  <http://pythonwheels.com/>`_

.. _py-windows:

Compile Python extensions on Windows
====================================

* Install Windows SDK
* Run "Windows SDK Command Prompt"
* Type::

    setenv /x64 /release
    set MSSDK=1
    set DISTUTILS_USE_SDK=1


Build a Python Wheel package on Windows
=======================================

* `Install pip
  <http://www.pip-installer.org/en/latest/installing.html>`_
* Install wheel using pip::

    \python27\python.exe -m pip install wheel

* Run "Windows SDK Command Prompt"
* Setup the environment to build code in 64-bit mode (replace ``/x64`` with
  ``/x86`` for 32bit)::

    setenv /x64 /release
    set MSSDK=1
    set DISTUTILS_USE_SDK=1

* Go to your project
* Cleanup the project (is it really needed?)::

    del build\*
    del dist\*

* Build the wheel and upload it::

    \python27\python.exe setup.py bdist_wheel upload

Notes:

* To build a 32-bit wheel, you need 32-bit Python and configure the SDK using
  ``/x86``.
* Python 2.7 requires the Windows SDK v7.0 because Python 2.7 is built using
  Visual Studio 2008 (MSVCR90). Python 3.3 is built using Visual Studio 2010.
* It looks like Python 3.3 doesn't need ``MSSDK`` and ``DISTUTILS_USE_SDK``
  environment variables anymore.


Python 3 is better than Python 2
================================

* `10 awesome features of Python that you can't use because you refuse to
  upgrade to Python 3
  <http://asmeurer.github.io/python3-presentation/slides.html>`_

Comparison on unequal types compares the type names
---------------------------------------------------

Python 2 implements a strange comparison operator. For the instruction ``a <
b``, if ``a.__cmp__(b)`` and ``b.__cmp__(a)`` raise ``NotImplementedError`` or
return ``NotImplemented``, Python 2 compares the *name* of the types.
Basically, ``a < b`` becomes ``type(a).__name__ < type(b).__name__``.
It's more complex than that, if a type is a number, the comparison uses an
empty string as the type name.

Extract of the ``default_3way_compare()`` function of Python 2 ``Objects/object.c``::

    /* different type: compare type names; numbers are smaller */
    if (PyNumber_Check(v))
        vname = "";
    else
        vname = v->ob_type->tp_name;
    if (PyNumber_Check(w))
        wname = "";
    else
        wname = w->ob_type->tp_name;
    c = strcmp(vname, wname);

Example in Python 2::

    >>> [1, 2, 3] < "abc"
    True
    >>> type([1, 2, 3]).__name__, type("abc").__name__
    ('list', 'str')
    >>> type([1, 2, 3]).__name__ < type("abc").__name__
    True

As a proof of the behaviour, it's possible to use type subclasses to modify the
type names::


    >>> class z(list): pass
    ...
    >>> class a(str): pass
    ...
    >>> [1, 2, 3] < "abc"
    True
    >>> z([1, 2, 3]) < a("abc")
    False
    >>> type(z([1, 2, 3])).__name__, type(a("abc")).__name__
    ('z', 'a')
    >>> type(z([1, 2, 3])).__name__ < type(a("abc")).__name__
    False

Python 3 doesn't have this strange fallback in comparison. It now raises
TypeError on this case::

    >>> [1, 2, 3] < "abc"
    TypeError: unorderable types: list() < str()

As a consequence, the builtin ``cmp()`` function was removed from Python 3. To
sort a list, the ``key`` parameter of ``list.sort()`` must be used. By the way,
on Python 2, using a *key* function (``list.sort(key=func)``) is more efficient
than using a *cmp* function (``list.sort(cmp=func)``).

On Python 2.7, it's possible to enable Python 3 comparison using ``-3 -Werror``
command line options::

    $ python2 -3 -Werror
    >>> [1, 2, 3] < "abc"
    DeprecationWarning: comparing unequal types not supported in 3.x

Bugs already fixed in Python 3
------------------------------

Some race conditions are already fixed in Python 3. The fix may be backported
to Python 2, but it takes more time because the Python 3 branch diverged from
the Python 2 branch, and Python core developer focus on Python 3.

* `python RLock implementation unsafe with signals
  <http://bugs.python.org/issue13697>`_
* Locks cannot be interrupted by signals in Python 2:
  `Condition.wait() doesn't raise KeyboardInterrupt
  <http://bugs.python.org/issue8844>`_
* subprocess is not thread-safe in Python 2:

  - file descriptor issue (see above)
  - `subprocess.Popen hangs when child writes to stderr
    <http://bugs.python.org/issue1336>`_
  - `Doc: subprocess should warn uses on race conditions when multiple threads
    spawn child processes <http://bugs.python.org/issue19809>`_

In Python 2, file descriptors are inherited by default in the subprocess
module, close_fds must be set explicitly to True. A race condition causes two
child processes to inherit a file descriptor, whereas only one specific child
process was supposed to inherit it. Python 3.2 fixed this issue by closing all
file descriptors by default.  Python 3.4 is even better: now all file
descriptors are not inheritable by default (`PEP 446: Make newly created file
descriptors non-inheritable <http://www.python.org/dev/peps/pep-0446/>`_).


Bugs that won't be fixed in Python 2 anymore
--------------------------------------------

Unicode
^^^^^^^

The Unicode support of Python 3 is much much better than in Python 2. Many
Unicode issues were closed as "won't fix" in Python 2, especially issues opened
after the release of Python 3.0. Some examples:

* `Outputting unicode crushes when printing to file on Linux
  <http://bugs.python.org/issue6832>`_
* `stdout.encoding not set when redirecting windows command line output
  <http://bugs.python.org/issue14192>`_

Bugs in the C stdio (used by the Python I/O)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Python 2 uses the buffer API of the C standard library: ``fopen()``,
``fread()``, ``fseek()``, etcThis API has many bugs. Python works around some
bugs, but some others cannot be fixed (in Python). Examples:

* `Issue #20866: Crash in the libc fwrite() on SIGPIPE (segfault with os.popen and SIGPIPE)
  <http://bugs.python.org/issue20866>`_
* `Issue #21638: Seeking to EOF is too inefficient!
  <http://bugs.python.org/issue21638>`_
* `Issue #1744752: end-of-line issue on Windows on file larger than 4 GB
  <http://bugs.python.org/issue1744752>`_
* `Issue #683160: Reading while writing-only permissions on Windows
  <http://bugs.python.org/issue683160>`_
* `Issue #2730: file readline w+ memory dumps
  <http://bugs.python.org/issue2730>`_
* `Issue #22651: Open file in a+ mode reads from end of file in Python 3.4
  <http://bugs.python.org/issue22651>`_
* `Issue #228210: Threads using same stream blow up (Windows)
  <http://bugs.python.org/issue228210>`_

Python 3 has a much better I/O library: the ``io`` module which uses directly
system calls like ``open()``, ``read()`` and  ``lseek()``.


Hash DoS
^^^^^^^^

The hash function of Python 2 has a "worst complexity" issue which can be
exploited for a denial of service (DoS). It's called the "hash DoS"
vulnerability. Python 3.3 randomizes the hash function by default, Python 2.7
can use randomized hash if enabled explicitly. But the real fix is in Python
3.4 with the `PEP 456 <http://www.python.org/dev/peps/pep-0456/>`_ which now
uses the new SipHash hash function which is much safer.


subprocess
^^^^^^^^^^

The subprocess module is written in pure Python in Python 2.7. There are
complex race conditions. The correct fix was to reimplement the critical part
in C, fix implemented in Python 3.

* `subprocess.Popen hangs when child writes to stderr
  <http://bugs.python.org/issue1336>`_

See also the `PEP 446: Make newly created file descriptors non-inheritable
<http://www.python.org/dev/peps/pep-0446/>`_ which also fixes a complex issues
related to subprocesses, PEP implemented in Python 3.4.

Workaround: install the subprocess32 module from PyPI (and use it instead of
subprocess).


No more polling (busy loop) in Lock.acquire(timeout)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In Python 3.2, locks got a new optional timeout parameter which uses the
native OS function.

Extract of ``threading._Condition.wait(timeout)`` of Python 2.7::

    def wait(self, timeout=None):
        ...
        # Balancing act:  We can't afford a pure busy loop, so we
        # have to sleep; but if we sleep the whole timeout time,
        # we'll be unresponsive.  The scheme here sleeps very
        # little at first, longer as time goes on, but never longer
        # than 20 times per second (or the timeout time remaining).
        endtime = _time() + timeout
        delay = 0.0005 # 500 us -> initial delay of 1 ms
        while True:
            gotit = waiter.acquire(0)
            if gotit:
                break
            remaining = endtime - _time()
            if remaining <= 0:
                break
            delay = min(delay * 2, remaining, .05)
            _sleep(delay)
        ...

Moreover, ``subprocess.Popen.communicate()`` also got a timeout parameter.


Monotonic clocks
^^^^^^^^^^^^^^^^

Timeouts must not use the system clocks but a monotonic clock. It is explained
in the `PEP 418 <http://legacy.python.org/dev/peps/pep-0418/>`_ which has been
implemented in Python 3.3.

Example of issue with system clock changes: `threading.Timer/timeouts break on
change of win32 local time <http://bugs.python.org/issue1508864>`_.

See also the PEP 418 for a list of issues related to the system clock.

Other bugs
^^^^^^^^^^

Misc bugs:

* `Destructor of ElementTree.Element is recursive
  <http://bugs.python.org/issue28871>`_
* `Ctrl-C doesn't interrupt simple loop
  <http://bugs.python.org/issue21870>`_: require the new GIL
  introduced in Python 3.2

Python 2 is slower
------------------

* The C code base doesn't respect strict aliasing and so must be compiled with
  ``-fno-strict-aliasing`` (to avoid bugs when the compiler optimizes the code)
  which is inefficient. The structure of Python C type has been deeply
  rewritten to fix the root cause.
* Python 3 uses less memory for Unicode text thanks to the `PEP 393: Flexible
  String Representation <https://www.python.org/dev/peps/pep-0393/>`_. Many
  operations on "ASCII" strings are faster on Python 3 than Python 2.


Port Python 3 code to Python 2
==============================

Notes based on my experience of porting Tulip to Python 2 (Trollius project).

* Remove keyword-only parameter: replace ``def func(*, loop=None): ...``
  with ``def func(loop=None): ...``
* ``super()`` requires the class and self, *and* the class must inherit from object
* A class must inherit explicitly from object to use properties and ``super()``,
  otherwise ``super()`` fails with a cryptic "TypeError: must be type, not
  classobj" message.
* Python 2.6: ``str.format()`` doesn't support ``{}``. For example,
  ``"{} {}".format("Hello", "World")`` must be written
  ``"{0} {1}".format("Hello", "World")``.
* Replace ``list.clear()`` with ``del list[:]``
* Replace ``list2 = list.copy()`` with ``list2 = list[:]``
* Python 3.3 has new specialized ``OSError`` exceptions: ``BlockingIOError``,
  ``InterruptedError``, ``TimeoutError``, etc. Python 2 has ``IOError``,
  ``OSError``, ``EnvironmentError``, ``WindowsError``, ``VMSError``,
  ``mmap.error``, ``select.error``, etc.
* ``raise ValueError("error") from None`` should be replaced with
  ``raise ValueError("error")``
* ``memoryview`` should be replaced with ``buffer``

Major changes in between Python 2.6 and 3.3:

* ``threading.Lock.acquire()`` and ``subprocess.Popen.communicate()`` support
  timeout.  A busy loop can be used for ``threading.Lock.acquire()``
  (non-blocking call + sleep) in Python 2.
* ``time.monotonic()`` (3.3)
* set and dict literals
* ``memoryview`` object
* ``collections.OrderedDict`` (2.7, 3.1)
* ``weakref.WeakSet`` (2.7, 3.0)
* ``argparse``
* Python 2 doesn't support ``ssl.SSLContext`` nor certificate validation
* ``ssl`` module: ``SSLContext``, ``SSLWantReadError``, ``SSLWantWriteError``,
  ``SSLError``
* Python 2 does not support ``yield from`` and does not support ``return`` in
  generators (3.3)
* Python 2 doesn't support the ``nonlocal`` keyword: use mutable types like
  list or dict instead (3.0)

New modules in the standard library between Python 2.6 and Python 3.3:

* concurrent.futures (3.2)
* faulthandler (3.3)
* importlib (3.1)
* ipaddress (3.3)
* lzma (3.3)
* tkinter.ttk (3.1)
* unittest.mock (3.3)
* venv (3.3)

Python 3.4 has even more modules:

* asyncio
* enum
* ensurepip
* pathlib
* selectors
* statistics
* tracemalloc


History of Python releases
==========================

See also `Status of Python branches
<https://docs.python.org/devguide/#status-of-python-branches>`_.

* Python 3.5: September 2015
* Python 3.4: March 2014
* Python 3.3: September 2012
* Python 3.2: February 2011
* Python 2.7: July 2010
* Python 3.1: June 2009
* Python 3.0: December 2008
* Python 2.6: October 2008
* Python 2.5: September 2006
* Python 2.0: October 2000
* Python 1.5: April 1999



History of the Python language (syntax)
=======================================

* *(Python 3.4: no change)*
* Python 3.3:

  * ``yield from``: `PEP 380 "Syntax for Delegating to a Subgenerator"
    <http://legacy.python.org/dev/peps/pep-0380/>`_
  * ``u'unicode'`` syntax is back: `PEP 414 "Explicit Unicode literals"
    <http://legacy.python.org/dev/peps/pep-0414/>`_

* *(Python 3.2: no change)*
* Python 2.7:

  * all changes of Python 3.1

* Python 3.1:

  * dict/set comprehension
  * set literals
  * multiple context managers in a single with statement

* Python 3.0:

  * all changes of Python 2.6
  * new ``nonlocal`` keyword
  * ``raise exc from exc2``: `PEP 3134 "Exception Chaining and Embedded
    Tracebacks" <http://legacy.python.org/dev/peps/pep-3134/>`_
  * ``print`` and ``exec`` become a function
  * ``True``, ``False``, ``None``, ``as``, ``with`` are reserved words
  * Change from ``except exc, var`` to ``except exc as var``:
    `PEP 3110 "Catching Exceptions in Python 3000"
    <http://legacy.python.org/dev/peps/pep-3110/>`_
  * Removed syntax: ``a <> b``, ```a```, ``123l``, ``123L``, ``u'unicode'``,
    ``U'unicode'`` and ``def func(a, (b, c)): pass``

* Python 2.6:

  * with: `PEP 343 "The "with" Statement"
    <http://legacy.python.org/dev/peps/pep-0343/>`_
  * ``b'bytes'`` syntax: `PEP 3112 "Bytes literals in Python 3000" <http://legacy.python.org/dev/peps/pep-3112/>`_


Python for PHP developers
=========================

* http://www.php2python.com/


Compile CPython on Windows
==========================

To build the Python ssl extension:

Need:

* Visual Studio 2015 (for Python 3.6)
* CPython source code: get it using Mercurial (TortoiseHG on Windows)
* svn.exe in PATH: install TortoiseSVN, but check the [x] command line tools in
  the installer
* ActivePerl: Community Edition, 64-bit

Commands::

    PCbuild\build -p x64 -d -e

See also: PCbuild/readme.txt.


Developer mode
==============

https://mail.python.org/pipermail/python-ideas/2016-March/039314.html

Enable all runtime debug checks in strict mode::

   PYTHONMALLOC=debug python3.6 -Wd -bb -X faulthandler script.py


Python builtin types
====================

* bool, int, float, complex, bytes, str
* tuple, list, dict, set, frozenset


Python packaging
================

* https://hynek.me/articles/conditional-python-dependencies/
* https://packaging.python.org/


Python developer mode
=====================

https://mail.python.org/pipermail/python-ideas/2016-March/039314.html

Strict developer mode::

    PYTHONMALLOC=debug python3.6 -Werror -bb -X faulthandler script.py

Developer mode::

    PYTHONMALLOC=debug python3.6 -Wd -b -X faulthandler script.py

* Show ``DeprecationWarning`` and ``ResourceWarning warnings``: ``python -Wd``
* Show ``BytesWarning`` warning: ``python -b``
* Enable ``faulthandler`` to get a Python traceback on segfault and fatal
  errors: ``python -X faulthandler``
* Debug hooks on Python memory allocators: ``PYTHONMALLOC=debug``
* Enable Python assertions (assert) and set ``__debug__`` to ``True``: remove
  (or just ignore) -O or -OO command line arguments

See also ``PYTHONASYNCIODEBUG=1`` for asyncio.


CPython infra
=============

Python infrastructure
---------------------

* http://infra.psf.io/
* https://status.python.org/ Status of services maintained by the Python infra
  team
* https://github.com/python/psf-chef/
* https://github.com/python/psf-salt/
* PSF pays a full-time sysadmin to maintain the Python infra: XXX
* https://www.python.org/psf/league/
* Managed services: http://infra.psf.io/overview/#details-of-various-services

Package Index (PyPI)
--------------------

* https://pypi.org/ "Warehouse", the new Python Package Index,

  - https://github.com/pypa/warehouse

* https://pypi.python.org/ "Python Cheeseshop", the old Python Package Index
* Python CDN: http://infra.psf.io/services/cdn/

cpython GitHub project
----------------------

* https://github.com/python/cpython/
* GitHub uses mention-bot: https://github.com/facebook/mention-bot

  * https://github.com/mention-bot/how-to-unsubscribe
  * userBlacklist, userBlacklistForPR in `CPython .mention-bot
    <https://github.com/python/cpython/blob/master/.mention-bot>`_
  * Adding you GitHub login to userBlacklistForPR stops the mention bot from
    mentioning anyone on your PRs.

* https://github.com/python/core-workflow/tree/master/cherry_picker/

Misc
----

* http://bugs.python.org/ Bug tracker (modified instance of Roundup)

  * https://pypi.python.org/pypi/roundup
  * Meta bug tracker: http://psf.upfronthosting.co.za/roundup/meta/
    (bug in the bug tracker software)

* Mailing lists: https://mail.python.org/mailman/listinfo

  - python-dev
  - python-ideas
  - python-list
  - lot of Special Interest Groups (SIG)
  - etc.

* http://buildbot.python.org/
* GitHub CLA bot: XXX

Documentation
-------------

* https://docs.python.org/ Python online documentation
* https://github.com/python/docsbuild-scripts/
* Mirror: http://python.readthedocs.io/en/latest/ Still use the old Mercurial repository.
* https://www.python.org/dev/peps/pep-0545/ i18n doc


Python platforms
================

========  =================================================  =========
Platform  sys.platform                                       os.name
========  =================================================  =========
AIX       ``aix3``, ``aix4``                                 ``posix``
Cygwin    ``cygwin``                                         ?
FreeBSD   ``freebsd5``, ``freebsd6``, ...                    ``posix``
Java      ``java`` (with a suffix?)                          ?
Linux     ``linux`` on Python 3, ``linux2`` on Python 2 (*)  ``posix``
Mac OS X  ``darwin``                                         ``posix``
NetBSD    ``netbsd`` (with a suffix?)                        ``posix``
OpenBSD   ``openbsd5``                                       ``posix``
Solaris   ``sunos5``                                         ``posix``
Windows   ``win32``                                          ``nt``
========  =================================================  =========

``sys.platform`` comes from the ``MACHDEP`` variable which is built by the
configure script using:

* ``uname -s`` command output converted to lowercase, with some special rules
  (ex: ``linux3`` is replaced with ``linux`` on Python 3)
* ``uname -r`` command output (or ``uname -v`` on AIX, UnixWare or OpenUNIX)
* ``$host`` variable (``./configure --host=xxx`` parameter)
  when cross-compiling

(*) ``sys.platform`` was also ``linux3`` on old versions of Python 2.6 and
Python 2.7 with Linux kernel 3.x.


Python environment markers
==========================

https://wiki.openstack.org/wiki/Python3#Environment_markers

pip supports environment markers in requirements since pip 6.0, example of
requirement::

    six
    futures; python_version < '3.2'

pip uses ";" (colon) separator but requires "; " (colon, space) if the
requirement uses an URL. A space is added for readability (spaces are ignored).

Environment markers in extra requirements of setup.cfg::

    [extras]
    test =
        six
        futures :python_version < '3.2'

The separator is the ":" (colon), space is only used for readability (spaces
are ignored).

Environment markers in extra requirements of setup.py::

    expected_requirements = {
        "test:python_version < '3.2'": ['futures'],
        "test": ['six']
    }


pip issues
==========

* Upgrading pip3 replaces /usr/bin/pip with the Python 3 pip
* Once, I got two dist-info directories for pip
  (``ls /usr/lib*/python3.4/site-packages/pip-*.dist-info -d``) which broke
  ``python3 -m venv``: ``ensurepip`` was unable to find the system pip and
  Fedora doesn't include bundled wheel packages of ``ensurepip``
  in the ``python3-libs`` package
* With pip 7.0 and newer, ``pip3 install Routes; pip2 install Routes`` installs
  the Python 3 version of Routes on Python 2. pip3 creates a wheel package
  using 2to3 but Routes 2.1 announces universal wheel support which is wrong.
* Wheel caching doesn't work on pip 7.0, 7.0.1 and 7.0.2. It was fixed in pip
  7.0.3.


pbr issues
==========

If running ``tox -e py27`` fails the following error, replace ``2014.1.4`` with
``2014.1.5`` in setup.cfg::

    ValueError: git history requires a target version of
    pbr.version.SemanticVersion(2014.1.5), but target version is
    pbr.version.SemanticVersion(2014.1.4) error in setup command: Error parsing
    /home/haypo/prog/redhat/openstack-nova/setup.cfg: ValueError: git history
    requires a target version of pbr.version.SemanticVersion(2014.1.5), but
    target version is pbr.version.SemanticVersion(2014.1.4)


pytracemalloc
=============

Usage:

* Write a scenario to reproduce the memory leak. The ideal is a scenario taking
  only a few minutes
* Enable tracemalloc and replay the scenario
* Take regulary tracemalloc snapshots
* Compare snapshots
* Enjoy!

If your application only uses Python memory allocators, tracemalloc must show
your the exact memory usage counting every single bytes.

If a C extensions uses other memory allocators like ``malloc()``, tracemalloc
is unable to trace these allocations.

If the application allocates a lot of memory to process some data (memory peak)
and then releases almost all memory, except a few small objects, the memory may
become fragmented. For example, the application only uses 20 MB whereas the
operating system see 24 or 30 MB.

See also:

* `pytracemalloc <http://pytracemalloc.readthedocs.org/>`_
* `Fragmentation of the Heap Memory
  <http://haypo-notes.readthedocs.org/heap_fragmentation.html>`_
* `Python Memory <http://haypo-notes.readthedocs.org/python_memory.html>`_


Zero copy
=========

Python3::

    offset = 0
    view = memoryview(large_data)
    while True:
        chunk = view[offset:offset + 4096]
        offset += file.write(chunk)

This copy creates views on ``large_data`` without copying bytes, no bytes is
copied in memory.


Misc
====

iter(obj):

- obj.__iter__()
- obj.__getitem__()

bool(obj):

- obj.__nonzero__()
- obj.__len__() != 0

item in obj:

 - obj.__contains__()
 - list-like: obj.__getitem__(0), obj.__getitem__(1) until obj.__getitem__(int) returns item!


pudb
====

Put a breakpoint:

* hit 'm', search 'test_api' to open glance.tests.unit.test_api
