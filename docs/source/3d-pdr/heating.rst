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




--------------------------------
Carbon Photoionization Heating
--------------------------------

This routine computes the heating from photoionization of carbon atoms. When UV photons ionize carbon atoms, the excess photon energy beyond the ionization potential is converted to kinetic energy of the ejected electron, which thermalizes through collisions.

The heating rate is given by:

.. math::
   \Gamma_{\mathrm{C,ion}} = E_{\mathrm{dep}} \times k_{\mathrm{C,phot}} \times n_{\mathrm{C}}

where:

- :math:`E_{\mathrm{dep}}` = 1.0 eV (average energy deposited as heat per ionization)
- :math:`k_{\mathrm{C,phot}}` = Carbon photoionization rate (s⁻¹)
- :math:`n_{\mathrm{C}}` = Carbon number density (cm⁻³)

Complete Expression
-------------------

.. math::
   
   \boxed{\Gamma_{\mathrm{C,ion}} = (1.0\ \mathrm{eV}) \times k_{\mathrm{C,phot}} \times f_{\mathrm{C}} \times n_{\mathrm{H}}}

In terms of physical quantities:

.. math::
   \Gamma_{\mathrm{C,ion}} = 1.602 \times 10^{-12} \times k_{\mathrm{C,phot}} \times f_{\mathrm{C}} \times n_{\mathrm{H}}\ \mathrm{erg\ cm}^{-3}\ \mathrm{s}^{-1}

The code implements a minimum value to prevent underflow:

.. code-block:: fortran
   
   if (CIONIZATION_HEATING.lt.1d-30) CIONIZATION_HEATING=1d-30

This ensures the heating rate never falls below :math:`10^{-30}` erg cm⁻³ s⁻¹.

Notes
~~~~~
1. The value of 1.0 eV represents the average excess energy of ionizing photons beyond the carbon ionization potential (11.26 eV).
2. This heating mechanism is important in diffuse atomic and ionized regions where carbon is a significant absorber of UV photons.
3. The heating rate is proportional to the carbon abundance, which scales with metallicity.



-------------------------
H₂ Formation Heating
-------------------------

This routine computes the heating from molecular hydrogen formation on dust grain surfaces. When two hydrogen atoms combine on a grain surface to form H₂, the binding energy (4.48 eV) is released, with approximately 1.5 eV deposited as thermal energy into the gas. See `Hollenbach & Tielens (1999) <https://ui.adsabs.harvard.edu/abs/1999RvMP...71..173H/abstract>`_ for complete description.

The heating rate follows:

.. math::
   \Gamma_{\mathrm{H_2,form}} = E_{\mathrm{dep}} \times k_{\mathrm{H_2,form}} \times n_{\mathrm{H}}^2 \times f_{\mathrm{H}}^2

where:

- :math:`E_{\mathrm{dep}}` = 1.5 eV (energy released as heat per H₂ formed)
- :math:`k_{\mathrm{H_2,form}}` = H₂ formation rate coefficient on grains (cm³ s⁻¹)
- :math:`n_{\mathrm{H}}` = Total hydrogen number density (cm⁻³)
- :math:`f_{\mathrm{H}}` = Atomic hydrogen fractional abundance

Complete Expression
~~~~~~~~~~~~~~~~~~~

.. math::
   
   \boxed{\Gamma_{\mathrm{H_2,form}} = (1.5\ \mathrm{eV}) \times k_{\mathrm{H_2,form}} \times n_{\mathrm{H}} \times f_{\mathrm{H}} \times n_{\mathrm{H}} \times f_{\mathrm{H}}}
   
   \Gamma_{\mathrm{H_2,form}} = (1.5\ \mathrm{eV}) \times k_{\mathrm{H_2,form}} \times f_{\mathrm{H}}^2 \times n_{\mathrm{H}}^2

In terms of physical quantities:

.. math::
   \Gamma_{\mathrm{H_2,form}} = 2.403 \times 10^{-12} \times k_{\mathrm{H_2,form}} \times f_{\mathrm{H}}^2 \times n_{\mathrm{H}}^2\ \mathrm{erg\ cm}^{-3}\ \mathrm{s}^{-1}

Physical Process Details
~~~~~~~~~~~~~~~~~~~~~~~~
1. **Energy release**: H₂ binding energy = 4.48 eV
2. **Partitioning**:

   - ~1.5 eV goes into heating the gas (through grain-gas collisions)
   - ~0.2 eV goes into internal excitation of the newly formed H₂
   - ~2.78 eV goes into heating the dust grain

3. **Formation mechanism**: 

   - Langmuir-Hinshelwood mechanism (mobile H atoms on grain surfaces)
   - Eley-Rideal mechanism (gas-phase H atoms hitting adsorbed H atoms)

Notes
~~~~~
1. H₂ formation heating is particularly important in photodissociation regions (PDRs) where H₂ forms efficiently.
2. The heating scales as :math:`n^2`, making it dominant in dense regions.
3. The rate coefficient :math:`k_{\mathrm{H_2,form}}` typically depends on dust temperature and grain properties.
4. This heating mechanism couples the gas and dust temperatures through the formation process.

------------------------------
H₂ Photodissociation Heating
------------------------------

This routine computes the heating from H₂ photodissociation. When H₂ molecules absorb far-ultraviolet (FUV) photons in the Lyman-Werner bands, they can dissociate, with some of the excess photon energy converted to thermal energy.

The heating rate is given by:

.. math::
   \Gamma_{\mathrm{H_2,diss}} = E_{\mathrm{dep}} \times k_{\mathrm{H_2,phot}} \times n_{\mathrm{H_2}}

where:

- :math:`E_{\mathrm{dep}}` = 0.4 eV (average energy deposited as heat per photodissociation)
- :math:`k_{\mathrm{H_2,phot}}` = H₂ photodissociation rate (s⁻¹)
- :math:`n_{\mathrm{H_2}}` = H₂ number density (cm⁻³)

Complete Expression
~~~~~~~~~~~~~~~~~~~

.. math::
   
   \boxed{\Gamma_{\mathrm{H_2,diss}} = (0.4\ \mathrm{eV}) \times k_{\mathrm{H_2,phot}} \times f_{\mathrm{H_2}} \times n_{\mathrm{H}}}

In terms of physical quantities:

.. math::
   \Gamma_{\mathrm{H_2,diss}} = 6.409 \times 10^{-13} \times k_{\mathrm{H_2,phot}} \times f_{\mathrm{H_2}} \times n_{\mathrm{H}}\ \mathrm{erg\ cm}^{-3}\ \mathrm{s}^{-1}

Physical Process Details
~~~~~~~~~~~~~~~~~~~~~~~~
1. **Photodissociation mechanism**:

   - H₂ absorbs FUV photon (11.2-13.6 eV) in Lyman-Werner bands
   - Excited to electronic state, then decays to repulsive ground state
   - Molecules dissociate into two H atoms

2. **Energy partitioning**:

   - H₂ dissociation energy = 4.48 eV
   - Average photon energy ~12.4 eV
   - Excess energy ~7.92 eV per photon
   - ~0.4 eV ends up as thermal energy (rest goes into kinetic energy of H atoms)

Notes
~~~~~
1. This heating mechanism is important in photon-dominated regions (PDRs) at the edges of molecular clouds.
2. The photodissociation rate :math:`k_{\mathrm{H_2,phot}}` includes self-shielding effects for high H₂ column densities.
3. H₂ photodissociation heating is typically smaller than H₂ FUV pumping heating (which uses 2.2 eV per excitation).
4. This heating scales linearly with H₂ density and radiation field strength.

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

----------------------------
Cosmic-Ray Heating
----------------------------

This routine computes the heating due to cosmic-ray ionization. When cosmic rays ionize atoms and molecules, the kinetic energy of the ejected electrons and subsequent thermalization of the ionization cascade deposits heat into the gas.

Key References
~~~~~~~~~~~~~~
- `Tielens & Hollenbach (1985, ApJ, 291, 772) <https://ui.adsabs.harvard.edu/abs/1985ApJ...291..722T/abstract>`_; Primary formulation
- `Shull & Van Steenberg (1985, ApJ, 298, 268) <https://ui.adsabs.harvard.edu/abs/1985ApJ...298..268S/abstract>`_
- `Clavel et al. (1978, A&A, 65, 435) <https://ui.adsabs.harvard.edu/abs/1978A%26A....65..435C/abstract>`_
- `Kamp & van Zadelhoff (2001, A&A, 373, 641) <https://ui.adsabs.harvard.edu/abs/2001A%26A...373..641K/abstract>`_

The cosmic-ray heating rate is given by:

.. math::
   \Gamma_{\mathrm{CR}} = E_{\mathrm{dep}} \times (\zeta_{\mathrm{H}_2} \times n_{\mathrm{H}_2})

where:

1. **Energy deposited per ionization** (:math:`E_{\mathrm{dep}}`): :math:`E_{\mathrm{dep}}=9.4 \mathrm{eV}`
   
2. **H₂ ionization rate** (:math:`\zeta_{\mathrm{H}_2}`): :math:`\zeta_{\mathrm{H}_2} = 1.3 \times 10^{-17} \times \zeta_{\mathrm{local}}\ \mathrm{s}^{-1}`, where :math:`\zeta_{\mathrm{local}}` is the local cosmic-ray ionization rate scaling factor.

Complete Expression
~~~~~~~~~~~~~~~~~~~

.. math::
   
   \boxed{\Gamma_{\mathrm{CR}} = (9.4\ \mathrm{eV}) \times (1.3 \times 10^{-17} \times \zeta_{\mathrm{local}}) \times n_{\mathrm{H}} \times f_{\mathrm{H}_2}}

Expressed in terms of physical quantities:

.. math::
   \Gamma_{\mathrm{CR}} = 9.4 \times 1.602 \times 10^{-12} \times 1.3 \times 10^{-17} \times \zeta_{\mathrm{local}} \times n_{\mathrm{H}} \times f_{\mathrm{H}_2}\ \mathrm{erg\ cm}^{-3}\ \mathrm{s}^{-1}

The product of constants gives:

.. math::
   9.4 \times 1.602 \times 10^{-12} \times 1.3 \times 10^{-17} = 1.958 \times 10^{-28}

Thus: :math:`\Gamma_{\mathrm{CR}} = 1.958 \times 10^{-28} \times \zeta_{\mathrm{local}} \times n_{\mathrm{H}} \times f_{\mathrm{H}_2}`

- Primary ionization by cosmic rays produces fast electrons (~30 eV)
- These electrons cause secondary ionizations and excitations
- Ultimately, ~9.4 eV per primary ionization ends up as thermal energy
- The remaining energy goes into ionization, excitation, and dissociation

Notes
~~~~~
1. The factor 1.3 × 10⁻¹⁷ s⁻¹ is the standard Galactic H₂ cosmic-ray ionization rate.
2. The heating depends linearly on H₂ abundance, as H₂ is the primary target for cosmic rays in molecular gas.
3. In predominantly atomic gas, the heating would need to be adjusted (though the current implementation assumes H₂-dominated regions).
4. Cosmic-ray heating is particularly important in dense, shielded regions where UV heating is negligible.

-----------------------------
Turbulent Dissipation Heating
-----------------------------

This routine computes the heating from dissipation of supersonic turbulence. Turbulent kinetic energy cascades to smaller scales and is ultimately converted to thermal energy through viscous dissipation.

Key References
~~~~~~~~~~~~~~~
- Black (1987, in `"Interstellar Processes", p. 731 <https://ui.adsabs.harvard.edu/abs/1987ASSL..134.....H/abstract>`_); Primary formulation
- `Rodríguez-Fernández et al. (2001, A&A, 365, 174) <https://ui.adsabs.harvard.edu/abs/2001A%26A...365..174R/abstract>`_

Physical Process
~~~~~~~~~~~~~~~~
Supersonic turbulence in molecular clouds decays on approximately a crossing time scale. The dissipated energy heats the gas. This mechanism is most relevant in regions with strong turbulent motions, such as galactic centers.

The turbulent heating rate follows:

.. math::
   \Gamma_{\mathrm{turb}} = \rho \times \frac{v_{\mathrm{turb}}^3}{L_{\mathrm{turb}}}

where:

- :math:`\rho` = Gas mass density (g cm⁻³)
- :math:`v_{\mathrm{turb}}` = Turbulent velocity (cm s⁻¹)
- :math:`L_{\mathrm{turb}}` = Turbulent driving scale (cm)


Complete Expression
~~~~~~~~~~~~~~~~~~~

.. math::
   
   \boxed{\Gamma_{\mathrm{turb}} = 3.5 \times 10^{-28} \times \left(\frac{v_{\mathrm{turb}}}{10^5\ \mathrm{cm\ s}^{-1}}\right)^3 \times \left(\frac{1}{L_{\mathrm{turb}}}\right) \times n_{\mathrm{H}}}
   
with :math:`L_{\mathrm{turb}}` in parsecs (pc).

In terms of physical quantities:

.. math::
   \Gamma_{\mathrm{turb}} = 3.5 \times 10^{-28} \times \left(\frac{v_{\mathrm{turb}}}{1\ \mathrm{km\ s}^{-1}}\right)^3 \times \left(\frac{1\ \mathrm{pc}}{L_{\mathrm{turb}}}\right) \times n_{\mathrm{H}}\ \mathrm{erg\ cm}^{-3}\ \mathrm{s}^{-1}

The factor 3.5 × 10⁻²⁸ comes from:

- Mass density: :math:`\rho = n_{\mathrm{H}} m_{\mathrm{H}}` (with :math:`m_{\mathrm{H}} = 1.67 \times 10^{-24}` g)
- Velocity conversion: 1 km s⁻¹ = 10⁵ cm s⁻¹
- Length conversion: 1 pc = 3.086 × 10¹⁸ cm

Thus: :math:`3.5 \times 10^{-28} \approx \frac{m_{\mathrm{H}}}{(10^5)^3 \times (3.086 \times 10^{18})} \times \text{(dimensionless efficiency factor)}`

Turbulent Parameters
~~~~~~~~~~~~~~~~~~~~

1. **Turbulent Velocity** (:math:`v_{\mathrm{turb}}`):

   - Galactic center: ~15 km s⁻¹
   - Typical molecular clouds: 1-5 km s⁻¹
   - Input units: km s⁻¹ (converted to cm s⁻¹ in formula)

2. **Turbulent Scale Length** (:math:`L_{\mathrm{turb}}`):

   - Default: 5.0 pc (as set in code)
   - Typical range: 0.1-100 pc depending on environment

3. **Density**: turbulence heats all gas

Dissipation Timescale
~~~~~~~~~~~~~~~~~~~~~
The turbulent dissipation timescale is:

.. math::
   t_{\mathrm{diss}} \approx \frac{L_{\mathrm{turb}}}{v_{\mathrm{turb}}} \approx 10^6\ \mathrm{yr} \times \left(\frac{L_{\mathrm{turb}}}{5\ \mathrm{pc}}\right) \left(\frac{v_{\mathrm{turb}}}{5\ \mathrm{km\ s}^{-1}}\right)^{-1}

Notes
~~~~~
1. This heating mechanism is most significant in regions with strong turbulence (e.g., galactic centers, starburst regions).
2. The simple :math:`v^3/L` scaling assumes Kolmogorov turbulence and isotropic dissipation.
3. The default scale length of 5 pc is appropriate for Galactic molecular clouds but may need adjustment for other environments.
4. Turbulent heating can be comparable to or exceed other heating mechanisms in strongly turbulent regions.
5. The heating rate scales linearly with density but cubically with velocity, making it very sensitive to the turbulent Mach number.


-----------------------------
Exothermic Reaction Heating
-----------------------------

This routine computes the heating from exothermic chemical reactions. When chemical bonds form or rearrange, the released energy (reaction enthalpy) can be deposited as thermal energy into the gas.

For each exothermic reaction:
:math:`\Gamma_{\mathrm{chem}} = n_1 n_2 \dots \times k \times E`
where:

- :math:`n_1, n_2, \dots` = Number densities of the reactants (cm⁻³)
- :math:`k` = Reaction rate coefficient (cm³ s⁻¹)
- :math:`E` = Energy released per reaction (erg)

.. note:: **Automatic reaction identification (module** ``chemical_heating.F90`` **)**

   The heating reactions are no longer picked out by hard-wired ``RATE(i)`` indices.
   Instead they are identified automatically by their chemistry, in the module
   ``chemical_heating_module`` (file ``chemical_heating.F90``). A single,
   network-independent table lists each canonical exothermic channel as
   reactants → products plus its exothermicity in eV. At startup,
   ``init_chemical_heating`` scans the loaded network (called once, right after
   the rate file is read) and tags every reaction whose reactants **and**
   products match a table entry, comparing them as unordered sets so that the
   order in which products are listed in the rate file does not matter.
   ``CHEMICAL_HEATING`` in ``CALC_HEATING`` then simply calls
   ``CHEMICAL_HEATING_RATE(ABUNDANCE,DENSITY,RATE,NSPEC,NREAC)``, which sums the
   contribution of every tagged reaction - no per-network editing of
   ``heatingfunctions.F90`` is required any more.

   This automatic tagging correctly handles:

   - **Channels absent from a given network** - simply skipped.
   - **Reactions split over several temperature ranges** in the rate file
     (duplicate reactant/product entries) - all duplicates are tagged, but only
     the one active at the local temperature has a non-zero ``RATE``, so the sum
     is still correct.
   - **Reordered products** - matched regardless of listing order.

   A start-up report lists every tagged reaction (index, reactants → products,
   eV) together with any canonical channel not found in the network, so the
   tagging can be checked whenever the chemical network is changed. To add a
   heating channel for a future network, add an entry to the table in
   ``init_chemical_heating`` (reactants, products, energy) - never an index.

The canonical table currently includes 12 exothermic channels:

1. **H₂⁺ recombination**: :math:`\mathrm{H}_2^+ + e^- \rightarrow \mathrm{H} + \mathrm{H}` (10.90 eV)
2. **H₂⁺ charge transfer**: :math:`\mathrm{H}_2^+ + \mathrm{H} \rightarrow \mathrm{H}_2 + \mathrm{H}^+` (0.94 eV)
3. **HCO⁺ recombination**: :math:`\mathrm{HCO}^+ + e^- \rightarrow \mathrm{CO} + \mathrm{H}` (7.51 eV)
4. **H₃⁺ recombination (a)**: :math:`\mathrm{H}_3^+ + e^- \rightarrow \mathrm{H} + \mathrm{H} + \mathrm{H}` (4.76 eV)
5. **H₃⁺ recombination (b)**: :math:`\mathrm{H}_3^+ + e^- \rightarrow \mathrm{H}_2 + \mathrm{H}` (9.23 eV)
6. **H₃O⁺ recombination (a)**: :math:`\mathrm{H}_3\mathrm{O}^+ + e^- \rightarrow \mathrm{OH} + \mathrm{H} + \mathrm{H}` (1.16 eV)
7. **H₃O⁺ recombination (b)**: :math:`\mathrm{H}_3\mathrm{O}^+ + e^- \rightarrow \mathrm{OH} + \mathrm{H}_2` (5.63 eV)
8. **H₃O⁺ recombination (c)**: :math:`\mathrm{H}_3\mathrm{O}^+ + e^- \rightarrow \mathrm{H}_2\mathrm{O} + \mathrm{H}` (6.27 eV)
9. **H₃O⁺ recombination (d)**: :math:`\mathrm{H}_3\mathrm{O}^+ + e^- \rightarrow \mathrm{O} + \mathrm{H}_2 + \mathrm{H}` (1.23 eV)
10. **He⁺ + H₂ ion-neutral**: :math:`\mathrm{He}^+ + \mathrm{H}_2 \rightarrow \mathrm{He} + \mathrm{H}^+ + \mathrm{H}` (6.51 eV)
11. **He⁺ + H₂ charge transfer**: :math:`\mathrm{He}^+ + \mathrm{H}_2 \rightarrow \mathrm{He} + \mathrm{H}_2^+` (9.16 eV)
12. **He⁺ + CO**: :math:`\mathrm{He}^+ + \mathrm{CO} \rightarrow \mathrm{He} + \mathrm{C}^+ + \mathrm{O}` (2.22 eV)

.. note::

   Channels 10 and 11 were previously merged in the manually-coded blocks (both
   given 6.51 eV); the automatic table separates them and assigns 9.16 eV to the
   charge-transfer channel, its correct exothermicity. Channel 9 (H₃O⁺ + e⁻ → O +
   H₂ + H) is a real channel present in several networks that the old hard-coded
   ``REDUCED`` block omitted; it is now picked up automatically whenever the
   network contains it. A minor He⁺ + CO channel producing O⁺ + C + He
   (rate ~10⁻¹⁶) is intentionally **not** included in the table.

Mathematical Expressions
~~~~~~~~~~~~~~~~~~~~~~~~

.. math::
   \Gamma_{\mathrm{chem}} = \sum_{r=1}^{N_{\mathrm{tagged}}} \left[\prod_{j} f_{j,r}\, n_{\mathrm{H}}\right] \times \mathrm{RATE}(r) \times E_r

where the sum runs over every reaction :math:`r` tagged by ``init_chemical_heating``
in the currently loaded network (:math:`N_{\mathrm{tagged}} \leq 12`), the product
runs over the (2 or 3) reactants :math:`j` of that reaction with fractional
abundance :math:`f_{j,r}`, and :math:`E_r` is the exothermicity listed in the table
above. Each term therefore scales as :math:`n_{\mathrm{H}}^{2}` (two reactants) or
:math:`n_{\mathrm{H}}^{3}` (three-body reactions, if present in the network),
consistent with :math:`n_{\mathrm{H}} = n_{\mathrm{H}} f_j` for each reactant's
number density.

.. note::

   The H₂⁺ terms (channels 1-2) now correctly use :math:`n_{\mathrm{H}}^2`
   (number density of *both* reactants); the previous manually-coded blocks used
   :math:`n_{\mathrm{H}}^1` for these two channels, under-estimating the rate at
   high density.

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


-------------------------
Total Heating Rate
-------------------------

The total heating rate used in the thermal balance is the sum of all
active contributions:

.. math::

   \boxed{\Gamma_{\rm total} =
   \sum_i \Gamma_i}

The table below provides the index of the heating rate for all the above functions. Note that indices 1 and 3 are missing because they do not contribute to the total heating.

+----------------------+--------------------------+----------------------------------------+
| **Index**            | **Variable Name**        | **Physical Process**                   |
+======================+==========================+========================================+
| ``HEATING_RATE(2)``  | ``PAHPHOTOELEC_HEATING`` | PAH photoelectric heating              |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(4)``  | ``CIONIZATION_HEATING``  | Carbon photoionization heating         | 
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(5)``  | ``H2FORMATION_HEATING``  | H₂ formation heating                   |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(6)``  | ``H2PHOTODISS_HEATING``  | H₂ photodissociation heating           |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(7)``  | ``FUVPUMPING_HEATING``   | H₂ FUV pumping heating                 |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(8)``  | ``COSMICRAY_HEATING``    | Cosmic-ray ionization heating          |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(9)``  | ``TURBULENT_HEATING``    | Supersonic turbulent decay heating     |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(10)`` | ``CHEMICAL_HEATING``     | Exothermic chemical reaction heating   |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(11)`` | ``GASGRAIN_HEATING``     | Gas–grain collisional heating/cooling  |
+----------------------+--------------------------+----------------------------------------+
| ``HEATING_RATE(12)`` | **Total Heating Rate**   | Sum of all heating mechanisms          |
+----------------------+--------------------------+----------------------------------------+
