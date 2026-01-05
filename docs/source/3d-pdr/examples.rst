Examples
========

.. note::
   There is a python wrapper that you may consider in case you prefer to run 1D models from a python interface. Please refer to the `HandsOn_1Dexamples.ipynb` jupyter notebook available on the `3D-PDR Github link <https://github.com/itamos-ism/3D-PDR>`_

One-dimensional Example Run
---------------------------

To run the benchmark test, use the ``params.dat`` file found in ``templates/1D`` within the ``3D-PDR/`` home directory. While in ``3D-PDR/`` execute the following commands:

.. code-block:: bash

    cp templates/1D/config.mk templates/1D/makefile src/
    cp templates/1D/params.dat .
    cd src/
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
While in the ``3D-PDR/`` home directory execute the following commands:

.. code-block:: bash

    cp templates/3D/config.mk templates/3D/makefile src/
    cp templates/3D/params.dat .
    cd src/
    make clean; make

**Configure OpenMP threads (optional):**

.. code-block:: bash

    export OMP_NUM_THREADS=8
    ./3DPDR

.. note::
   Omitting the ``export`` command uses all available CPU cores.

Visualization
~~~~~~~~~~~~~

Convert the outputs to h5 using the ``convert`` script (see documentation :ref:`sec-convert`).


Column Density Maps
~~~~~~~~~~~~~~~~~~~

Use ``cd.py`` to plot column densities along coordinate axes. Species are identified by numerical suffixes matching ``chemfiles/species_reduced.d`` (e.g., abundance011 for C⁺).

**Example for C⁺ column density:**

.. code-block:: bash

    python3 cd.py --rho rho --abundance abundance011 --output coldens --visualize test.pdr.h5

This generates a column density map for C⁺ similar to the example shown in the documentation.
