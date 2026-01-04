Shielding Functions
===================

Overview
--------

The routines in ``shield.F90`` compute **attenuated photodissociation and
photoionization rates** for key species in photodissociation regions (PDRs),
accounting for:

* Dust extinction and scattering
* Molecular self-shielding
* Mutual shielding by H₂
* Line overlap effects
* Temperature-dependent shielding (for CI)

These functions follow well-established prescriptions from the literature,
primarily:

* `Federman, Glassgold & Kwan (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJ...227..466F/abstract>`_
* `van Dishoeck & Black (1988) <https://ui.adsabs.harvard.edu/abs/1988ApJ...334..771V/abstract>`_
* `Lee et al. (1996) <https://ui.adsabs.harvard.edu/abs/1996A%26A...311..690L/abstract>`_
* `Kamp & Bertoldi (2000) <https://ui.adsabs.harvard.edu/abs/2000A%26A...353..276K/abstract>`_
* `Savage & Mathis (1979) <https://ui.adsabs.harvard.edu/abs/1979ARA%26A..17...73S/abstract>`_
* `Wagenblast & Hartquist (1989) <https://ui.adsabs.harvard.edu/abs/1989MNRAS.237.1019W/abstract>`_

All rates scale linearly with the incident FUV field :math:`G_0`
(in Draine units).

General Form of Photoreaction Rates
-----------------------------------

All photorates computed here follow the generic structure:

.. math::

   k = k_0 \, G_0 \, f_{\mathrm{shield}} \, f_{\mathrm{dust}}

where:

* :math:`k_0` is the unattenuated rate
* :math:`f_{\mathrm{shield}}` accounts for line shielding
* :math:`f_{\mathrm{dust}}` accounts for dust extinction and scattering

H₂ Photodissociation Rate
-------------------------

``H2PDRATE(K0, G0, AV, NH2)``

This function computes the H₂ photodissociation rate including **self-shielding**
and **dust attenuation**.

.. math::

   k_{\mathrm{H_2}} =
   k_0 \, G_0 \,
   f_{\mathrm{H_2}}(N_{\mathrm{H_2}})
   \times f_{\mathrm{dust}}(A_V, \lambda)

Self-Shielding
^^^^^^^^^^^^^^

Self-shielding is computed using the `Federman et al. (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJ...227..466F/abstract>`_ formalism via
``H2SHIELD1``.

The Doppler linewidth is determined by turbulent broadening:

.. math::

   \Delta \nu_D = \frac{v_{\mathrm{turb}}}{\lambda}

where a representative wavelength of :math:`\lambda = 1000\,Å` is
assumed.

CO Photodissociation Rate
-------------------------

``COPDRATE(K0, G0, AV, NCO, NH2)``

The CO photodissociation rate includes **CO self-shielding**, **H₂ shielding**,
and **dust extinction**:

.. math::

   k_{\mathrm{CO}} =
   k_0 \, G_0 \,
   f_{\mathrm{CO}}(N_{\mathrm{CO}}, N_{\mathrm{H_2}})
   \times f_{\mathrm{dust}}(A_V, \bar{\lambda})

The mean wavelength :math:`\bar{\lambda}` of the dissociating bands is computed
using ``LBAR`` following `van Dishoeck & Black (1988) <https://ui.adsabs.harvard.edu/abs/1988ApJ...334..771V/abstract>`_.

CI Photoionization Rate
-----------------------

``CIPDRATE(K0, G0, AV, KAV, NCI, NH2, TGAS)``

The CI photoionization rate follows `Kamp & Bertoldi (2000) <https://ui.adsabs.harvard.edu/abs/2000A%26A...353..276K/abstract>`_, Equation (8):

.. math::

   k_{\mathrm{CI}} = k_0 \, G_0 \, e^{-\tau_{\mathrm{CI}}}

with optical depth:

.. math::

   \tau_{\mathrm{CI}} =
   K_{AV} A_V
   + 1.1 \times 10^{-17} N_{\mathrm{CI}}
   + 0.9 \, T_{\mathrm{gas}}^{0.27}
     \left( \frac{N_{\mathrm{H_2}}}{1.59 \times 10^{21}} \right)^{0.45}

This formulation accounts for:

* Dust extinction
* CI line shielding
* Overlapping H₂ absorption lines
* Temperature-dependent shielding effects

SI Photoionization Rate
-----------------------

``SIPDRATE(K0, G0, AV, KAV)``

Currently implemented as a **dust-only attenuated rate**:

.. math::

   k_{\mathrm{SI}} = k_0 \, G_0 \, e^{-K_{AV} A_V} / 2

This routine is a placeholder and does **not yet include line shielding**.

H₂ Self-Shielding (Analytic)
----------------------------

``H2SHIELD1(NH2, DOPW, RADW)``

Implements the `Federman, Glassgold & Kwan (1979) <https://ui.adsabs.harvard.edu/abs/1979ApJ...227..466F/abstract>`_ shielding function:

.. math::

   f_{\mathrm{H_2}} = J_D + J_R

where:

* :math:`J_D` is the Doppler core contribution
* :math:`J_R` is the radiative wing contribution

The optical depth at line centre is:

.. math::

   \tau_D =
   N_{\mathrm{H_2}} \,
   f_{\mathrm{para}} \,
   \frac{\pi e^2}{m c} \,
   \frac{f}{\sqrt{\pi} \Delta \nu_D}

A fixed ortho–para ratio of 1 is assumed (:math:`f_{\mathrm{para}} = 0.5`).

H₂ Shielding (Tabulated)
------------------------

``H2SHIELD2(NH2)``

Alternative H₂ shielding based on tabulated values from
`Lee et al. (1996) <https://ui.adsabs.harvard.edu/abs/1996A%26A...311..690L/abstract>`_, Table 10.

* Shielding factors are interpolated using cubic splines.
* Includes shielding by both H₂ and atomic H.
* Used mainly for validation or comparison.

CO Shielding
------------

``COSHIELD(NCO, NH2)``

Computes combined **CO self-shielding** and **H₂ shielding** using
two-dimensional spline interpolation over the tables of
`van Dishoeck & Black (1988) <https://ui.adsabs.harvard.edu/abs/1988ApJ...334..771V/abstract>`_, Table 5.

Interpolation is performed in:

.. math::

   \log_{10}(N_{\mathrm{CO}}), \quad \log_{10}(N_{\mathrm{H_2}})

Dust Scattering Attenuation
---------------------------

``SCATTER(AV, LAMBDA)``

Computes attenuation due to **dust scattering and absorption**, following
`Wagenblast & Hartquist (1989) <https://ui.adsabs.harvard.edu/abs/1989MNRAS.237.1019W/abstract>`_ and `Flannery et al. (1980) <https://ui.adsabs.harvard.edu/abs/1980ApJ...236..598F/abstract>`_.

The optical depth at wavelength :math:`\lambda` is:

.. math::

   \tau_\lambda = \frac{A_V}{1.086} \times
   \frac{\tau(\lambda)}{\tau(V)}

The attenuation factor is piecewise:

* For :math:`\tau_\lambda < 1`:

  .. math::

     f = a_0 \, e^{-k_0 \tau_\lambda}

* For :math:`\tau_\lambda \ge 1`:

  .. math::

     f = \sum_{i=1}^5 a_i \, e^{-k_i \tau_\lambda}

The coefficients assume:

* Dust albedo = 0.3
* Scattering asymmetry parameter = 0.8

Extinction Curve
----------------

``XLAMBDA(LAMBDA)``

Computes the ratio:

.. math::

   \frac{\tau(\lambda)}{\tau(V)}

using spline interpolation over the extinction curve of
`Savage & Mathis (1979) <https://ui.adsabs.harvard.edu/abs/1979ARA%26A..17...73S/abstract>`_.

Mean CO Band Wavelength
-----------------------

``LBAR(NCO, NH2)``

Computes the mean wavelength of the 33 CO-dissociating bands:

.. math::

   \bar{\lambda} =
   (5675 - 200.6 W)
   - (571.6 - 24.09 W) U
   + (18.22 - 0.7664 W) U^2

where:

.. math::

   U = \log_{10}(N_{\mathrm{CO}}), \quad
   W = \log_{10}(N_{\mathrm{H_2}})

The wavelength is constrained to:

.. math::

   913.6\,\mathrm{\AA} \le \bar{\lambda} \le 1076.1\,\mathrm{\AA}

Summary
-------

The shielding and photorate routines in ``shield.F90`` provide a
physically consistent and literature-calibrated treatment of
UV attenuation in PDRs, accounting for:

* Line self-shielding
* Mutual shielding
* Dust extinction and scattering
* Temperature-dependent effects

They form a critical link between **radiative transfer**, **chemistry**, and
**thermal balance** in the 3D-PDR framework.
