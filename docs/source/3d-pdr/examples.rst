Examples
========

One-dimensional Example Run
---------------------------

To run the benchmark test, execute the following sequence from your ``3D-PDR/`` home directory:

.. code-block:: bash

    cd templates
    cp config.mk makefile ../src
    cp params.dat ../
    cd ..
    cd src
    make clean; make
    cd ..
    ./3DPDR

The sample 1D model features:

- Uniform density cloud: :math:`n_{\rm H}=10^3\,{\rm cm}^{-3}`
- Visual extinction: :math:`A_{\rm V}=10\,{\rm mag}`
- Radiation field: :math:`\chi/\chi_0=10\,{\rm Draine}`
- Cosmic-ray ionization rate: :math:`\zeta_{\rm CR}=10^{-16}\,{\rm s}^{-1}`
- Solar metallicity

**GNUPLOT Commands for Visualization:**

.. code-block:: bash

    set log
    plot 'test.pdr.fin' u 3:4 w l t 'Gas temperature'
    plot 'test.pdr.fin' u 3:39 w l t 'H2', '' u 3:40 w l t 'HI'
    plot 'test.pdr.fin' u 3:19 w l t 'C+', '' u 3:33 w l t 'C', '' u 3:36 w l t 'CO'

These commands generate:

- HI-to-H₂ transition
- C⁺-C-CO transition  
- Gas temperature profile
- CO(1-0) radiation temperature vs. column density

Three-dimensional Example (32³ cells)
-------------------------------------

**Setup and Execution:**

.. code-block:: bash

    cp templates/3D/config.mk templates/3D/makefile src/
    cp templates/3D/params.dat ./
    cd src/
    make

**Configure OpenMP threads (optional):**

.. code-block:: bash

    export OMP_NUM_THREADS=8
    ./3DPDR

.. note::
   Omitting the ``export`` command uses all available CPU cores.

Visualization
~~~~~~~~~~~~~

Convert ``.fin`` files to HDF5 format using the converter in ``sims/convert_fin2h5``.

**Required parameters in ``m_parameters.F90``:**

- ``F90``: Fortran compiler (e.g., ``gfortran`` or ``h5fc``)
- ``HDF_DIR``: Path to HDF5 directory
- ``coo``: Number of coolants (from ``params.dat``)
- ``nspec``: Number of species (lines in ``chemfiles/species_reduced.d``)
- ``nrays``: Number of HEALPix rays (default: 12)
- ``cnlev``: Energy levels (line 6 in coolant file, e.g., 41 for ``12co.dat``)
- ``nxc/nyc/nzc``: Grid dimensions (line 1 in density distribution)
- ``xlx/yly/zlz``: Domain size (line 2 in density distribution)
- ``outname``: Output prefix (from ``params.dat``)

**Compile and run converter:**

.. code-block:: bash

    make
    ./convert

Column Density Maps
~~~~~~~~~~~~~~~~~~~

Use ``cd.py`` to plot column densities along coordinate axes. Species are identified by numerical suffixes matching ``chemfiles/species_reduced.d`` (e.g., abundance011 for C⁺).

**Example for C⁺ column density:**

.. code-block:: bash

    python3 cd.py --rho rho --abundance abundance011 --output coldens --visualize test.pdr.h5

This generates a column density map for C⁺ similar to the example shown in the documentation.
