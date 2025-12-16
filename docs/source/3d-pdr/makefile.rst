The makefile
============

.. important::
   The required ``config.mk``, ``makefile``, and ``params.dat`` files are provided as templates. To set up your simulation:
 
   1. Copy ``config.mk`` and ``makefile`` from ``templates/1D/`` or ``templates/3D/`` into your ``src/`` directory, based on your simulation type (1D or 3D).
   2. Copy the ``params.dat`` file from the same template directory into the **3D-PDR** home directory.

The main flags of the code are specified in the ``config.mk`` file.  
This file is then called within the ``makefile``.  

In ``config.mk``, the flags are divided into two main sections:

1. **Compiler options**
2. **Chemistry and density options**


First Section (Compiler Options)
--------------------------------

.. code-block:: text

    F90               = gfortran
    CC                = gcc
    CPPFLAGS          = -cpp
    OPENMP            = 1
    OPTIMISE          = 3
    PYWRAP            = 0

- **F90**  
  Specifies the Fortran compiler. The default compiler is ``gfortran``.  
  No other Fortran compilers have been tested, so change this only if you are willing to debug potential issues yourself.

- **CC**  
  Specifies the C compiler. For macOS users, you may need to use ``gcc-14`` or ``mp-gcc-14`` (or equivalent) instead of ``gcc``.  
  *It is strongly recommended to use GCC versions later than 7.3.0.*

- **CPPFLAGS**  
  Specifies the C preprocessor flag, which should be ``-cpp``.

- **OPENMP**  
  Controls whether the code runs in parallel (``OPENMP = 1``) or serial mode (``OPENMP = 0``).  
  It is recommended to always use ``OPENMP = 1`` unless there is a specific reason not to.

- **OPTIMISE**  
  Defines the compiler optimization level.  
  Flags 0 and 1 are for development and debugging, while 2 and 3 are used for production runs.  
  The recommended value is ``OPTIMISE = 3`` for normal use.  
  A flag 4 also exists but is not recommended, as it may lower precision and cause instability.

- **PYWRAP**  
  Enables Python wrapper compilation (``PYWRAP = 1``) for the ``py3DPDR.py`` interface.  
  See the Jupyter notebook ``HandsOn_1Dexamples.ipynb`` for usage examples.


Second Section (Chemistry and Density Options)
----------------------------------------------

.. code-block:: text

    DIMENSIONS        = 1
    RAYTHEIA          = 0
    XYZ               = 0
    NETWORK           = REDUCED
    XRAYS             = 0
    DUST              = HTT91
    GUESS_TEMP        = 1
    THERMALBALANCE    = 1
    FORCECONVERGENCE  = 1
    GRAINRECOMB       = 0
    SUPRATHERMAL      = 0
    H2FORM            = CT02
    CRATTENUATION     = 0
    RESTART           = 0
    OUTRAYINFO        = 0

- **DIMENSIONS**:  
  Specifies the dimensionality of the model.  
  
  - ``DIMENSIONS = 1`` — for one-dimensional models. 
  - ``DIMENSIONS = 3`` — for three-dimensional models.

- **RAYTHEIA**:
  Activates the new ray-tracing algorithm implemented for 3D models in 3D-PDR.  
  This self-consistent algorithm performs fast ray tracing to compute column densities, line emission, and escape probabilities.  

  - ``RAYTHEIA = 1`` — uses memory-optimized mode (slower but minimal RAM usage).  
  - ``RAYTHEIA = 2`` — faster mode without memory optimization (higher RAM usage; not recommended above 64³ resolution).  
  - ``RAYTHEIA = 0`` — uses the original method from `Bisbas et al. (2012) <https://ui.adsabs.harvard.edu/abs/2012MNRAS.427.2100B/abstract>`_.  

  .. note::

     For 1D models, always set ``RAYTHEIA = 0``.

- **XYZ**:  
  Defines the format of the 3D density distribution (use ``XYZ = 0`` for 1D models).  

  - ``XYZ = 0`` — z changes fastest.  
  - ``XYZ = 1`` — x changes fastest.  

  See the :ref:`ics3D` section for more details.

- **NETWORK**:
  Selects the chemical network based on the UMIST database.  

  - ``REDUCED`` — 33 species, 331 reactions.  
  - ``MEDIUM`` — 77 species, 1158 reactions (due to B. Gaches).  
  - ``FULL`` — 215 species, 2926 reactions.  

  More complex networks increase runtime. See the :ref:`species` section on how to specify the initial elemental abundances.

- **XRAYS**:
  Enables X-ray chemistry (currently experimental).  
  Works in limited cases for ``NETWORK = REDUCED`` only.  
  Set ``XRAYS = 0`` in all other cases.

- **DUST**:
  Determines how dust temperature is treated.  

  - ``HTT91`` — uses `Hollenbach, Takahashi & Tielens (1991) <https://ui.adsabs.harvard.edu/abs/1991ApJ...377..192H/abstract>`_ (default).  
  - ``0`` — isothermal dust. Specify the temperature in ``params.dat``.

- **GUESS_TEMP**:
  Sets the initial gas temperature used to start iterations.  

  - ``1`` — uses an approximate function based on FUV intensity (recommended).  
  - ``0`` — uniform temperature across the cloud.  

- **THERMALBALANCE**:  
  Determines whether the model solves for thermal balance or not.
  
  - ``THERMALBALANCE = 1`` — switch on thermal balance.
  - ``THERMALBALANCE = 0`` — isothermal runs.

  .. note::
   
      For isothermal models, also set ``GUESS_TEMP = 0`` and specify temperature in ``params.dat``

- **FORCECONVERGENCE**: 
  Accelerates convergence by restricting variable changes after several iterations.  
  Recommended value: ``FORCECONVERGENCE = 1``.

- **GRAINRECOMB**:  
  Enables electron recombination on dust grains.  
  ``GRAINRECOMB = 1`` uses the treatment from `Weingartner & Draine (2001) <https://ui.adsabs.harvard.edu/abs/2001ApJ...563..842W/abstract>`_.  
  Some issues remain for the ``FULL`` network.

- **SUPRATHERMAL**:
  Enables suprathermal formation of CO via CH⁺.  
  See `Visser et al. (2009) <https://ui.adsabs.harvard.edu/abs/2009A%26A...503..323V/abstract>`_ and `Bisbas et al. (2019) <https://ui.adsabs.harvard.edu/abs/2019MNRAS.485.3097B/abstract>`_.  
  Values for the critical A_V and Alfvén velocity are set in ``params.dat``.

- **H2FORM**:
  Specifies the H₂ formation recipe:  

  - ``CT02`` — Cazaux & Tielens (2002, 2004).  
  - ``SIMPLE`` — simplified function (see ``makefile`` comments).  
  - ``R07`` — `Röllig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_ benchmark function.

- **CRATTENUATION**:
  Enables cosmic-ray attenuation.  

  - ``1`` — uses `Padovani et al. (2018) <https://ui.adsabs.harvard.edu/abs/2018A%26A...614A.111P/abstract>`_ low/high models. Replace CRIR value in ``params.dat`` with *L* or *H*.  
  - ``0`` — uses constant CR ionization rate throughout the cloud.

.. - ``2`` — uses a softened power-law model :math:`\zeta(N) = \zeta_0 \times (1 + N/N_0)^\alpha`. Specify :math:`\zeta_0`, :math:`N_0`, and :math:`\alpha` in ``params.dat``

- **RESTART**  
  Allows resuming an interrupted run (``RESTART = 1``).  
  Writes a binary file ``restart.bin`` in the working directory.  
  To restart after a crash or timeout, simply rerun the code.  

  .. note::
    
     Delete any old ``restart.bin`` before starting a new model. 
     Works only with ``THERMALBALANCE = 1``.

- **OUTRAYINFO**:
  Saves information for each ray in separate files.  
  Default: ``OUTRAYINFO = 0``.
