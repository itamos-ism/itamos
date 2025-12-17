=================
Heating Processes
=================

This file documents the heating terms implemented in the
``CALC_HEATING`` subroutine of the 3D-PDR code. Each heating mechanism
contributes to the total volumetric heating rate of the gas
(:math:`\mathrm{erg\ cm^{-3}\ s^{-1}}`) at a given grid cell, based on the
local physical conditions and chemical abundances.

The total heating rate is computed as the sum of the individual
processes described below.

Only the heating mechanisms that are **active in the default energy
balance** are described here. Dust photoelectric heating (classical),
the Weingartner & Draine formulation, soft X-ray heating, and mechanical
heating are intentionally omitted.

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

Photoelectric heating from polycyclic aromatic hydrocarbons (PAHs) and
very small grains is included using the formulation of
Bakes & Tielens (1994), with updates from Wolfire et al. (2003, 2008).

This mechanism accounts for both heating and cooling due to PAH charge
exchange with the gas.

The net PAH photoelectric heating rate is

.. math::

   \Gamma_{\rm PAH} =
   \left( \Gamma_{\rm heat} - \Lambda_{\rm cool} \right) Z

where :math:`Z` is the metallicity scaling factor.

The heating efficiency depends on the local FUV radiation field
(converted internally from Draine to Habing units), the gas temperature,
and the electron abundance.

This term is stored as:

- ``HEATING_RATE(2)`` — PAH photoelectric heating

--------------------------------
Carbon Photoionization Heating
--------------------------------

Photoionization of neutral carbon deposits kinetic energy into the gas.
An average energy of 1 eV per ionization is assumed.

The heating rate is computed as

.. math::

   \Gamma_{\rm C^0} =
   (1~{\rm eV}) \, k_{\rm ion}(C) \, n({\rm C}) \, n_{\rm H}

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
(Hollenbach & Tielens 1999).

The heating rate is

.. math::

   \Gamma_{\rm H_2,form} =
   (1.5~{\rm eV}) \, k_{\rm gr} \, n({\rm H}) \, n_{\rm H}

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

   \Gamma_{\rm H_2,diss} =
   (0.4~{\rm eV}) \, k_{\rm diss} \, n({\rm H_2}) \, n_{\rm H}

Stored as:

- ``HEATING_RATE(6)`` — H₂ photodissociation heating

-------------------------
H₂ FUV Pumping Heating
-------------------------

FUV excitation of H₂ followed by collisional de-excitation contributes
to gas heating. Each vibrationally excited H₂ molecule deposits on
average 2.2 eV (Hollenbach & McKee 1979).

Collisional quenching is accounted for using a critical density
formalism.

The heating rate is

.. math::

   \Gamma_{\rm pump} =
   \frac{(2.2~{\rm eV}) \, 9 \, k_{\rm diss} \, n({\rm H_2}) \, n_{\rm H}}
        {1 + n_{\rm cr}/n_{\rm H}}

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
Chemical Heating
-----------------------------

Exothermic chemical reactions deposit energy into the gas. The dominant
contributors are dissociative recombination and ion–neutral reactions,
including:

- HCO⁺ + e⁻
- H₃⁺ + e⁻
- H₃O⁺ + e⁻
- He⁺ + H₂
- He⁺ + CO

For each reaction, the heating rate is

.. math::

   \Gamma = n_1 \, n_2 \, k \, E

where :math:`k` is the reaction rate coefficient and :math:`E` is the
released energy.

The exact set of reactions depends on the selected chemical network
(``REDUCED``, ``MEDIUM``, or ``FULL``).

Stored as:

- ``HEATING_RATE(10)`` — chemical heating

-------------------------------
Gas–Grain Collisional Heating
-------------------------------

Collisions between gas particles and dust grains exchange thermal
energy. This process can either heat or cool the gas, depending on the
temperature difference.

The formulation follows Burke & Hollenbach (1983), with a temperature-
dependent accommodation coefficient.

The heating rate scales as

.. math::

   \Gamma_{\rm gg} \propto
   n_{\rm H} \, n_{\rm grain} \,
   \sqrt{T_{\rm gas}} \,
   (T_{\rm dust} - T_{\rm gas})

Stored as:

- ``HEATING_RATE(11)`` — gas–grain collisional heating

-------------------------
Total Heating Rate
-------------------------

The total heating rate used in the thermal balance is the sum of all
active contributions:

.. math::

   \Gamma_{\rm total} =
   \sum_i \Gamma_i

Stored as:

- ``HEATING_RATE(12)`` — total heating rate
