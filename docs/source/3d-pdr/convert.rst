Convert to hdf5
===============

Convert ``.fin`` files to HDF5 format using the converter in ``sims/convert_fin2h5``.

**Required flags in ``makefile``:**
Edit the makefile and insert the following information:

- ``F90``: Fortran compiler (e.g., ``gfortran`` or ``h5fc``)
- ``HDF_DIR``: Path to HDF5 directory

**Required parameters in ``m_parameters.F90``:**
Edit this file and insert the following information:

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
