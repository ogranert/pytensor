.. image:: https://cdn.rawgit.com/pymc-devs/pytensor/main/doc/images/PyTensor_RGB.svg
    :height: 100px
    :alt: PyTensor logo
    :align: center

|Tests Status| |Coverage|

|Project Name| is a Python library that allows one to define, optimize, and
efficiently evaluate mathematical expressions involving multi-dimensional arrays.
It provides the computational backend for `PyMC <https://github.com/pymc-devs/pymc>`__.

Features
========

- A hackable, pure-Python codebase
- Extensible graph framework suitable for rapid development of custom operators and symbolic optimizations
- Implements an extensible graph transpilation framework that currently provides
  compilation via C, `JAX <https://github.com/google/jax>`__, and `Numba <https://github.com/numba/numba>`__
- Contrary to PyTorch and TensorFlow, PyTensor maintains a static graph which can be modified in-place to
  allow for advanced optimizations

Getting started
===============

.. code-block:: python

    import pytensor
    from pytensor import tensor as pt

    # Declare two symbolic floating-point scalars
    a = pt.dscalar("a")
    b = pt.dscalar("b")

    # Create a simple example expression
    c = a + b

    # Convert the expression into a callable object that takes `(a, b)`
    # values as input and computes the value of `c`.
    f_c = pytensor.function([a, b], c)

    assert f_c(1.5, 2.5) == 4.0

    # Compute the gradient of the example expression with respect to `a`
    dc = pytensor.grad(c, a)

    f_dc = pytensor.function([a, b], dc)

    assert f_dc(1.5, 2.5) == 1.0

    # Compiling functions with `pytensor.function` also optimizes
    # expression graphs by removing unnecessary operations and
    # replacing computations with more efficient ones.

    v = pt.vector("v")
    M = pt.matrix("M")

    d = a/a + (M + a).dot(v)

    pytensor.dprint(d)
    #  Add [id A]
    #  ├─ ExpandDims{axis=0} [id B]
    #  │  └─ True_div [id C]
    #  │     ├─ a [id D]
    #  │     └─ a [id D]
    #  └─ dot [id E]
    #     ├─ Add [id F]
    #     │  ├─ M [id G]
    #     │  └─ ExpandDims{axes=[0, 1]} [id H]
    #     │     └─ a [id D]
    #     └─ v [id I]

    f_d = pytensor.function([a, v, M], d)

    # `a/a` -> `1` and the dot product is replaced with a BLAS function
    # (i.e. CGemv)
    pytensor.dprint(f_d)
    # Add [id A] 5
    #  ├─ [1.] [id B]
    #  └─ CGemv{inplace} [id C] 4
    #     ├─ AllocEmpty{dtype='float64'} [id D] 3
    #     │  └─ Shape_i{0} [id E] 2
    #     │     └─ M [id F]
    #     ├─ 1.0 [id G]
    #     ├─ Add [id H] 1
    #     │  ├─ M [id F]
    #     │  └─ ExpandDims{axes=[0, 1]} [id I] 0
    #     │     └─ a [id J]
    #     ├─ v [id K]
    #     └─ 0.0 [id L]

See `the PyTensor documentation <https://pytensor.readthedocs.io/en/latest/>`__ for in-depth tutorials.


Installation
============

The latest release of |Project Name| can be installed from PyPI using ``pip``:

::

    pip install pytensor


Or via conda-forge:

::

    conda install -c conda-forge pytensor


The current development branch of |Project Name| can be installed from GitHub, also using ``pip``:

::

    pip install git+https://github.com/pymc-devs/pytensor

Build Instructions and Dependency
---------------------------------
PyTensor use `setuptools <https://setuptools.pypa.io/en/latest/userguide/index.html>`__ to provide package installation (build) and pip support.
The setup configuration is read from the pyproject.toml file.

Packages installed during pip install git... (date 2022/12/20 - pytensor version 'untagged').

* cons-0.4.5
* etuples-0.3.8
* filelock-3.8.2
* logical-unification-0.4.5
* miniKanren-1.0.3
* multipledispatch-0.6.0
* pytensor-2.8.11+62.gd7985536a
* scipy-1.7.3
* setuptools-65.6.3
* six-1.16.0
* toolz-0.12.0
* typing-extensions-4.4.0

Background
==========

PyTensor is a fork of `Aesara <https://github.com/aesara-devs/aesara>`__, which is a fork of `Theano <https://github.com/Theano/Theano>`__.

Contributing
============

We welcome bug reports and fixes and improvements to the documentation.

For more information on contributing, please see the
`contributing guide <https://pytensor.readthedocs.io/en/latest/dev_start_guide.html>`__.

A good place to start contributing is by looking through the issues
`here <https://github.com/pymc-devs/pytensor/issues>`__.


.. |Project Name| replace:: PyTensor
.. |Tests Status| image:: https://github.com/pymc-devs/pytensor/workflows/Tests/badge.svg?branch=main
  :target: https://github.com/pymc-devs/pytensor/actions?query=workflow%3ATests+branch%3Amain
.. |Coverage| image:: https://codecov.io/gh/pymc-devs/pytensor/branch/main/graph/badge.svg?token=WVwr8nZYmc
  :target: https://codecov.io/gh/pymc-devs/pytensor
