Installation Guide
==================

Where to get ChemEx
-------------------

Install files for all platforms (Linux, OS X and Windows) are available 
for `download <https://github.com/gbouvignies/ChemEx/releases>`_ from
`github <https://github.com>`_.

Requirements
------------

ChemEx requires `NumPy <https://numpy.org>`_, 
`SciPy <https://www.scipy.org>`_, `matplotlib <https://matplotlib.org/>`_
and `LMfit <https://lmfit.github.io/lmfit-py/>`_ to be installed.  An easy
way of obtaining and installing these packages is to use a Python distribution
which provides these packages, such as 
`Anaconda <https://www.anaconda.com/distribution/>`_ or 
`Intel Distribution for Python <https://software.intel.com/en-us/distribution-for-python>`_.

.. _ChemEx: https://github.com/gbouvignies/ChemEx

.. highlight:: console

Installation
------------
The easiest way to install `ChemEx`_ is via `conda <https://conda.io/en/latest/>`_ from the command 
line (indicated with ``$``)::

    $ conda install -c conda-forge chemex

Note that there is minimum version of Python required, if your version of Python is less
than 3.7, you can install `ChemEx`_ in a separate conda environment enforcing the use of Python 3.7::

    $ conda create -c conda-forge -n chemex python=3.7 chemex
    $ conda activate chemex

`ChemEx`_ is also available via the Python package index using :command:`pip`::

    $ pip install chemex

The development version can be installed directly from `github <https://github.com>`_ via :command:`pip`::

    $ pip install git+https://github.com/gbouvignies/chemex.git

Another option (not recommended) is to extract the downloaded .tar.gz or .zip file and run::

    $ python setup.py install
