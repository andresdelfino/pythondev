.. _finalization:

+++++++++++++++++++
Python Finalization
+++++++++++++++++++

At exit, Python calls ``Py_Finalize()`` which is responsible to stop
Python and cleanly all states, variables, modules. etc. This code is
very fragile.

Issues with clearing memory at exit
===================================

Since Python 3.6, a big refactoring started in CPython to clear more and more
variables and states at Python exit: in ``Py_FinalizeEx()``, but also in Python
``main()`` (for variables which cannot be cleared in ``Py_FinalizeEx()``.

Sadly, it caused some nasty issues:

* `The os module should unset() environment variable at exit
  <https://bugs.python.org/issue39395>`_
* `Make struct module PEP-384 compatible
  <https://bugs.python.org/issue38076#msg351608>`_


Prevent deadlock in io.BufferedWriter
=====================================

https://bugs.python.org/issue23309

Commit::

    commit 25f85d4bd58d86d3e6ce99cb9f270e96bf5ba08f
    Author: Antoine Pitrou <solipsis@pitrou.net>
    Date:   Mon Apr 13 19:41:47 2015 +0200

        Issue #23309: Avoid a deadlock at shutdown if a daemon thread is aborted
        while it is holding a lock to a buffered I/O object, and the main thread
        tries to use the same I/O object (typically stdout or stderr).  A fatal
        error is emitted instead.

Code::

    relax_locking = _Py_IsFinalizing();
    Py_BEGIN_ALLOW_THREADS
    if (!relax_locking)
        st = PyThread_acquire_lock(self->lock, 1);
    else {
        /* When finalizing, we don't want a deadlock to happen with daemon
         * threads abruptly shut down while they owned the lock.
         * Therefore, only wait for a grace period (1 s.).
         * Note that non-daemon threads have already exited here, so this
         * shouldn't affect carefully written threaded I/O code.
         */
        st = PyThread_acquire_lock_timed(self->lock, (PY_TIMEOUT_T)1e6, 0);
    }
    Py_END_ALLOW_THREADS
    if (relax_locking && st != PY_LOCK_ACQUIRED) {
        PyObject *msgobj = PyUnicode_FromFormat(
            "could not acquire lock for %A at interpreter "
            "shutdown, possibly due to daemon threads",
            (PyObject *) self);
        const char *msg = PyUnicode_AsUTF8(msgobj);
        Py_FatalError(msg);
    }


Notes
=====

To workaround `bpo-19565 <https://bugs.python.org/issue19565>`_ on Windows,
multiprocessing crash at exit, ``_winapi.Overlapped`` deallocator leaves
the overlapped handle open if Python is exiting, see the `commit
<https://github.com/python/cpython/commit/633db6f6a69fd44b4a27e7e216ff7a138f69aaf3>`__::

    commit 633db6f6a69fd44b4a27e7e216ff7a138f69aaf3
    Author: Richard Oudkerk <shibturn@gmail.com>
    Date:   Sun Nov 17 13:15:51 2013 +0000

        Issue #19565: Prevent warnings at shutdown about pending overlapped ops.

Python issues
=============

* 2013-10-31: `Clear state of threads earlier in Python shutdown
  <https://bugs.python.org/issue19466>`_. Call
  ``_PyThreadState_DeleteExcept(tstate)`` in ``Py_Finalize()``. This issue
  introduced corrupted a Python frame of an asyncio daemon thread which leaded
  to a crash: `bpo-20526 <https://bugs.python.org/issue20526>`__. I had to
  revert the ``_PyThreadState_DeleteExcept(tstate)`` change.

Cython
======

`__dealloc__()
<https://cython.readthedocs.io/en/latest/src/userguide/special_methods.html#finalization-method-dealloc>`_:

    By the time your ``__dealloc__()`` method is called, the object may already
    have been partially destroyed and may not be in a valid state as far as
    Python is concerned, so you should avoid invoking any Python operations
    which might touch the object. In particular, don’t call any other methods
    of the object or do anything which might cause the object to be
    resurrected. It’s best if you stick to just deallocating C data.


Daemon threads
==============

* Subinterpreters cannot spawn daemon threads anymore, since Python 3.9:
  https://bugs.python.org/issue37266

* In Python 3.8, daemon threads now exit immediately when they attempt to
  acquire the GIL, after Py_Finalize() has been called:

  * https://bugs.python.org/issue36475 with https://github.com/python/cpython/commit/f781d202a2382731b43bade845a58d28a02e9ea1
  * https://bugs.python.org/issue39877 with https://github.com/python/cpython/commit/eb4e2ae2b8486e8ee4249218b95d94a9f0cc513e

* `change <https://hg.python.org/cpython/rev/c2a13acd5e2b>`_ of `bpo-19466
  <https://bugs.python.org/issue19466>`_ caused `bpo-20526
  <https://bugs.python.org/issue20526>`_ regression
