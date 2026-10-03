=============================
Reaction Rate Calculations
=============================

The ``CALCULATE_REACTION_RATES`` subroutine evaluates the rate coefficients for **all reactions in the
chemical network** at the local gas temperature, dust temperature,
radiation field, visual extinction, and column densities. The resulting
rates are used by the chemical solver to advance the abundances.

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

- `Cazaux & Tielens (2002) <https://ui.adsabs.harvard.edu/abs/2002ApJ...575L..29C/abstract>`_; see detailed description :ref:`here <h2f>`,
- a simplified temperature-dependent prescription given by :math:`3\times10^{-18}\sqrt{T_{\rm gas}}e^{-\frac{T_{\rm gas}}{10^3 {\rm K}}}`, or
- the rate of `Röllig et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007A%26A...467..187R/abstract>`_ given by :math:`3\times10^{-18}\sqrt{T_{\rm gas}}`.

The selected rate depends explicitly on both gas and dust temperatures.

The corresponding reaction index is stored as:

- ``NRGR`` — H\ :sub:`2` grain formation

--------------------------------
PAH-Related Reactions
--------------------------------

Reactions involving PAHs (neutral or charged) follow the prescription
of Wolfire et al. (`2003 <https://ui.adsabs.harvard.edu/abs/2003ApJ...587..278W/abstract>`_, `2008 <https://ui.adsabs.harvard.edu/abs/2008ApJ...680..384W/abstract>`_).

The rate is given by:

.. math::

   k = \alpha \left(\frac{T}{100}\right)^\beta \phi_{\rm PAH}

with a constant PAH efficiency factor
:math:`\phi_{\rm PAH} = 0.4`.

--------------------------------
Suprathermal Ion–Neutral Reactions
--------------------------------

Optionally, suprathermal chemistry is included following
`Visser et al. (2009) <https://ui.adsabs.harvard.edu/abs/2009A%26A...503..323V/abstract>`_.

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

Special Cases
~~~~~~~~~~~~~

The following reactions are treated with dedicated shielding functions:

- **H2 photodissociation**  
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

Cosmic-Ray Desorption
~~~~~~~~~~~~~~~~~~~~~

Desorption due to transient grain heating by cosmic rays follows
`Roberts et al. (2007) <https://ui.adsabs.harvard.edu/abs/2007MNRAS.382..733R/abstract>`_, with a fixed cosmic-ray flux and a temperature-
dependent yield.

Photodesorption
~~~~~~~~~~~~~~~

Photodesorption rates depend on:

- dust temperature (via the yield),
- attenuated FUV flux along each ray.

Thermal Desorption
~~~~~~~~~~~~~~~~~~

Thermal evaporation from grains follows
`Hasegawa, Herbst & Leung (1992) <https://ui.adsabs.harvard.edu/abs/1992ApJS...82..167H/abstract>`_ and depends exponentially on the
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
and C\ :sup:`+` is included following `Gong et al. (2017) <https://ui.adsabs.harvard.edu/abs/2017ApJ...843...38G/abstract>`_.

These rates depend on:

- electron density,
- gas density,
- local FUV radiation field,
- dust charging parameter :math:`\Psi`.

In addition, the ``GRAINRECOMB`` compile option (see :doc:`makefile`) adds
a grain term to the electron recombination of cations computed with one of the two
treatments below.
Both non-zero options act on the electron-recombination reactions
:math:`\mathrm{X}^+ + e^- \rightarrow \ldots` of the chemical network, by adding
a grain term to the gas-phase rate coefficient. The term is written
*per electron* (divided by :math:`n(e^-)`) so that, after multiplication by
:math:`n(e^-)` in the right-hand side of the chemical ODEs,
it gives the electron-density-independent grain-collision rate
:math:`k_{\rm gr}\,n(\mathrm{X}^+)`.

``GRAINRECOMB = 1``
^^^^^^^^^^^^^^^^^^^

Grain-assisted recombination of H\ :sup:`+`, He\ :sup:`+`, C\ :sup:`+`,
Mg\ :sup:`+`, S\ :sup:`+` and Fe\ :sup:`+` follows the fitting formulae of
`Weingartner & Draine (2001) <https://ui.adsabs.harvard.edu/abs/2001ApJ...563..842W/abstract>`_
(coefficients :math:`C_0, \ldots, C_6` tabulated per ion),

.. math::

   \alpha_{\rm gr} = 10^{-14}\,C_0\,\left[1 + C_1\,\Psi^{C_2}\left(1 + C_3\,T^{C_4}\,
   \Psi^{-C_5 - C_6 \ln T}\right)\right]^{-1}\ \mathrm{cm^3\,s^{-1}},

where :math:`\Psi` is the dust charging parameter, computed from the local
FUV field of every ray (attenuated with the ray :math:`A_V`), the gas
temperature and the electron density,
and the rate is multiplied by :math:`n_{\rm H}/n(e^-)`.
The fits are normalized to the Milky-Way :math:`R_V=3.1` grain
size distribution **including PAHs**, so they correspond to a large
total grain surface and therefore to a strong sink for metal ions.
Molecular ions are not treated.

Some issues remain for the ``FULL`` network.

``GRAINRECOMB = 2``
^^^^^^^^^^^^^^^^^^^

This option replaces the empirical fits by a generic grain-collision rate in the
spirit of `Draine & Sutin (1987) <https://ui.adsabs.harvard.edu/abs/1987ApJ...320..803D/abstract>`_,
applied identically to **every** reaction of the form
:math:`\mathrm{X}^{Z+} + e^- \rightarrow \ldots` in the network, atomic
or molecular (e.g. H\ :sup:`+`, Mg\ :sup:`+`, HCO\ :sup:`+`, H\ :sub:`3`\ :sup:`+`).
Ions are identified by the trailing ``+`` characters of the species name,
without any list of species, so it works for any network.
Reactions that involve grain surfaces (``#`` in the reactant list) and
:math:`\mathrm{PAH}^+` (which has its own treatment) are excluded.

The model uses a single representative grain of radius :math:`a`
(``Grain radius`` in ``params.dat``) and material density
:math:`\rho_{\rm gr} = 3\ \mathrm{g\,cm^{-3}}`.
The grain number density follows from the dust-to-gas mass ratio
:math:`0.01\,Z/Z_\odot` (``Dust-to-gas normalized to 1e-2`` in ``params.dat``):

.. math::

   n_{\rm gr} = \frac{0.01\,(Z/Z_\odot)\,n_{\rm H}\,m_{\rm H}}{(4/3)\pi a^3 \rho_{\rm gr}}.

The grain term of the rate coefficient of an ion of mass :math:`m_X` and charge
:math:`Z_{\rm ion}` is

.. math::

   k_{\rm gr}(\mathrm{X}^{Z+}) = n_{\rm gr}\,\pi a^2\,J(\tau)\,\sqrt{\frac{8 k_B T}{\pi m_X}},
   \qquad J(\tau) = 1 + \sqrt{\frac{\pi}{2\tau}},
   \qquad \tau = \frac{a k_B T}{(Z_{\rm ion} e)^2},

i.e. the collision rate of the ion with a neutral grain, enhanced by
the ion-induced polarization of the grain (eq. 3.1 of Draine & Sutin 1987;
:math:`Z_{\rm ion}` is the number of trailing ``+`` characters, 1 for most species and 2 for
doubly ionized ones such as C\ :sup:`++` or S\ :sup:`++`).
Every collision is assumed to neutralize the ion (unit charge-transfer probability).
If an ion has :math:`N` electron-recombination channels in the network,
the grain term is shared equally between them, so the total added rate equals
:math:`k_{\rm gr}` exactly once.

.. note::

   - ``GRAINRECOMB = 2`` includes **no** PAH population: with a single
     grain of :math:`a = 10^{-5}` cm the total grain surface is much smaller than
     the one implied by the Weingartner & Draine fits, so the grain sink for the metal
     ions is weaker than for ``GRAINRECOMB = 1``. For a fixed dust-to-gas ratio
     :math:`k_{\rm gr}\propto 1/a`, i.e. the result is sensitive to the grain
     radius adopted in ``params.dat``.
   - The grain is assumed to be neutral (polarization limit); the grain charge
     distribution is not computed.
   - Molecular ions are also neutralized on grains, which is not the case for
     ``GRAINRECOMB = 1``. Abundances of molecular ions, in particular
     HCO\ :sup:`+`, can therefore change.
   - The ``REDUCED`` network contains explicit grain-surface recombination
     reactions of He\ :sup:`+` and C\ :sup:`+` (with ``#`` in the reactant
     list, following `Gong et al. (2017) <https://ui.adsabs.harvard.edu/abs/2017ApJ...843...38G/abstract>`_).
     They are not modified by ``GRAINRECOMB = 2`` but remain active, so for these two ions
     the grain sink is counted twice with this network (the same holds for ``GRAINRECOMB = 1``).
     The ``MEDIUM``, ``FULL`` and ``MYNETWORK`` networks do not have such reactions.
   - The ``REDUCED``, ``MEDIUM``, ``FULL`` and ``MYNETWORK`` networks can all be
     used.

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
