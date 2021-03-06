++++++++++++++++++++
Test the next Python
++++++++++++++++++++

To evolve and remain relevant, incompatible changes must happen in Python. The
problem is how to migrate existing code to the "next" Python with these
incompatible changes.

Deprecation warnings
====================

DeprecationWarning and PendingDeprecationWarning warnings are hidden by
default, to not bother users. Developers can use ``-Wd`` (``-Wdefault``)
command line option on Python which displays these warnings.

`Python Development Mode <https://docs.python.org/dev/library/devmode.html>`_.

What's New in Python 3.9: `You should check for DeprecationWarning in your code
<https://docs.python.org/dev/whatsnew/3.9.html#you-should-check-for-deprecationwarning-in-your-code>`_
section.

Fedora COPR
============

Test Rawhide (future Fedora 33) with Python 3.9 as the "system Python":
https://copr.fedorainfracloud.org/coprs/g/python/python3.9/

pythonci project
================

https://github.com/vstinner/pythonci

Projects having "Python nightly in their CI
===========================================

XXX

numpy?

Cython?

Searching deprecated API usage
==============================

Sourcegraph
-----------

GitHub code search is not powerful enough and there is a lot of noise.  (e.g.
many people copy CPython source code). On the other hand, Sourcegraph only
searches from major repositories, and has powerful filtering. Example:
`search PyEval_ReleaseLock
<https://sourcegraph.com/search?q=PyEval_ReleaseLock+file:.*%5C.%28cc%7Ccxx%7Ccpp%7Cc%29+-file:ceval.c+-file:pystate.c&patternType=literal&case=yes>`_.

Top 4000 packages
-----------------

You can `download a list of top 4000 PyPI packages
<https://hugovk.github.io/top-pypi-packages/>`_ in JSON format from hugovk
site.

INADA-san's script to download sdist packages from the JSON file:
`download_sdist.py
<https://github.com/methane/notes/blob/master/2020/wchar-cache/download_sdist.py>`_.

Note that this script doesn't download packages without sdist (e.g.  only
universal wheel). It is because INADA-san has searched Python/C API. The pain
of the removal can be reduced by fixing most of top 4000 packages.


Docker images
=============

Barry Warsaw maintains `CI Images for Python
<https://gitlab.com/python-devs/ci-images/-/tree/master>`_. Example:
`.gitlab-ci.yml of flufl.lock
<https://gitlab.com/warsaw/flufl.lock/-/blob/master/.gitlab-ci.yml>`_.
