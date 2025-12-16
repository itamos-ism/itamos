The Makefile
============

Edit the ``makefile`` in ``RT-synth/``:

First section (compilers):
~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``FC``: Fortran compiler (e.g., ``gfortran``)
* ``HDF5_DIR``: Specify the hdf5 installation directory
* ``HDF5_INCLUDE``: Do not change this unless you manually specify the /include directory
* ``HDF5_LIB``: Do not change this unless you manually specify the /lib directory
* ``LIBS``: Default flags
* ``FFLAGS``: Debugging flags
* ``HDF5_CFLAGS``: Optional CFLAGS
* ``HDF5_LDLIBS``: Optional LDLIBS

Second section (RT parameters):
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* ``HDF5``: Flag to handle HDF5 files (1) or ascii files (0)
* ``VELOCITY``: Set to 1 if velocity file exists (creates velocity-integrated maps)
* ``OPENMP``: Run in OpenMP parallelization (1) or in serial (0)
* ``DUST``: Include dust contribution (1) or not (0)
* ``BACKGROUND``: Remove background (1) or not (0)
* ``CRATTENUATION``: Flag to produce a density-weighted CR map (1) or not (0), provided you have performed the appropriate 3D-PDR simulation
* ``HI21CM``: Calculate the HI 21cm emission (1) or not (0)
* ``MULTILINES``: Compute all default lines (1) or single line (0)
* ``MULTIDIRECTION``: Compute all viewing angles (1) or single direction (0)
* ``CDONLY``: Calculate the column densities only and terminate RT-synth (1) or proceed with RT calculation (0)
