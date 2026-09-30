.. _model-summary:

The model summary tool
======================

Once a model has finished, it is often useful to check quickly which input parameters and which elemental abundances it actually used, for example when comparing models of a grid or revisiting an old run. ``model_summary.py``, located in the repository root next to ``params.dat``, reads the output files of a model and prints a compact summary of its parameters together with the total elemental abundances. It is a plain Python 3 script that requires only ``numpy`` (and, optionally, ``h5py``).

Usage
-----

Run the script from the repository root with the output prefix of the model, i.e. the path of the output files without their suffix. Since outputs are written to ``sims/``, the prefix includes that directory:

.. code-block:: console

   $ python3 model_summary.py sims/PREFIX

Any other path works too (e.g. ``python3 model_summary.py /path/to/sims/PREFIX``). The script reads the following files (see :doc:`outputs`):

- ``PREFIX.species``: the list of species, which gives the order of the abundance columns.
- ``PREFIX.params``: the global model parameters.
- ``PREFIX.pdr.h5`` if it exists, otherwise ``PREFIX.pdr.fin``: the abundances of a single cell.

The HDF5 file is produced by the converter described in :doc:`convert`. It is used automatically whenever it exists and ``h5py`` is installed. If ``h5py`` is not available the script prints a note and falls back to ``PREFIX.pdr.fin``. In both cases **only one cell is read**, so the script runs in a fraction of a second even for multi-GB outputs of large 3D grids.

Options:

.. list-table::
   :header-rows: 1
   :widths: 15 40

   * - Option
     - Purpose
   * - ``--cell N``
     - Cell used to compute the elemental abundances, given as the 1-based line number of ``.pdr.fin`` (default: 1). The same cell is selected when reading ``.pdr.h5``.
   * - ``--top K``
     - Number of main carrier species listed for each element (default: 3).

What it reports
---------------

**Model parameters.** The first three rows of ``PREFIX.params`` are interpreted as:

.. list-table::
   :header-rows: 1
   :widths: 8 40

   * - Row
     - Content
   * - 1
     - FUV intensity :math:`G_0` (Draine units), cosmic-ray ionization rate :math:`\zeta` (s\ :sup:`-1`), dust-to-gas ratio (metallicity, relative to solar; it also scales :math:`A_V/N_{\rm H}`), microturbulent velocity (km s\ :sup:`-1`).
   * - 2
     - Resolution of the grid: ``xres yres zres``.
   * - 3
     - Size of the box: ``xsize ysize zsize`` (pc).

When the code is compiled with ``CRATTENUATION=1``, the cosmic-ray entry of the first row is the letter (``L``, ``H`` or ``U``) of the column-density-dependent attenuation model instead of a single value of :math:`\zeta`, and the script reports that letter. With ``CRATTENUATION=2`` the code does not write the first row; the script then reports only the resolution and the box size. For one-dimensional models the resolution and size are zero and are reported as such.

**Cell information.** The density, gas and dust temperatures and the FUV intensity of the selected cell, the abundances of a few key species (H\ :sub:`2`, H, CO, C\ :sup:`+`, e\ :sup:`-`), and a charge-neutrality check (sum of cation abundances minus anions minus electrons, which should be close to zero).

**Elemental abundances.** Each species name in ``PREFIX.species`` is decomposed into its elements (e.g. ``HCO+`` → H + C + O, ``CH3OH`` → C + 4 H + O), and the abundance of each element is obtained by summing the abundances of all species that contain it, weighted by the number of atoms. For example, the total carbon abundance is

.. math::

   \frac{n({\rm C})}{n_{\rm H}} = x({\rm C^+}) + x({\rm C}) + x({\rm CO}) + x({\rm CH}) + 2\,x({\rm C_2}) + \dots

For each element the script prints the abundance relative to total hydrogen, :math:`X/{\rm H}`, and in the logarithmic form :math:`12+\log_{10}(X/{\rm H})`, together with the main species carrying that element. Ice species (names starting with ``#``) are included in the totals and reported separately as a gas/ice split. Electrons are excluded; grain or PAH species, if present in the network, are listed but not counted. Any species name that cannot be parsed is flagged with a warning and excluded from the totals.

Because the chemistry conserves the total number of nuclei, the elemental abundances are the same in every cell and equal the initial elemental abundances of the model (see :doc:`species`). The total hydrogen, :math:`x({\rm H}) + 2\,x({\rm H_2}) + \dots`, is therefore always close to 1. Using ``--cell`` to check a second cell is a quick way to confirm this.

Example
-------

.. code-block:: console

   $ python3 model_summary.py sims/Freeze3D
   ==============================================================================
   3D-PDR model summary : sims/Freeze3D
   ==============================================================================
   FUV field G0     : 10  (Draine units)
   CR ionization    : attenuated CR model "L" (column-dependent zeta, CRATTENUATION=1)
   Dust-to-gas      : 1  (x Solar; also scales A_V/N_H)
   v_turb           : 1 km/s
   Resolution       : 64 x 64 x 64 cells
   Box size         : 16 x 16 x 16  pc
   Source           : sims/Freeze3D.pdr.fin
   Species          : 113 in .species (3D layout)
   Cell used        : cell 1 (idx 1): n=78.1 cm^-3, Tgas=142.4 K, Tdust=19.45 K, UV=6.877
   ------------------------------------------------------------------------------
     x(H2) = 4.3458e-02  x(H) = 9.1300e-01  x(CO) = 2.6892e-11  x(C+) = 1.3999e-04  x(e-) = 2.4452e-04
     net charge check: sum(ions)-sum(anions)-x_e = -8.392e-09
   ------------------------------------------------------------------------------
   Total H (nuclei per n_H, should be ~1) = 1   [gas 1, ice 0]
   Elemental abundances relative to total H (gas+ice), with gas/ice split:
   El            X/H   12+log       gas X/H     ice X/H   main carriers
   H      1.0000e+00   12.000    1.0000e+00  0.0000e+00   H 91%, H2 9%, H+ 0%
   He     1.0000e-01   11.000    1.0000e-01  0.0000e+00   He 100%, He+ 0%, HeH+ 0%
   O      3.0000e-04    8.477    3.0000e-04  1.2229e-26   O 100%, O+ 0%, OH+ 0%
   C      1.4000e-04    8.146    1.4000e-04  1.2229e-26   C+ 100%, C 0%, CO 0%
   N      5.6500e-05    7.752    5.6500e-05  6.2200e-30   N 100%, N+ 0%, NO+ 0%
   S      3.5100e-06    6.545    3.5100e-06  0.0000e+00   S+ 100%, S 0%, SO+ 0%
   Mg     2.7000e-06    6.431    2.7000e-06  0.0000e+00   Mg+ 100%, Mg 0%
   ==============================================================================

.. note::

   Only the elements present in the chemical network are listed. The ``.species`` file contains only the species names, so the script cannot compare against the initial abundances directly; it recovers them instead from the conserved elemental totals of the selected cell.
