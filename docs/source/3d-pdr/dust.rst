==========================
Dust Temperature Treatment
==========================

This file documents the calculation of the dust temperature in the
``CALCULATE_DUST_TEMPERATURES`` subroutine of the 3D-PDR code.

The dust temperature is computed self-consistently for each grid
particle by balancing heating from the local far-ultraviolet (FUV)
radiation field, reprocessed infrared (IR) radiation, and the cosmic
microwave background (CMB). The adopted formulation is based primarily
on the work of `Hollenbach, Takahashi & Tielens (1991; hereafter "HTT91") <https://ui.adsabs.harvard.edu/abs/1991ApJ...377..192H/abstract>`_, with important
modifications to account for attenuation of the re-emitted IR radiation
into the cloud.

-------------------------
Physical Motivation
-------------------------

The dust temperature plays a central role in the thermal and chemical
evolution of photodissociation regions. In 3D-PDR it affects:

1. Gas cooling via far-infrared dust emission coupled to line radiative
   transfer;
2. Gas–grain collisional heating or cooling;
3. H\ :sub:`2` formation through the temperature-dependent sticking
   probability;
4. Freeze-out, evaporation, and surface chemistry on grains.

Accurate modelling of the dust temperature is therefore essential for a
realistic description of both the gas energetics and the chemistry.

-------------------------
Adopted Formalism
-------------------------

The dust temperature calculation follows the treatment of HTT91, who derive analytic
expressions for dust heating by FUV photons.

For each particle, an initial contribution to the dust emission is
computed from:

- the local isotropic FUV radiation field, and
- the cosmic microwave background (CMB), included as a temperature floor.

This is expressed as a sum of fifth powers of temperature, consistent
with radiative equilibrium of dust grains.

-------------------------
Directional FUV Heating
-------------------------

The directional dependence of the radiation field is explicitly taken
into account by looping over all rays associated with a particle.

For each ray:

- The incident FUV field at the cloud surface is converted from Draine
  units to Habing units by a factor of 1.71.
- A characteristic surface dust temperature :math:`T_0` is computed as

  .. math::

     T_0 = 12.2 \, G_0^{0.2}

  where :math:`G_0` is the local FUV field in Habing units.

- For rays with visual extinction :math:`A_V > 1`, the temperature is
  attenuated according to

  .. math::

     T_0 \propto A_V^{-0.4}

This prescription follows the infrared-only dust temperature
approximation of Rowan-Robinson (1980).

The contribution from each ray is added to the total dust emission using
the modified HTT91 formulation, which depends on the effective dust
optical depth at 100 μm.

-------------------------
Infrared Attenuation
-------------------------

In the original HTT91 treatment, the infrared radiation re-emitted
by dust at the cloud surface is not attenuated as it propagates deeper
into the cloud. This leads to unrealistically high dust (and therefore
gas) temperatures at large depths for high-density, high-FUV models.

To address this issue, 3D-PDR introduces attenuation of the reprocessed
IR radiation following Rowan-Robinson (1980). The dust temperature
decreases with depth as

.. math::

   T_{\rm dust} \propto r^{-0.4}

where :math:`r` corresponds to the depth into the cloud, normalized such
that :math:`A_V \simeq 1` marks the outer FUV-processed layer.

This modification prevents excessive heating deep inside the cloud and
yields more realistic dust and gas temperatures at high extinctions.

-------------------------
Final Dust Temperature
-------------------------

After summing all directional and isotropic contributions, the dust
temperature is obtained by converting the total emission intensity to a
temperature:

.. math::

   T_{\rm dust} = I^{1/5}

where :math:`I` is the accumulated emission term.

-------------------------
Temperature Limits
-------------------------

Two safeguards are applied:

- **Lower limit:**  
  The dust temperature is not allowed to fall below 10 K. Lower values
  would strongly suppress H\ :sub:`2` formation by preventing molecule
  desorption from grain surfaces.

- **Upper limit:**  
  If the computed dust temperature exceeds 1000 K, the code terminates,
  as such values are considered unphysical in the context of PDR
  modelling.

-------------------------
Summary
-------------------------

The dust temperature treatment in 3D-PDR:

- Accounts for both isotropic and directional FUV heating;
- Includes heating from the CMB;
- Incorporates attenuation of reprocessed infrared radiation;
- Avoids unphysical dust and gas temperatures at large cloud depths;
- Provides a physically motivated and numerically stable dust
  temperature for use in the thermal and chemical balance.
