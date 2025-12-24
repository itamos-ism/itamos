=============================
Reaction Rate Calculations
=============================

The ``CALCULATE_REACTION_RATES`` subroutine evaluates the rate coefficients for **all reactions in the
chemical network** at the local gas temperature, dust temperature,
radiation field, visual extinction, and column densities. The resulting
rates are used by the chemical solver to advance the abundances.

--------------------------------
Scope and General Philosophy
--------------------------------

- Reaction rates depend on **local physical conditions**:
  gas temperature, dust temperature, radiation field, extinction,
  column densities, electron density, and cosmic-ray ionization rate.
- Multiple entries for the same reaction are allowed in the rate file
  (``DUPLICATE`` flag) and are activated based on their valid temperature
  ranges.
- Photodissociation of H\ :sub:`2` and CO, and photoionization of C and S,
  are treated **explicitly with shielding and ray tracing**, rather than
  via simple analytic fits.
..
  - X-ray–induced reactions follow the treatment of  Meijerink & Spaans (2005).

--------------------------------
Inputs and Outputs
--------------------------------

**Key inputs**

- ``TEMPERATURE``: gas temperature (K)
- ``DUST_TEMPERATURE``: dust temperature (K)
- ``RAD_SURFACE(J)``: incident radiation field along ray ``J``
- ``AV(J)``: visual extinction along ray ``J``
- ``COLUMN_NH2``, ``COLUMN_NHD``, ``COLUMN_NCO``, ``COLUMN_NC``,
  ``COLUMN_NS``: column densities along rays
- ``ALPHA``, ``BETA``, ``GAMMA``: Arrhenius-type reaction parameters
- ``RTMIN``, ``RTMAX``: temperature validity range
- ``ZETALOCAL``: local cosmic-ray ionization rate
- ``nelectron``, ``density``: electron and gas densities

**Outputs**

- ``RATE(I)``: rate coefficient for reaction ``I``
- Stored indices for key reactions:
  ``NRGR``, ``NRH2``, ``NRHD``, ``NRCO``, ``NRCI``, ``NRSI``

--------------------------------
Thermal Gas-Phase Reactions
--------------------------------

Most two-body gas-phase reactions are computed using a modified
Arrhenius form:

.. math::

   k(T) = \alpha \left(\frac{T}{300}\right)^\beta
          \exp\!\left(-\frac{\gamma}{T}\right)

Important implementation details:

- Reactions with **large negative activation energies**
  (:math:`\gamma < -200`) are suppressed below their minimum valid
  temperature.
- For duplicated reactions, the code selects the appropriate entry
  based on the current temperature.
- Rates are capped at unity (except for grain reactions) to maintain
  numerical stability.

--------------------------------
H\ :sub:`2` Formation on Grains
--------------------------------

The formation of molecular hydrogen on dust grains is treated
separately and does **not** use the standard Arrhenius form.

Depending on compile-time options, the rate follows:

- `Cazaux & Tielens (2002) <https://ui.adsabs.harvard.edu/abs/2002ApJ...575L..29C/abstract>`_; see detailed description :ref:`here <h2form>`,
- a simplified temperature-dependent prescription, or
- the rate of `Röllig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_.

The selected rate depends explicitly on both gas and dust temperatures.

The corresponding reaction index is stored as:

- ``NRGR`` — H\ :sub:`2` grain formation

--------------------------------
PAH-Related Reactions
--------------------------------

Reactions involving PAHs (neutral or charged) follow the prescription
of Wolfire et al. (2003, 2008).

The rate is given by:

.. math::

   k = \alpha \left(\frac{T}{100}\right)^\beta \phi_{\rm PAH}

with a constant PAH efficiency factor
:math:`\phi_{\rm PAH} = 0.4`.

--------------------------------
Suprathermal Ion–Neutral Reactions
--------------------------------

Optionally, suprathermal chemistry is included following
Visser et al. (2009).

For ion–neutral reactions at low visual extinction, the effective
temperature is increased by a contribution proportional to the Alfvén
speed:

.. math::

   T_{\rm eff} = T + \Delta T_{\rm sup}

This enhancement applies only when the local extinction is below a
critical value ``Av_crit``.

--------------------------------
Photoreactions
--------------------------------

Photoreaction rates are computed by **explicit ray integration**:

.. math::

   k = \sum_J \alpha \, G_J \, e^{-\gamma A_{V,J}}

where the sum runs over all rays *J*.

### Special Cases

The following reactions are treated with dedicated shielding functions:

- **H\ :sub:`2` photodissociation**  
  Computed using ``H2PDRATE`` and self-shielding by H\ :sub:`2`.

- **HD photodissociation**  
  Treated analogously to H\ :sub:`2`.

- **CO photodissociation**  
  Includes shielding by CO and H\ :sub:`2` via ``COPDRATE``.

- **C and S photoionization**  
  Computed with ``CIPDRATE`` and ``SIPDRATE``, including temperature
  dependence and column-density shielding.

The indices of these reactions are stored for later use
(``NRH2``, ``NRHD``, ``NRCO``, ``NRCI``, ``NRSI``).

--------------------------------
Cosmic-Ray Ionization
--------------------------------

Primary cosmic-ray ionization rates are proportional to the local
ionization rate:

.. math::

   k = \alpha \, \zeta_{\rm local}

Duplicate reactions are again selected by temperature range.

--------------------------------
Cosmic-Ray–Induced Photoreactions
--------------------------------

Secondary UV photons generated by cosmic rays drive additional
photoreactions. Their rates follow:

.. math::

   k = \alpha \, \zeta_{\rm local}
       \left(\frac{T}{300}\right)^\beta
       \frac{\gamma}{1 - \omega}

where :math:`\omega` is the dust albedo.

--------------------------------
Freeze-Out onto Dust Grains
--------------------------------

Neutral species and singly charged ions can freeze onto dust grains.

The rate depends on:

- the thermal velocity of the species,
- Coulomb focusing (for ions),
- a fixed sticking probability (0.3).

The general scaling is:

.. math::

   k \propto \sqrt{T} \, C_{\rm ion} \, S

--------------------------------
Desorption Processes
--------------------------------

### Cosmic-Ray Desorption

Desorption due to transient grain heating by cosmic rays follows
Roberts et al. (2007), with a fixed cosmic-ray flux and a temperature-
dependent yield.

### Photodesorption

Photodesorption rates depend on:

- dust temperature (via the yield),
- attenuated FUV flux along each ray.

### Thermal Desorption

Thermal evaporation from grains follows
Hasegawa, Herbst & Leung (1992) and depends exponentially on the
dust temperature:

.. math::

   k \propto \exp\!\left(-\frac{E_b}{T_{\rm dust}}\right)

--------------------------------
Grain-Surface Reactions
--------------------------------

Grain-mantle reactions are treated with constant rates provided
directly in the reaction file:

.. math::

   k = \alpha

--------------------------------
Grain-Assisted Recombination
--------------------------------

Optionally, grain-assisted recombination of H\ :sup:`+`, He\ :sup:`+`,
and C\ :sup:`+` is included following Gong et al. (2017).

These rates depend on:

- electron density,
- gas density,
- local FUV radiation field,
- dust charging parameter :math:`\Psi`.

--------------------------------
Numerical Safeguards
--------------------------------

To ensure numerical stability:

- Negative rates trigger a fatal error.
- Rates below :math:`10^{-99}` are set to zero.
- Gas-phase rates are capped at unity.
- Grain-surface and desorption reactions are allowed to exceed unity.

--------------------------------
Summary
--------------------------------

The reaction rate module in 3D-PDR:

- Supports a wide range of gas-phase, grain-surface, photo-, and
  cosmic-ray–driven reactions;
- Includes detailed ray-based attenuation and self-shielding;
- Allows multiple temperature-dependent rate entries per reaction;
- Incorporates optional suprathermal and grain-assisted processes;
- Ensures physically consistent and numerically stable rate
  coefficients across extreme PDR conditions.
