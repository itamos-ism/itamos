H2 Formation
============

This routine is activated when the ``H2FORM = CT02`` flag is specified in the ``config.mk``. The function implements the detailed H₂ formation model developed by
`Cazaux & Tielens <https://ui.adsabs.harvard.edu/abs/2002ApJ...575L..29C/abstract>`_, which accounts for:

1. **Thermal Physics**:

   - Mean thermal velocity of hydrogen atoms: :math:`v_{th} = 1.45 \times 10^5 \sqrt{T_g / 100}` cm/s
   - Temperature-dependent sticking coefficient from `Hollenbach & McKee (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJS...41..555H/abstract>`_

2. **Grain Surface Chemistry**:

   - Separate treatment for silicate and graphite grains
   - Distinction between physisorbed and chemisorbed hydrogen atoms
   - Energy barriers (saddle points) between adsorption sites
   - Desorption energies for H atoms and H₂ molecules
   - Vibrational frequencies in surface sites

3. **Formation Efficiency**:

   - Computes efficiency factors for both silicate and graphite grains
   - Accounts for competition between H₂ formation, desorption, and surface diffusion
   - Includes the fraction of newly formed H₂ that remains on grains (μ = 0.005)

Formulation
-----------

**Sticking Coefficient** (`Hollenbach & McKee (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJS...41..555H/abstract>`_):

.. math::
   S = \frac{1}{1 + 0.04\sqrt{T_g + T_d} + 0.2(T_g/100) + 0.08(T_g/100)^2}

**Silicate Formation Efficiency**:

.. math::
   \epsilon_{sil} = \frac{1}{1 + \text{FACTOR1} + \text{FACTOR2}} \times \epsilon

   \text{FACTOR1} = \frac{\mu F}{2\nu_{H2} \exp(-E_{H2}/T_d)}

   \text{FACTOR2} = \frac{\left[1 + \sqrt{\frac{E_{HC} - E_S}{E_{HP} - E_S}}\right]^2}{4} \exp(-E_S/T_d)

   \epsilon = \frac{1}{1 + \frac{\nu_{HC}}{2F} \exp(-1.5E_{HC}/T_d) \left[1 + \sqrt{\frac{E_{HC} - E_S}{E_{HP} - E_S}}\right]^2}

**Graphite Formation Efficiency**:
   Similar formulation with graphite-specific parameters.

**Final Rate**:

.. math::
   R = 0.5 \times v_{th} \times (A_{sil}\eta_{sil} + A_{gra}\eta_{gra}) \times S \times Z

where :math:`Z` is the metallicity scaling factor equivalent to the dust-to-gas ratio normalized to :math:`10^{-2}`.

Parameters Used
---------------

+--------------+--------------+---------------------------------------------------+
| Parameter    |   Value      |  Description                                      |
+--------------+--------------+---------------------------------------------------+
| Flux (F)     |   1.0×10⁻¹⁰  |  H atom flux in monolayers per second             |
+--------------+--------------+---------------------------------------------------+
| A_silicate   |  8.473×10⁻²² |  Silicate grain cross section per H nucleus (cm²) |
+--------------+--------------+---------------------------------------------------+
| A_graphite   |  7.908×10⁻²² |  Graphite grain cross section per H nucleus (cm²) |
+--------------+--------------+---------------------------------------------------+


**Silicate Grain Properties**:

  - μ (retention fraction): 0.005
  - E_S (saddle energy): 110 K
  - E_H2 (H₂ desorption): 320 K
  - E_HP (physisorbed H): 450 K
  - E_HC (chemisorbed H): 30000 K
  - ν_H2 (H₂ vibration): 3.0×10¹² s⁻¹
  - ν_HC (H vibration): 1.3×10¹³ s⁻¹

**Graphite Grain Properties**:

  - μ (retention fraction): 0.005
  - E_S (saddle energy): 260 K
  - E_H2 (H₂ desorption): 520 K
  - E_HP (physisorbed H): 800 K
  - E_HC (chemisorbed H): 30000 K
  - ν_H2 (H₂ vibration): 3.0×10¹² s⁻¹
  - ν_HC (H vibration): 1.3×10¹³ s⁻¹

Notes
-----

- The function includes commented-out alternative formulations from:

  - Traditional rate with simple temperature dependence
  - de Jong (1977) treatment with exponential cutoff
  - Tielens & Hollenbach (1985) treatment
  - Markus Röllig's expression from the 2012 Leiden workshop

- Metallicity scaling is applied to the final rate, assuming linear dependence
  on metal abundance relative to solar.
