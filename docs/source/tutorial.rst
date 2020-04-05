.. _chemex_tutorial:

========
Tutorial
========

Introduction
------------

ChemEx is a python module for analyzing chemical exchange detected by NMR.  
It is designed to take almost any kind of NMR data to aid the analysis, the 
most commonly used experiments include CPMG relaxation dispersion and 
Chemical Exchange Saturation Transfer (CEST). This tutorial provides an 
overview of some of the major features of ChemEx.

ChemEx performs numerical fitting by minimizing predefined :math:`\chi^2`, which 
typically has the following form:

.. math::
    \chi^2 = \sum_{i}{\left(\frac{I^{expt}_i-I^{calc}_i}{\sigma^{expt}_i}\right)^2}

For any given set of parameters, ChemEx performs numerical simulation and 
calculate the corresponding :math:`\chi^2` value. The purpose of numerical 
fitting is to locate the sets of parameters that reaches :math:`\chi^2` 
minimum, which is carried out with Levenberg-Marquardt non-linear optimization
with `LMfit <https://lmfit.github.io/lmfit-py/>`_ module. Unlike 
most other programs for chemical exchange data analysis, ChemEx does not 
rely on analytical equations therefore most experimental details (e.g. 
finite pulse width, off-resonance effects etc.) can be taken into account.


Running ChemEx
--------------

ChemEx is intended to use with the command line (indicated with ``$``), a 
typical command for running ChemEx is:

.. code-block:: console

    $ chemex fit -e <FILE> \
                 -p <FILE> \
                 -m <FILE> \
                 -d <MODEL> \
                 -o <DIR>

Such command is usually saved in a shell script to save some typing efforts.
The first argument should be one of the following:

================   ========================================================
``info``           Show experiments that can be fit
``config``         Show sample configuration files of the modules
``fit``            Start a fit
``simulate``       Start a simulation
``pick_cest``      Plot CEST profiles for dip picking
``plot_param``     Plot one selected parameter from a 'parameters.fit' file
================   ========================================================

The second argument and later can be one of the following:

====================  ===============================================================
``-h`` or ``--help``  Show help message
``-e``                Input files containing experimental setup and data location
``-p``                Input files containing the initial values of fitting parameters
``-m``                Input file containing the fitting method
``-o``                Directory for output files
``-d``                Exchange model used to fit the data
``--plot``            Plotting level (``nothing``, ``normal``, ``all``)
``--include``         Residue(s) to include in the fit
``--exclude``         Residue(s) to exclude from the fit
``--mc``              Number of Monte-Carlo simulations
``--bs``              Number of Bootstrap simulations
====================  ===============================================================


File formats
------------

The input and output files of ChemEx uses `TOML <https://en.wikipedia.org/wiki/TOML>`_
file format. Detailed description and usage about TOML file format can be found in the 
`project home page <https://github.com/toml-lang/toml>`_.


Experiment files
----------------

The experiment files (indicated with ``-e``) contain information such as the name and 
Larmor frequency etc., it typically looks like this:

.. literalinclude:: experiment.toml
    :language: toml

Several most commonly used keys among all experiments are summarized as below:

+-------------------------+--------------------------+-------------------------------------------+
| Section                 | Key                      |     Description                           |
+=========================+==========================+===========================================+
| :confval:`[experiment]` |  :confval:`name`         |     Experiment name                       |
+                         +--------------------------+-------------------------------------------+
|                         |  :confval:`carrier`      |  RF carrier of studied nuclei in ppm      |
+-------------------------+--------------------------+-------------------------------------------+
| :confval:`[conditions]` | :confval:`h_larmor_freq` |   Magnetic field strength in MHz          |
+                         +--------------------------+-------------------------------------------+
|                         | :confval:`label`         |   Labeling scheme of the sample           |
+-------------------------+--------------------------+-------------------------------------------+
| :confval:`[data]`       | :confval:`path`          |  The directory containing data files      |
+                         +--------------------------+-------------------------------------------+
|                         | :confval:`error`         |  The method for error estimation          |
+                         +--------------------------+-------------------------------------------+
|                         | :confval:`profiles`      |  The name of each data file               |
+-------------------------+--------------------------+-------------------------------------------+

Aside from that each experiment also contains certain keys that are specific for 
each type of experiment, an example config file for each type of experiment can be 
generated with:

.. code-block:: console

    $ chemex config <NAME>

Look for those comments in each example config file to better understand the meaning for each key.

.. important::

    Under :confval:`profiles` section, the name of each dataset should be chosen properly 
    according to the spin system for each experiment, e.g. :confval:`F4N` should be used
    for experiment with single-spin system, :confval:`F4N-HN` should be used for experiment
    with two-spin system.

Data files
----------

The location of data files is specified in experiment files. Data files typically 
contain three columns with the following information:

+---------------------------+--------------+---------------+-----------------+
| Experiment                | First column | Second column |  Third column   |
+===========================+==============+===============+=================+
|    CPMG                   |     ncyc     |   Intensity   |   Uncertainty   |
+---------------------------+--------------+               +                 +
|    CEST/DCEST/COSCEST     | Offset (Hz)  |               |                 |
+---------------------------+--------------+               +                 +
|    Relaxation             |   Time (s)   |               |                 |
+---------------------------+--------------+---------------+-----------------+

An example data file looks like this:

.. literalinclude:: data.out
    :language: python


Parameter files
---------------

The parameter files (indicated with ``-p``) contain initial estimate of parameters
to be used during the fitting process, which typically looks like this:

.. literalinclude:: parameters.toml
    :language: toml

If certain required parameter is not included in the parameter files, a default
value will be used to initialize, and the initial value depends on each specific module.
Due to the multidimensional feature of the minimization process, it is essential to
set suitable initial value for each parameter to avoid being trapped in a local minimum. 


Method files
------------

The method file (indicated with ``-m``) contain the fitting methods to be used 
during the fitting process, which typically looks like this:


Kinetic models
--------------

The kinetic model (indicated with ``-d``) indicates the type of exchange model to be
used for the data analysis, which can be one of the following:

====================  ============================================================================
``2st``               2-state exchange model (default)
``3st``               3-state exchange model
``4st``               4-state exchange model
``2st_rs``            2-state exchange model for residue-specific study
``2st_hd``            2-state exchange model for H/D solvent exchange study
``2st_eyring``        2-state exchange model for temperature-dependent study
``3st_eyring``        3-state exchange model for temperature-dependent study
``2st_binding``       2-state exchange model for binding study
``4st_hd``            4-state exchange model for simutaneous normal and H/D solvent exchange study
====================  ============================================================================

For each kinetic model, it is possible to add ``.mf`` suffix (e.g. ``2st.mf``) to 
directly fit model-free parameters (e.g. :confval:`TAUC_A`, :confval:`S2_A` etc.) instead of each 
individual relaxation parameter (e.g. :confval:`R1_A`, :confval:`R2_A` etc.).

Multiple states under exchange are distinguished with parameter suffix :confval:`A`, :confval:`B`, 
:confval:`C`, :confval:`D` etc.  For example, :confval:`R1_A` indicates R\ :sub:`1` rate 
of the major state (i.e. ground state), :confval:`R2_B` indicates R\ :sub:`2` rate of the first minor state.


Output files
------------

The output files

first
  second 
    3rd
      4th

  [
  Ix(a), Iy(a), Iz(a),
  Ix(b), Iy(b), Iz(b),
  ...]


[From Mark Nodine] For cells in simple tables that comprise a single line, the justification can be inferred according to the following rules:

    1. If the text begins at the leftmost column of the cell, then left justification, ELSE
    2. If the text begins at the rightmost column of the cell, then right justification, ELSE
    3. Center justification.
