The params.dat
==============

.. important::
   The required ``config.mk``, ``makefile``, and ``params.dat`` files are provided as templates. To set up your simulation:
 
   1. Copy ``config.mk`` and ``makefile`` from ``templates/1D/`` or ``templates/3D/`` into your ``src/`` directory, based on your simulation type (1D or 3D).
   2. Copy the ``params.dat`` file from the same template directory into the **3D-PDR** home directory.

The ``params.dat`` file configures input/output settings, PDR environmental parameters, thermal balance options, coolants, and other technical parameters for 3D-PDR simulations. 

Custom parameter files (e.g., ``paramsUser.dat``) can be specified using the command-line options ``-p=`` or ``--params``:

.. code-block:: bash

    ./3DPDR -p=paramsUser.dat

.. note::
   Parameter filenames must not exceed 50 characters.

Input/Output - Densities
------------------------

::

    1) ics                        !ICs directory
    2) 1Dn30.dat                  !ICs file
    3) sims                       !Output directory
    4) test                       !Output prefix

* **Entry 1**: Directory containing initial conditions and density distributions.
* **Entry 2**: Input file specifying the 1D density distribution.
* **Entry 3**: Output directory where simulation results are stored.
* **Entry 4**: Prefix for output files (e.g., results will be saved as ``sims/test.pdr.fin``).

PDR Parameters
--------------

::

    5)  10                         !G0 (in Draine field units) -x to +x in 1D
    6)  1.0E-16                    !Cosmic-ray ionization rate (s^-1)
    7)  0.0                        !Xray flux (erg/cm2/s)
    8)  1.0                        !Dust-to-gas normalized to 1e-2
    9)  1.0                        !Turbulent velocity (km/s)
    10) 1e7                        !End time (yr)
    11) 1.0E-5                     !Grain radius (cm)
    12) 6.289e-22                  !Av conversion factor
    13) 3.02                       !UV factor
    14) 0                          !Redshift (for CMB temperature)
    15) 0.7                        !Av critical (SUPRATHERMAL switch)
    16) 3.3                        !Alfven velocity (km/s) (SUPRATHERMAl switch)

* **Entry 5**: FUV radiation field intensity normalized to `Draine (1978) <https://ui.adsabs.harvard.edu/abs/1978ApJS...36..595D/abstract>`_ spectral shape.
  
  * For ``DIMENSIONS = 1``: one-sided radiation field
  * For ``DIMENSIONS = 3``: isotropic radiation field (each of 12 HEALPix rays receives intensity/12)

* **Entry 6**: Cosmic-ray ionization rate (:math:`{\rm s}^{-1}`), fixed throughout the cloud.
* **Entry 7**: X-ray flux (:math:`\rm erg/cm^2/s`). Not recommended to change in this version.
* **Entry 8**: Dust-to-gas ratio normalized to :math:`10^{-2}`. For non-solar metallicities, modify this value and specify elemental abundances separately.
* **Entry 9**: Microturbulent velocity (km/s) for turbulent heating.
* **Entry 10**: Chemical evolution end time (years). Default 10 Myr ensures chemical equilibrium; adjustable for younger cloud studies.
* **Entry 11**: Grain radius (cm).
* **Entry 12**: Visual extinction conversion factor from `Roellig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_.
* **Entry 13**: UV attenuation factor from `Roellig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_.
* **Entry 14**: Redshift value determining CMB temperature.
* **Entry 15**: Critical Av for ``SUPRATHERMAL = 1`` switch (Av threshold where suprathermal CO formation via CH⁺ becomes negligible).
* **Entry 16**: Alfvén velocity (km/s) for ``SUPRATHERMAL = 1`` switch.

.. note::
   Entries 15 and 16 are only used when compiled with ``SUPRATHERMAL = 1``.

Softened-Power-Law Cosmic Ray Attenuation
-----------------------------------------

For softened-power-law cosmic ray attenuation, modify the parameter structure as follows:

::

    5)  10                         !G0 (in Draine field units) -x to +x in 1D
    6a) 1.0E-16                    !Surface cosmic-ray ionization rate, zeta0 (s^-1)
    6b) 1.0E21                     !Normalizing column density, N0
    6c) -1.0                       !Attenuation slope, alpha
    .... continued as normal

Add entries 6b and 6c as new lines in your parameter file.

Numerical Parameters
--------------------

Ray-tracing, ODE Solver, and Chemistry Iterations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

        ===========================|
        Ray-tracing parameters     |
        ===========================|
    17) 0                          !HEALPix level of refinement
    18) 1.3                        !Theta critical (0<phi<pi/2)
        ===========================|
        ODE Solver parameters      |
        ===========================|
    19) 1.0D-8                     !relative abundance tolerance
    20) 1.0D-30                    !absolute abundance tolerance
        ===========================|
        Chemistry iterations       |
        ===========================|
    21) 8                          !First set of Chemical Iterations
    22) 6000                       !Total iterations

.. warning::
   Modify these values only if you understand their impact on numerical stability and convergence.

Thermal Balance Parameters
--------------------------

::

    23) 40.0                       !Gas temperature for isothermal models
    24) 10.0                       !Floor temperature (if <Tcmb it's set to Tcmb)
    25) 30000.0                    !Maximum allowed gas temperature
    26) 20.0                       !Dust temperature for isothermal dust models
    27) 0.005                      !Fcrit (% Accuracy)
    28) 0.01                       !Tdiff (maximum temperature difference)

* **Entry 23**: Gas temperature for isothermal models (requires ``THERMALBALANCE = 0`` and ``GUESS_TEMP = 0`` in makefile).
* **Entry 24**: Minimum gas temperature floor. Automatically raised to CMB temperature if lower.
* **Entry 25**: Maximum allowed gas temperature (not recommended to change).
* **Entry 26**: Dust temperature for isothermal dust models (requires ``DUST = 0`` in makefile).
* **Entry 27**: Thermal balance accuracy tolerance (modify with caution).
* **Entry 28**: Maximum temperature difference for thermal balance convergence (modify with caution).

Coolant Files
-------------

::

    29) 12co.dat                   !CO Do not change the following order
    30) 12c+.dat                   !CII
    31) 12c.dat                    !CI
    32) 16o.dat                    !OI

The default coolant list (12CO, C⁺, C, O) should maintain the specified order. Additional coolants can be appended after ``16o.dat``, though this may impact computational performance.

Adding Additional Coolants
--------------------------

Including additional coolants enables study of their emission lines. **3D-PDR** computes level populations for all specified coolants, which can be used by **RT-tool** and **RT-synth** for radiative transfer calculations.

To add coolants beyond the defaults:

1. Verify the chemical network includes the desired species
2. Append coolant filenames to ``params.dat`` after the default entries

Example:
::

    29) 12co.dat                   !CO Do not change the following order
    30) 12c+.dat                   !CII
    31) 12c.dat                    !CI
    32) 16o.dat                    !OI
    33) hco+.dat                   !HCO⁺
    34) cs.dat                     !CS

Custom Coolant Data Files
~~~~~~~~~~~~~~~~~~~~~~~~~

For coolants not available in ``chemfiles/``, download from the `LEIDEN LAMBDA database <https://home.strw.leidenuniv.nl/~moldata/>`_ and modify the file header:

Original LEIDEN format:
::

    !MOLECULE
    O (neutral atom)
    !MOLECULAR WEIGHT
    16.0
    !NUMBER OF ENERGY LEVELS
    5

Modified for 3D-PDR:
::

    !MOLECULE
    O                            !Name matching species_NETWORK.d
    !MOLECULAR WEIGHT
    16.0
    !NUMBER OF ENERGY LEVELS
    5 27                         !Add max "COLL TEMPS" count

Multiple coolants can be added by extending the parameter file list. Each additional coolant increases computation time.
