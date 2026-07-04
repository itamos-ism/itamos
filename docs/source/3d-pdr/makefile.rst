The makefile
============

.. important::
   The required ``config.mk``, ``makefile``, and ``params.dat`` files are provided as templates. To set up your simulation:

   1. Copy ``config.mk`` and ``makefile`` from ``3D-PDR/templates/1D/`` or ``3D-PDR/templates/3D/`` into your ``src/`` directory, based on your simulation type (1D or 3D).
   2. Copy the ``params.dat`` file from the same template directory into the ``3D-PDR/`` home directory.

The main flags of the code are specified in the ``config.mk`` file.
This file is then called within the ``makefile``.

In ``config.mk``, the flags are divided into three main sections:

1. **Compiler options**
2. **Chemistry and density options**
3. **Convergence and numerical robustness options**


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

  .. note::

     ``PYWRAP = 1`` is not compatible with ``DIMENSIONS = 3``.


Second Section (Chemistry and Density Options)
------------------------------------------------

.. code-block:: text

    DIMENSIONS        = 1
    RAYTHEIA          = 0
    XYZ               = 0
    NETWORK           = REDUCED
    DUST              = HTT91
    GUESS_TEMP        = 1
    THERMALBALANCE    = 1
    FORCECONVERGENCE  = 1
    GRAINRECOMB       = 0
    SUPRATHERMAL      = 0
    H2FORM            = CT02
    CRATTENUATION     = 0

- **DIMENSIONS**:
  Specifies the dimensionality of the model.

  - ``DIMENSIONS = 1`` — for one-dimensional models.
  - ``DIMENSIONS = 3`` — for three-dimensional models.

  .. note::

     Only ``1`` and ``3`` are valid; any other value stops the build.

- **RAYTHEIA**:
  Activates the new ray-tracing algorithm implemented for 3D models in 3D-PDR.
  This self-consistent algorithm performs fast ray tracing to compute column densities, line emission, and escape probabilities.

  - ``RAYTHEIA = 1`` — uses memory-optimized mode (slower but minimal RAM usage).
  - ``RAYTHEIA = 2`` — faster mode without memory optimization (higher RAM usage; not recommended above 64³ resolution).
  - ``RAYTHEIA = 0`` — uses the original method from `Bisbas et al. (2012) <https://ui.adsabs.harvard.edu/abs/2012MNRAS.427.2100B/abstract>`_.

  .. note::

     For 1D models, always set ``RAYTHEIA = 0`` (``DIMENSIONS = 1`` together with ``RAYTHEIA = 1`` stops the build).

- **XYZ**:
  Defines the format of the 3D density distribution (use ``XYZ = 0`` for 1D models, and note that ``DIMENSIONS = 1`` together with ``XYZ = 1`` stops the build).

  - ``XYZ = 0`` — z changes fastest.
  - ``XYZ = 1`` — x changes fastest.

  See :doc:`ics` for more details.

- **NETWORK**:
  Selects the chemical network based on the UMIST database.

  - ``REDUCED`` — 33 species, 331 reactions.
  - ``MEDIUM`` — 78 species, 1322 reactions (due to B. Gaches).
  - ``FULL`` — 215 species, 2926 reactions.
  - ``MYNETWORK`` — a user-supplied network. Provide your own ``odes_mynetwork.c`` and the matching ``species_mynetwork.d`` / ``rates_mynetwork.d`` files in ``chemfiles/``.

  More complex networks increase runtime. See :doc:`species` for how to specify the initial elemental abundances.

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

      For isothermal models, also set ``GUESS_TEMP = 0`` and specify temperature in ``params.dat`` (the combination ``THERMALBALANCE = 0`` with ``GUESS_TEMP = 1`` stops the build).

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

  - ``CT02`` — Cazaux & Tielens (`2002 <https://ui.adsabs.harvard.edu/abs/2002ApJ...575L..29C/abstract>`_, `2004 <https://ui.adsabs.harvard.edu/abs/2004ApJ...604..222C/abstract>`_).
  - ``SIMPLE`` — simplified function (see ``makefile`` comments).
  - ``R07`` — `Röllig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_ benchmark function.

- **CRATTENUATION**:
  Enables cosmic-ray attenuation, replacing the constant ionization rate with a column-density-dependent one.

  - ``1`` — uses Padovani et al. (`2018 <https://ui.adsabs.harvard.edu/abs/2018A%26A...614A.111P/abstract>`_; `2024 <https://ui.adsabs.harvard.edu/abs/2024A%26A...682A.131P/abstract>`_) low/high/ultra models. Replace the CRIR value in ``params.dat`` with ``L``, ``H`` or ``U``.
  - ``2`` — uses a softened power-law model :math:`\zeta(N) = \zeta_0 \times (1 + N/N_0)^\alpha`. Specify :math:`\zeta_0`, :math:`N_0`, and :math:`\alpha` in ``params.dat`` instead of the CRIR — see :doc:`params` for the exact line order.
  - ``0`` — uses a constant CR ionization rate throughout the cloud (default).

  .. note::

     Any value of ``CRATTENUATION`` other than ``0``, ``1`` or ``2`` stops the build.


Third Section (Convergence and Numerical Robustness Options)
----------------------------------------------------------------

.. code-block:: text

    #Convergence of ODEs
    CHEMSTEADY        = 1
    CHEMRETRY         = 1
    #Ng acceleration
    NGACCEL           = 1
    NGCYCLE           = 1
    NGRELAX           = 1
    #Thermal balance
    ILLINOIS          = 1
    REBRACKET         = 1
    #Escape probability
    SOBOLEV           = 1
    ###
    RESTART           = 0
    OUTRAYINFO        = 0
    CHEMANALYSIS      = 0
    LEVMAX            = 0

These flags control the numerics of the chemical and thermal-balance solvers. They were added to address convergence and stability issues found in stiff chemistry, optically thick lines, and stretched grids; all are recommended unless otherwise noted, and are set to ``1`` by default.

*Convergence of ODEs*

- **CHEMSTEADY**:
  Continues each chemical integration in doubling-time blocks (up to 128× the evolution time) until no species changes by more than 1% over a doubling.

  - ``1`` — switch on (recommended). Without this, the chemical age of a grid point depends on how many chemistry calls it received before its thermal balance converged, leaving unphysical abundance jumps between neighbouring points.
  - ``0`` — switch off; each call advances by the evolution time only.

- **CHEMRETRY**:
  Re-integrates a grid cell whose CVODE chemistry solve fails (e.g. ``error test failed ... |h|=hmin at t=0`` for very stiff chemistry under extreme conditions), restarting from the saved initial abundances with a forced, shrinking initial step (up to 4 retries).

  - ``1`` — switch on (recommended). Recovers the few stiff cells that CVODE's automatic initial-step choice over-reaches; the first attempt is unchanged, so normal cells are unaffected.
  - ``0`` — switch off; failed cells are abandoned at best effort.

*Ng acceleration*

- **NGACCEL**:
  Enables Ng (1974) acceleration of the level-population iterations.

  - ``1`` — switch on (recommended); gives faster convergence.
  - ``0`` — switch off.

- **NGCYCLE**:
  Dynamic per-point damping of oscillating (flip-flop) level populations: whenever an update reverses the previous one, the last two iterates are averaged. Acts on each point/coolant individually, every iteration, as soon as the oscillation appears, instead of stalling until the ``FORCECONVERGENCE`` averaging at level iteration 75.

  - ``1`` — switch on (recommended).
  - ``0`` — switch off.

  .. note::

     Requires ``NGACCEL = 1``.

- **NGRELAX**:
  Adaptive per-point under-relaxation of persistently oscillating level populations: a per-cell counter shrinks the relaxation weight (``1/(2+n)``) while a cell keeps reversing, damping the violent CO(1-0) flip-flop near the inversion boundary (excitation temperature → ∞) that a fixed weight of 1/2 cannot, in hot CR-heated cells at densities near the critical density.

  - ``1`` — switch on (recommended, especially for 3D models). Supersedes the ``NGCYCLE`` damping when both are on.
  - ``0`` — switch off.

  .. note::

     Requires ``NGACCEL = 1``.

*Thermal balance*

- **ILLINOIS**:
  Uses the Illinois-modified regula-falsi method (in ln(T)) for the thermal-balance temperature refinement, replacing plain bisection.

  - ``1`` — switch on (recommended); gives faster convergence.
  - ``0`` — switch off; uses plain bisection.

- **REBRACKET**:
  When the thermal-balance search stalls with a large residual imbalance (the bracket is confined away from the root by early, not-yet-relaxed cooling rates), re-opens the bracket and resumes the search (up to 3 attempts per point) instead of forcing convergence.

  - ``1`` — switch on (recommended).
  - ``0`` — switch off; forces convergence on stall.

*Escape probability*

- **SOBOLEV**:
  Uses Sobolev (ALI-like) net radiative rates (:math:`A\beta` and :math:`\beta B \bar{B}`) instead of the mean-field expression :math:`(1-\beta)S + \beta \bar{B}`. Both give the same solution, but Sobolev removes the lagged self-coupling that makes optically thick lines falsely appear converged (less than 1% change per iteration while still far from equilibrium), which otherwise leaves spurious cooling at low gas temperature.

  - ``1`` — switch on (recommended).
  - ``0`` — switch off; uses the original mean-field iteration.

*Output, restart and diagnostics*

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

- **CHEMANALYSIS**:
  Outputs a per-point chemical analysis (formation/destruction rates of selected species).

  - ``1`` — switch on. Not recommended for 3D models!
  - ``0`` — switch off (default).

- **LEVMAX**:
  Enables the runtime ``-lmax=N`` command-line flag, which caps the number of energy levels loaded per coolant.

  - ``1`` — switch on. See :ref:`levmax-flag` below for full details and usage.
  - ``0`` — switch off (default); ``-lmax`` is an unrecognised option and all levels from each coolant's LAMDA file are used.


.. _levmax-flag:

The LEVMAX flag
----------------

``LEVMAX`` is a compile-time switch that enables an optional *runtime* command-line flag, ``-lmax=N``, which caps the number of rotational/energy levels loaded from each coolant's LAMDA file at ``N``.

Why this matters for RAM
~~~~~~~~~~~~~~~~~~~~~~~~~

The level-population arrays for each coolant (the ``line(N,N)`` emissivity matrix plus the population, solution, relative-change, and Ng-history vectors) scale as :math:`N^2` per grid cell, where :math:`N` is the number of energy levels. For coolants with many levels — e.g. ``12co.dat`` has 41 levels — this quadratic scaling dominates the total memory footprint of a large 3D model. Capping the number of *used* levels at a smaller ``N`` (while still reading the full collision-rate tables needed for accurate excitation at the levels that matter) can cut coolant RAM by a factor of 3–6, typically with less than 0.1% change in the resulting gas temperature.

Coolants that already have few levels (e.g. ``c+.dat``, ``12c.dat``, ``16o.dat``, all ≤5 levels) are unaffected by any ``N ≥ 5``.

Usage
~~~~~

First, ensure ``LEVMAX = 1`` in ``config.mk`` and recompile. Then pass the cap as a command-line argument when running the code:

.. code-block:: console

   $ ./3DPDR -lmax=15

This loads at most 15 levels per coolant. Omitting the flag (or compiling with ``LEVMAX = 0``) uses all levels from each LAMDA file — the original, backward-compatible behaviour.

.. note::

   The active ``LEVMAX`` value is also recorded in the ``.RTspop.fin`` output (as ``LMAX=N``) so that downstream tools, such as :doc:`../rt-tool/RT-tool`, know how many levels were used when the model was run. See :doc:`outputs` for details.

Estimating the RAM savings beforehand
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before committing to a large 3D run, use :doc:`ramestimate` with the ``--lmax`` option to compare the estimated RAM with and without a cap, e.g.:

.. code-block:: console

   $ python3 ram_estimate.py --res 256 --raytheia 1 --lmax 13

This reports the per-coolant level counts actually used after the cap, alongside the resulting RAM breakdown, so you can choose an ``N`` that fits your available memory before running the model.
