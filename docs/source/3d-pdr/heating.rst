=================
Heating Processes
=================

The file ``heatingfunctions.F90`` documents the heating terms implemented in the
``CALC_HEATING`` subroutine of the 3D-PDR code. Each heating mechanism
contributes to the total volumetric heating rate of the gas
(:math:`\mathrm{erg\ cm^{-3}\ s^{-1}}`) at a given grid cell, based on the
local physical conditions and chemical abundances.

The total heating rate is computed as the sum of the individual
processes described below.

Only the heating mechanisms that are **active in the default energy
balance** are described here. 

.. 
   Dust photoelectric heating (classical), the Weingartner & Draine formulation, soft X-ray heating, and mechanical heating are intentionally omitted.

-------------------------
Units and Conventions
-------------------------

- Gas density ``DENSITY`` is the total hydrogen number density
  (:math:`n_\mathrm{H}`) in :math:`\mathrm{cm^{-3}}`.
- Gas and dust temperatures are in Kelvin.
- Reaction rates ``RATE(i)`` are provided by
  ``CALCULATE_REACTION_RATES``.
- Elemental and molecular abundances are fractional abundances relative
  to hydrogen nuclei.
- Energies expressed in eV are internally converted to erg using the
  constant ``EV``.

-------------------------------
PAH Photoelectric Heating
-------------------------------

This routine computes the photoelectric heating rate from dust grains and Polycyclic Aromatic Hydrocarbons (PAHs) using the formalism of Bakes & Tielens (1994) with modifications from Wolfire et al. (2003, 2008). The calculation assumes an MRN grain-size distribution with radii ranging from 3 to 100 Å.

Key references
~~~~~~~~~~~~~~

- `Bakes & Tielens (1994, ApJ, 427, 822) <https://ui.adsabs.harvard.edu/abs/1994ApJ...427..822B/abstract>`_
- `Wolfire et al. (2003, ApJ, 587, 278) <https://ui.adsabs.harvard.edu/abs/2003ApJ...587..278W/abstract>`_; PAH abundance revision from Spitzer data
- `Wolfire et al. (2008, ApJ, 680, 384) <https://ui.adsabs.harvard.edu/abs/2008ApJ...680..384W/abstract>`_; PAH scaling factor
- `Wolfire et al. (1995, ApJ, 443, 152) <https://ui.adsabs.harvard.edu/abs/1995ApJ...443..152W/abstract>`_
- `Le Page, Snow & Bierbaum (2001, ApJS, 132, 233) <https://ui.adsabs.harvard.edu/abs/2001ApJS..132..233L/abstract>`_


1. **PAH Scaling Factor** (Wolfire et al. 2008)

   .. math::
      \phi_{\mathrm{PAH}} = 0.4

   .. note:: 
            Setting :math:`\phi_{\mathrm{PAH}} = 1.0` recovers the original Bakes & Tielens (1994) formulation.

2. **Temperature-dependent Exponents**

   .. math::
      \alpha = 0.944
   
   .. math::
      \beta(T) = \frac{0.735}{T^{0.068}}
   
   where :math:`T` is the gas temperature in Kelvin.

3. **Dimensionless Ionization Parameter** (:math:`\delta`)

   .. math::
      \delta = \frac{G_0 \sqrt{T}}{n_e n_{\mathrm{H}} \phi_{\mathrm{PAH}}}

   where:
   
   - :math:`G_0` = Habing radiation field strength (1 Habing unit = :math:`1.6 \times 10^{-3}` erg cm⁻² s⁻¹)
   - :math:`T` = Gas temperature (K)
   - :math:`n_e` = Electron number density (cm⁻³)
   - :math:`n_{\mathrm{H}}` = Total hydrogen number density (cm⁻³)

4. **Photoelectric Heating Efficiency** (:math:`\epsilon`)

   .. math::
      \epsilon(T, \delta) = \frac{4.87 \times 10^{-2}}{1 + 4.0 \times 10^{-3} \delta^{0.73}} 
                          + \frac{3.65 \times 10^{-2} \left(\dfrac{T}{10^4}\right)^{0.7}}{1 + 2.0 \times 10^{-4} \delta}

5. **Heating and Cooling Rates**

   **PAH Heating Rate** (erg cm⁻³ s⁻¹):
   
   .. math::
      \Gamma_{\mathrm{heat}} = 1.30 \times 10^{-24} \epsilon G_0 n_{\mathrm{H}}

   **PAH Cooling Rate** (erg cm⁻³ s⁻¹):
   
   .. math::
      \Lambda_{\mathrm{cool}} = 4.65 \times 10^{-30} T^{\alpha} \delta^{\beta} n_e n_{\mathrm{H}} \phi_{\mathrm{PAH}}

6. **Net Photoelectric Heating Rate**

   .. math::
      \Gamma_{\mathrm{PE}} = \left(\Gamma_{\mathrm{heat}} - \Lambda_{\mathrm{cool}}\right) \times Z

   where :math:`Z` is the metallicity relative to solar (linear scaling factor).

Combined Formula
~~~~~~~~~~~~~~~~

Substituting all components:

.. math::
      
   \boxed{\Gamma_{\mathrm{PE}} = Z \left[ 1.30 \times 10^{-24} \epsilon G_0 n_{\mathrm{H}} 
          - 4.65 \times 10^{-30} T^{\alpha} \delta^{\beta} n_e n_{\mathrm{H}} \phi_{\mathrm{PAH}} \right]}

with the auxiliary definitions:

.. math::
   \delta &= \frac{G_0 \sqrt{T}}{n_e n_{\mathrm{H}} \phi_{\mathrm{PAH}}} \\
   \epsilon &= \frac{4.87 \times 10^{-2}}{1 + 4.0 \times 10^{-3} \delta^{0.73}} 
              + \frac{3.65 \times 10^{-2} \left(\dfrac{T}{10^4}\right)^{0.7}}{1 + 2.0 \times 10^{-4} \delta} \\
   \alpha &= 0.944 \\
   \beta &= \frac{0.735}{T^{0.068}} \\
   \phi_{\mathrm{PAH}} &= 0.4

Parameter Ranges and Notes
~~~~~~~~~~~~~~~~~~~~~~~~~~

- **Typical ranges**:
  - :math:`G_0`: 0.1 to 10⁶ (Habing units)
  - :math:`T`: 10 to 10⁴ K
  - :math:`n_{\mathrm{H}}`: 0.1 to 10⁶ cm⁻³
  - :math:`n_e/n_{\mathrm{H}}`: ~10⁻⁴ to 1 (ionization fraction)

- **Physical meaning of** :math:`\delta`:
  The parameter :math:`\delta` represents the ratio of ionization rate (proportional to :math:`G_0\sqrt{T}`) 
  to recombination rate (proportional to :math:`n_e n_{\mathrm{H}}\phi_{\mathrm{PAH}}`).
  
  - When :math:`\delta \gg 1`: Ionization dominates so heating is efficient
  - When :math:`\delta \ll 1`: Recombination dominates so cooling becomes important

- **Efficiency** :math:`\epsilon`:
  The efficiency function has two terms representing different physical regimes:
  
  1. First term: Dominates at low temperatures and moderate :math:`\delta`
  2. Second term: Becomes important at high temperatures (:math:`T \gtrsim 10^4` K)


This term is stored as:

- ``HEATING_RATE(2)`` — PAH photoelectric heating

--------------------------------
Carbon Photoionization Heating
--------------------------------

Photoionization of neutral carbon deposits kinetic energy into the gas.
An average energy of 1 eV per ionization is assumed.

The heating rate is computed as

.. math::

   \boxed{\Gamma_{\rm heat} =
   (1~{\rm eV}) \, k_{\rm ion}(C) \, n({\rm C}) \, n_{\rm H}}

where the photoionization rate
:math:`k_{\rm ion}(C)` is provided by the chemical network.

A numerical floor of :math:`10^{-30}` is imposed for stability.

Stored as:

- ``HEATING_RATE(4)`` — carbon photoionization heating

-------------------------
H₂ Formation Heating
-------------------------

Molecular hydrogen formation on grain surfaces releases energy into the
gas. A value of 1.5 eV per formed H₂ molecule is adopted
(`Hollenbach & Tielens 1999 <https://ui.adsabs.harvard.edu/abs/1999RvMP...71..173H/abstract>`_).

The heating rate is

.. math::

   \boxed{\Gamma_{\rm H_2,form} =
   (1.5~{\rm eV}) \, k_{\rm gr} \, n({\rm H}) \, n_{\rm H}}

where ``k_gr`` is the grain-surface formation rate coefficient.

Stored as:

- ``HEATING_RATE(5)`` — H₂ formation heating

------------------------------
H₂ Photodissociation Heating
------------------------------

Photodissociation of H₂ deposits kinetic energy into the gas. An average
energy of 0.4 eV per dissociation is assumed.

The heating rate is

.. math::

   \boxed{\Gamma_{\rm H_2,diss} =
   (0.4~{\rm eV}) \, k_{\rm diss} \, n({\rm H_2}) \, n_{\rm H}}

Stored as:

- ``HEATING_RATE(6)`` — H₂ photodissociation heating

-------------------------
H₂ FUV Pumping Heating
-------------------------

This routine computes the heating due to Far-Ultraviolet (FUV) pumping of molecular hydrogen (H₂). When H₂ molecules absorb FUV photons, they are excited to higher vibrational/rotational states and subsequently decay collisionally, converting radiative energy into thermal energy. See `Hollenbach & McKee (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJS...41..555H/abstract>`_ for the original formulation.

Each vibrationally excited H₂* molecule deposits approximately **2.2 eV** of energy into the gas through collisional de-excitation. The heating rate depends on:
1. The H₂ photodissociation rate
2. The critical density for collisional de-excitation
3. The fractional abundance of H₂

1. **Critical Density for H₂** (:math:`n_{\mathrm{cr,H₂}}`)

   .. math::
      n_{\mathrm{cr,H₂}} = \frac{10^6}{\sqrt{T}} \times \frac{1}{1.6 f_{\mathrm{H}} \exp\left[-\left(\dfrac{400}{T}\right)^2\right] 
                          + 1.4 f_{\mathrm{H₂}} \exp\left[-\dfrac{18100}{T+1200}\right]}


   The critical density represents the density at which collisional de-excitation competes with radiative decay.

2. **FUV Pumping Heating Rate** (erg cm⁻³ s⁻¹)

   .. math::
      \Gamma_{\mathrm{FUV,pump}} = (2.2\ \mathrm{eV}) \times 9.0 \times k_{\mathrm{H₂,phot}} \times f_{\mathrm{H₂}} \times n_{\mathrm{H}}
                                 \times \frac{1}{1 + \dfrac{n_{\mathrm{cr,H₂}}}{n_{\mathrm{H}}}}

   where:
   
   - :math:`2.2\ \mathrm{eV} = 3.524 \times 10^{-12}` erg (energy per excited H₂*)
   - :math:`k_{\mathrm{H₂,phot}}` = H₂ photodissociation rate (s⁻¹)

Complete Combined Expression
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. math::
   
   \boxed{\Gamma_{\mathrm{FUV,pump}} = \frac{(2.2\ \mathrm{eV}) \times 9.0 \times k_{\mathrm{H₂,phot}} \times f_{\mathrm{H₂}} \times n_{\mathrm{H}}}{1 + \dfrac{n_{\mathrm{cr,H₂}}}{n_{\mathrm{H}}}}}

with:

.. math::
   n_{\mathrm{cr,H₂}} = \frac{10^6}{\sqrt{T}} \left[1.6 f_{\mathrm{H}} \exp\left(-\left(\frac{400}{T}\right)^2\right) 
                      + 1.4 f_{\mathrm{H₂}} \exp\left(-\frac{18100}{T+1200}\right)\right]^{-1}


Density Regimes
~~~~~~~~~~~~~~~

1. **Low-density limit** (:math:`n_{\mathrm{H}} \ll n_{\mathrm{cr,H₂}}`):
   
   .. math::
      \Gamma_{\mathrm{FUV,pump}} \approx (2.2\ \mathrm{eV}) \times 9.0 \times k_{\mathrm{H₂,phot}} \times f_{\mathrm{H₂}} \times n_{\mathrm{H}} \times \frac{n_{\mathrm{H}}}{n_{\mathrm{cr,H₂}}}
   
   Heating is suppressed because excited H₂* decays radiatively before collisional de-excitation.

2. **High-density limit** (:math:`n_{\mathrm{H}} \gg n_{\mathrm{cr,H₂}}`):
   
   .. math::
      \Gamma_{\mathrm{FUV,pump}} \approx (2.2\ \mathrm{eV}) \times 9.0 \times k_{\mathrm{H₂,phot}} \times f_{\mathrm{H₂}} \times n_{\mathrm{H}}
   
   Heating is maximized as all excited H₂* molecules decay collisionally.

Critical Density Components
~~~~~~~~~~~~~~~~~~~~~~~~~~~
The critical density expression contains two terms:

1. **Atomic hydrogen term**:
   
   .. math::
      1.6 f_{\mathrm{H}} \exp\left[-\left(\dfrac{400}{T}\right)^2\right]
   
   Represents H-H₂ collisions. The exponential term accounts for the temperature dependence of the collision cross-section.

2. **Molecular hydrogen term**:
   
   .. math::
      1.4 f_{\mathrm{H₂}} \exp\left[-\dfrac{18100}{T+1200}\right]
   
   Represents H₂-H₂ collisions. The exponential term accounts for the energy barrier for vibrational de-excitation.


Notes
~~~~~
- The factor of 9.0 comes from observations that there are approximately 9 excitations (FUV pumpings) per photodissociation event.
- The critical density calculation assumes H and H₂ are the dominant collision partners for H₂*.
- This heating mechanism is particularly important in photon-dominated regions (PDRs) where FUV radiation is strong.

Stored as:

- ``HEATING_RATE(7)`` — H₂ FUV pumping heating

----------------------------
Cosmic-Ray Heating
----------------------------

Cosmic-ray ionization of molecular hydrogen heats the gas.
An effective energy deposition of 9.4 eV per primary ionization is used.

The heating rate is

.. math::

   \Gamma_{\rm CR} =
   (9.4~{\rm eV}) \,
   (1.3 \times 10^{-17}) \,
   \zeta \,
   n({\rm H_2}) \,
   n_{\rm H}

where :math:`\zeta` is the local cosmic-ray ionization scaling factor.

Stored as:

- ``HEATING_RATE(8)`` — cosmic-ray heating

-----------------------------
Turbulent Dissipation Heating
-----------------------------

Dissipation of supersonic turbulence provides an additional heating
source, particularly relevant in dense and dynamically active regions.

The heating rate follows Black (1987):

.. math::

   \Gamma_{\rm turb} \propto
   \rho \, \frac{v_{\rm turb}^3}{L_{\rm turb}}

with a default turbulent length scale of 5 pc.

Stored as:

- ``HEATING_RATE(9)`` — turbulent heating

-----------------------------
Exothermic Reaction Heating
-----------------------------

.. warning:: **INDEX DEPENDENCE ON CHEMICAL NETWORK**
   
   The rate indices (e.g., ``RATE(216)``, ``RATE(240)``) in this routine are **specific to each chemical network**. 
   If you are designing or using a custom chemical network:
   
   1. **You must manually update all rate indices** to match your network's reaction numbering.
   2. The indices differ between ``REDUCED``, ``MEDIUM``, ``FULL``, and custom networks.
   3. The ``MYNETWORK`` section includes a ``STOP`` statement as a reminder to update these indices.

This routine computes the heating from exothermic chemical reactions. When chemical bonds form or rearrange, the released energy (reaction enthalpy) can be deposited as thermal energy into the gas.

For each exothermic reaction:
:math:`\Gamma_{\mathrm{chem}} = n_1 n_2 \times k \times E`
where:

- :math:`n_1, n_2` = Number densities of reactants (cm⁻³)
- :math:`k` = Reaction rate coefficient (cm³ s⁻¹)
- :math:`E` = Energy released per reaction (erg)


The code includes heating from six classes of exothermic reactions:

1. **H₂⁺ recombination**: :math:`\mathrm{H}_2^+ + e^- \rightarrow \mathrm{H} + \mathrm{H}` (10.9 eV)
2. **H₂⁺ charge transfer**: :math:`\mathrm{H}_2^+ + \mathrm{H} \rightarrow \mathrm{H}^+ + \mathrm{H}_2` (0.94 eV)
3. **HCO⁺ recombination**: :math:`\mathrm{HCO}^+ + e^- \rightarrow \mathrm{CO} + \mathrm{H}` (7.51 eV)
4. **H₃⁺ recombination**: :math:`\mathrm{H}_3^+ + e^- \rightarrow` products (4.76 + 9.23 eV total)
5. **H₃O⁺ recombination**: :math:`\mathrm{H}_3\mathrm{O}^+ + e^- \rightarrow` products (1.16 + 5.63 + 6.27 eV total)
6. **He⁺ ion-neutral reactions**: 
   - :math:`\mathrm{He}^+ + \mathrm{H}_2 \rightarrow \mathrm{H}^+ + \mathrm{H} + \mathrm{He}` (6.51 eV)
   - :math:`\mathrm{He}^+ + \mathrm{CO} \rightarrow \mathrm{C}^+ + \mathrm{O} + \mathrm{He}` (2.22 eV)

Mathematical Expressions
~~~~~~~~~~~~~~~~~~~~~~~~

.. math::
   \Gamma_{\mathrm{chem}} = \sum_{i=1}^{8} \Gamma_i

where:

1. **H₂⁺ + e⁻ recombination**: :math:`\Gamma_1 = f_{\mathrm{H}_2^+} n_{\mathrm{H}} \times f_e n_{\mathrm{H}} \times k_1 \times (10.9\ \mathrm{eV})`

2. **H₂⁺ + H charge transfer**: :math:`\Gamma_2 = f_{\mathrm{H}_2^+} n_{\mathrm{H}} \times f_{\mathrm{H}} n_{\mathrm{H}} \times k_2 \times (0.94\ \mathrm{eV})`

3. **HCO⁺ + e⁻ recombination**: :math:`\Gamma_3 = f_{\mathrm{HCO}^+} n_{\mathrm{H}} \times f_e n_{\mathrm{H}} \times k_3 \times (7.51\ \mathrm{eV})`

4. **H₃⁺ + e⁻ recombination** (two channels): :math:`\Gamma_4 = f_{\mathrm{H}_3^+} n_{\mathrm{H}} \times f_e n_{\mathrm{H}} \times (k_{4a} \times 4.76\ \mathrm{eV} + k_{4b} \times 9.23\ \mathrm{eV})`

5. **H₃O⁺ + e⁻ recombination** (three channels): :math:`\Gamma_5 = f_{\mathrm{H}_3\mathrm{O}^+} n_{\mathrm{H}} \times f_e n_{\mathrm{H}} \times (k_{5a} \times 1.16\ \mathrm{eV} + k_{5b} \times 5.63\ \mathrm{eV} + k_{5c} \times 6.27\ \mathrm{eV})`

6. **He⁺ + H₂ reactions** (two channels): :math:`\Gamma_6 = f_{\mathrm{He}^+} n_{\mathrm{H}} \times f_{\mathrm{H}_2} n_{\mathrm{H}} \times (k_{6a} + k_{6b}) \times (6.51\ \mathrm{eV})`

7. **He⁺ + CO reactions** (one to three channels depending on network): :math:`\Gamma_7 = f_{\mathrm{He}^+} n_{\mathrm{H}} \times f_{\mathrm{CO}} n_{\mathrm{H}} \times (\sum k_{7,i}) \times (2.22\ \mathrm{eV})`

Network-Specific Rate Indices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The reaction rate indices vary significantly between networks:


+-----------------+----------------+----------------+----------------+-------------------+
| **Reaction**    | **REDUCED**    | **MEDIUM**     | **FULL**       | **MYNETWORK**     |
+=================+================+================+================+===================+
| H₂⁺ + e⁻       | ``RATE(216)``  | ``RATE(255)``  | ``RATE(670)``  | Not included      |
+-----------------+----------------+----------------+----------------+-------------------+
| H₂⁺ + H        | ``RATE(155)``  | ``RATE(11)``   | ``RATE(290)``  | Not included      |
+-----------------+----------------+----------------+----------------+-------------------+
| HCO⁺ + e⁻      | ``RATE(240)``  | ``RATE(280)``  | ``RATE(730)``  | ``RATE(185)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| H₃⁺ + e⁻ (a)   | ``RATE(217)``  | ``RATE(266)``  | ``RATE(706)``  | ``RATE(173)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| H₃⁺ + e⁻ (b)   | ``RATE(218)``  | ``RATE(265)``  | ``RATE(705)``  | ``RATE(172)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| H₃O⁺ + e⁻ (a)  | ``RATE(236)``  | ``RATE(275)``  | ``RATE(717)``  | ``RATE(179)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| H₃O⁺ + e⁻ (b)  | ``RATE(237)``  | ``RATE(274)``  | ``RATE(716)``  | ``RATE(178)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| H₃O⁺ + e⁻ (c)  | ``RATE(238)``  | ``RATE(272)``  | ``RATE(714)``  | ``RATE(176)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| He⁺ + H₂ (a)   | ``RATE(50)``   | ``RATE(445)``  | ``RATE(1227)`` | ``RATE(297)``     |
+-----------------+----------------+----------------+----------------+-------------------+
| He⁺ + H₂ (b)   | ``RATE(170)``  | ``RATE(96)``   | ``RATE(265)``  | ``RATE(67)``      |
+-----------------+----------------+----------------+----------------+-------------------+
| He⁺ + CO       | 3 rates        | 1 rate         | 1 rate         | 1 rate            |
|                 | ``(89,90,91)``| ``RATE(553)``  | ``RATE(1541)`` | ``RATE(364)``     |
+-----------------+----------------+----------------+----------------+-------------------+

Custom Network Implementation Guide
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you are using a custom chemical network:

1. **Locate the reaction numbers** in your network for the 8 key reactions above.
2. **Update all ``RATE(i)`` indices** in the ``MYNETWORK`` section.
3. **Comment out the ``STOP`` statement** once indices are updated.
4. **Verify** that all relevant species abundances (e.g., ``NH2x`` for H₂⁺) are defined in your network.
5. **Add or remove reactions** as needed for your specific chemistry.

Stored as:

- ``HEATING_RATE(10)`` — chemical heating

-------------------------------
Gas–Grain Collisional Heating
-------------------------------

This routine computes the thermal energy exchange between gas and dust grains through inelastic collisions. When gas particles collide with dust grains, they may transfer kinetic energy to the grains (cooling the gas) or receive energy from the grains (heating the gas), depending on the relative temperatures.


Key References
~~~~~~~~~~~~~~
- `Burke & Hollenbach (1983, ApJ, 265, 223) <https://ui.adsabs.harvard.edu/abs/1983ApJ...265..223B/abstract>`_; Primary formulation
- `Groenewegen (1994, A&A, 290, 531) <https://ui.adsabs.harvard.edu/abs/1994A%26A...290..531G/abstract>`_; Accommodation coefficient fitting formula
- `Hollenbach & McKee (1979, ApJS, 41, 555) <https://ui.adsabs.harvard.edu/abs/1979ApJS...41..555H/abstract>`_
- `Tielens & Hollenbach (1985, ApJ, 291, 722) <https://ui.adsabs.harvard.edu/abs/1985ApJ...291..722T/abstract>`_
- `Goldsmith (2001, ApJ, 557, 736) <https://ui.adsabs.harvard.edu/abs/2001ApJ...557..736G/abstract>`_
- `Tielens (2005, "The Physics and Chemistry of the Interstellar Medium") <https://ui.adsabs.harvard.edu/abs/2005pcim.book.....T/abstract>`_


Physical Process
~~~~~~~~~~~~~~~~
Gas particles (atoms, molecules, ions) collide with dust grains. A fraction of the particles "accommodate" to the grain temperature through inelastic interactions. The net energy transfer depends on:
1. The gas-dust temperature difference
2. The accommodation coefficient (probability of thermal accommodation)
3. The grain surface area per unit volume
4. The gas thermal velocity

Mathematical Formulation (Burke & Hollenbach 1983)
---------------------------------------------------

1. **Number Density of Grains** (:math:`n_{\mathrm{grain}}`)

   .. math::
      n_{\mathrm{grain}} = 1.998 \times 10^{-12} \times n_{\mathrm{H}} \times Z

   where:
   
   - :math:`n_{\mathrm{H}}` = Total hydrogen number density (cm⁻³)
   - :math:`Z` = Metallicity relative to solar (linear scaling factor)
   
   This assumes a standard grain abundance scaling with metallicity.

2. **Geometric Cross-Section per Grain** (:math:`C_{\mathrm{grain}}`)

   .. math::
      C_{\mathrm{grain}} = \pi a^2

   where :math:`a` is the grain radius (assumed uniform in this implementation).

3. **Thermal Accommodation Coefficient** (:math:`\alpha_T`)

   Using Groenewegen (1994) fitting formula:
   
   .. math::
      \alpha_T(T_{\mathrm{gas}}) = 0.37 \times \left[1 - 0.8 \exp\left(-\frac{75}{T_{\mathrm{gas}}}\right)\right]

   This represents the probability that a gas particle thermalizes with the grain surface during a collision.

4. **Gas-Grain Heating Rate** (erg cm⁻³ s⁻¹)

   .. math::
      \Gamma_{\mathrm{gas-grain}} = n_{\mathrm{grain}} C_{\mathrm{grain}} n_{\mathrm{H}}
                                  \times v_{\mathrm{th}} \times \alpha_T
                                  \times \left(2k_B T_{\mathrm{dust}} - 2k_B T_{\mathrm{gas}}\right)

   where the mean thermal speed is:
   
   .. math::
      v_{\mathrm{th}} = \sqrt{\frac{8k_B T_{\mathrm{gas}}}{\pi m_{\mathrm{H}}}}

   with :math:`m_{\mathrm{H}}` being the mass of a hydrogen atom.

Complete Combined Expression
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. math::
   
   \boxed{\Gamma_{\mathrm{gas-grain}} = n_{\mathrm{grain}} \pi a^2 n_{\mathrm{H}} 
          \times \sqrt{\frac{8k_B T_{\mathrm{gas}}}{\pi m_{\mathrm{H}}}}
          \times \alpha_T(T_{\mathrm{gas}})
          \times 2k_B \left(T_{\mathrm{dust}} - T_{\mathrm{gas}}\right)}

with:

.. math::
   n_{\mathrm{grain}} &= 1.998 \times 10^{-12} \times n_{\mathrm{H}} \times Z \\
   \alpha_T(T_{\mathrm{gas}}) &= 0.37 \left[1 - 0.8 \exp\left(-\dfrac{75}{T_{\mathrm{gas}}}\right)\right]

Simplified Constant Pre-factor
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Burke & Hollenbach (1983) note that the combination:

.. math::
   \sqrt{\frac{8k_B}{\pi m_{\mathrm{H}}}} \times 2k_B = 4.003 \times 10^{-12}\ \mathrm{cgs\ units}

Thus the heating rate can also be written as:

.. math::
   \Gamma_{\mathrm{gas-grain}} = n_{\mathrm{grain}} \pi a^2 n_{\mathrm{H}}
                               \times 4.003 \times 10^{-12} \sqrt{T_{\mathrm{gas}}}
                               \times \alpha_T(T_{\mathrm{gas}})
                               \times \left(T_{\mathrm{dust}} - T_{\mathrm{gas}}\right)

Alternative Formulation (Tielens 2005)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The code also includes an alternative simplified expression from Tielens (2005):

.. math::
   \Gamma_{\mathrm{gas-grain}}^{\mathrm{(Tielens)}} = -1 \times 10^{-33} \times n_{\mathrm{H}}^2
                                                    \times \sqrt{T_{\mathrm{gas}}}
                                                    \times \left(T_{\mathrm{gas}} - T_{\mathrm{dust}}\right)

This is provided for comparison but not used in the primary calculation (commented out in the code).


Stored as:

- ``HEATING_RATE(11)`` — gas–grain collisional heating

-------------------------
Total Heating Rate
-------------------------

The total heating rate used in the thermal balance is the sum of all
active contributions:

.. math::

   \boxed{\Gamma_{\rm total} =
   \sum_i \Gamma_i}

Stored as:

- ``HEATING_RATE(12)`` — total heating rate
