Building the module
===================

Binary releases
---------------

If you're on Windows, you download the current binary release at
https://github.com/bastienleonard/pysfml2-cython/downloads, and ignore
most of this section.  There are two zip files named
``python2-sfml2-cython-win32.zip`` and
``python3-sfml2-cython-win32.zip``. They contain the compiled module,
the dependencies as well as the docs and the examples (note that the
Python 3 version still contains examples written for Python 2,
though).

You should be able to use pySFML 2 without installig anything
else. Feedback is welcome.

On other platforms, there may still be easier ways to build the
module. Someone has written AUR scripts for Arch Linux users:

* https://aur.archlinux.org/packages.php?ID=50841

* https://aur.archlinux.org/packages.php?ID=50842


Getting SFML 2
--------------

The first thing you should do is get `SFML 2
<https://github.com/LaurentGomila/SFML>`_ and make sure it
works. Please refer to the official tutorial:
http://sfml-dev.org/tutorials/2.0/compile-with-cmake.php

Some platforms may make it easier to install it, for example Arch
Linux users can get it from AUR.

If you are on Windows, you will probably want to copy SFML's headers
and libraries directories to the corresponding directories of your
compiler/IDE, and SFML's DLLs to Windows' DLL directory.


Building on Windows
-------------------

If you don't have a C++ compiler installed, I suggest using `MinGW
<http://www.mingw.org>`_.

If you are using a recent version of MinGW, you may encounter this
error when building the module::

    error: unrecognized command line option '-mno-cygwin'

The `problem <http://bugs.python.org/issue12641>`_ is that the
``-mno-cygwin`` has been dropped in recent MinGW releases.  A quick
way to fix this is to remove the option from the distutils
source. Find the ``distutils/cygwinccompiler.py`` in your Python
installation (it should be something like
``C:\Python27\Lib\distutils\cygwinccompiler.py``). Find the
``MinGW32CCompiler`` class and remove the ``-mno-cygwin`` options::

    # class CygwinCCompiler
    self.set_executables(compiler='gcc -mno-cygwin -O -Wall',
                         compiler_so='gcc -mno-cygwin -mdll -O -Wall',
                         compiler_cxx='g++ -mno-cygwin -O -Wall',
                         linker_exe='gcc -mno-cygwin',
                         linker_so='%s -mno-cygwin %s %s'
                                    % (self.linker_dll, shared_option,
                                       entry_point))


Common build options
--------------------

You can build the module with the ``setup.py`` script (or
``setup3k.py`` for Python 3).  This section discusses some common
options that you may need or find useful.

``--inplace`` means that the module will be dropped in the current
directory. I find this more practical, so it makes it easier to test
the module once built.

``--compiler=mingw32`` obviously means that `MinGW`_
will be invoked instead of the default compiler. This is needed when you want
to use GCC on Windows. This command will show you the list of compiler you
specify: ``python setup.py build_ext --help-compiler``.

In the end, the command will look something like this::

    python setup.py build_ext --inplace --compiler=mingw32


Building without Cython
-----------------------

If you downloaded a release that already contains the sf.cpp file, you don't
need to install Cython.

Make sure that ``USE_CYTHON`` is set to ``False`` in setup.py.  You can then
build the module by typing this command::

    python setup.py build_ext


Building with Cython installed
------------------------------

.. warning::

   A common issue on Ubuntu is that the Cython package is currently
   outdated.  One solution is to `install Cython manually
   <http://docs.cython.org/src/quickstart/install.html>`_, for example with
   ``easy_install cython``.

If you downloaded the source straight from the Git repo or if you have
modified the source, you'll need to install Cython to build a module
including the changes.  Also, make sure that ``USE_CYTHON`` is set to
``True`` in setup.py.

When you've done so, you can build the module by typing this command::

    python setup.py build_ext


Building a Python 3 module
--------------------------

It's possible to build a Python 3 module, but you may encounter two problems.

First of all, on my machine, the Cython class used in ``setup3k.py`` to
automate Cython invocation is only installed for Python 2. It's
probably possible to install it for Python 3, but it's not complicated
to invoke Cython manually::

    cython --cplus sf.pyx

The next step is to invoke the ``setup3k.py`` script to build the
module. Since we called Cython already, make sure that ``USE_CYTHON``
is set to ``False`` in ``setup3k.py``, then invoke this command::

    python3 setup3k.py build_ext

(Note that you may have to type ``python`` instead of ``python3``;
typically, GNU/Linux systems provide this as a way to call a specific
version of the interpreter, but I'm not sure that's the case for all
of them as well as Windows.)

(Also note that on GNU/Linux, the generated file won't be called
``sf.so`` but something like ``sf.cpython-32mu.so``. Apparently, on
Windows it's still ``sf.pyd``.)

The second problem is that the SFML API uses raw strings a lot. This
maps well into Python 2: you just use normal string litterals most of
the time, except that when you want to use the Unicode functionality
exposed in the :py:class:`sf.Text` class.

However, in Python 3, string literals are Unicode by default, and you
need to use the ``b`` prefix if you want a raw string.  For example,
when you create a :py:class:`sf.RenderWindow`::

    window = sf.RenderWindow(video_mode, b'The title')
