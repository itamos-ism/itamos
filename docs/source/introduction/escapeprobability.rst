Escape Probability
==================

Radiative line emission is a key mechanism regulating the thermal
balance of astrophysical gas. In many interstellar environments,
however, spectral lines become optically thick, and photons emitted
in atomic or molecular transitions may undergo multiple absorption
and re-emission events before escaping the medium. A full treatment
of this problem requires solving the radiative transfer equation,
which is computationally demanding, particularly in multi-level
and multi-dimensional systems. This formalism
provides a physically motivated approximation that captures the
essential effects of radiative trapping while remaining tractable.

The escape probability, usually denoted by :math:`\beta`, represents the
probability that a photon emitted in a given transition can leave the system
without being reabsorbed. In optically thin conditions (i.e. where :math:`\tau \ll 1`),
photons escape freely and :math:`\beta \rightarrow 1`. In optically thick
media (i.e. where :math:`\tau \gg 1`), photons are trapped and :math:`\beta \ll 1`,
reducing the effective cooling rate.


Statistical Equilibrium and Radiation Field
-------------------------------------------

Consider a parcel of gas that emits and absorbs radiation through
bound–bound transitions between discrete energy levels. Under the
assumption of statistical equilibrium, the population of each level
is governed by the balance between all upward and downward
radiative and collisional processes,

.. math::

   n_i \sum_{j \ne i} R_{ij} = \sum_{j \ne i} n_j R_{ji},

where :math:`n_i` and :math:`n_j` are the populations of levels
:math:`i` and :math:`j`, respectively, and :math:`R_{ij}` denotes the
total transition rate from level :math:`i` to :math:`j`.

For a given transition, the rate coefficients include spontaneous
emission, stimulated emission or absorption, and collisional
excitation or de-excitation. These rates depend on the local radiation
field through the angle-averaged mean intensity
:math:`\langle J_{ij} \rangle`, which couples the level populations to
the surrounding medium.

Escape Probability Approximation
--------------------------------

In the escape probability (or large velocity gradient, LVG; `Sobolev 1960 <https://ui.adsabs.harvard.edu/abs/1960mes..book.....S/abstract>`_; `Castor 1970 <https://ui.adsabs.harvard.edu/abs/1970MNRAS.149..111C/abstract>`_; `de Jong et al. 1975 <https://ui.adsabs.harvard.edu/abs/1975ApJ...199...69D/abstract>`_; `Poelman & Spaans 2005 <https://ui.adsabs.harvard.edu/abs/2005A%26A...440..559P/abstract>`_)
approximation, the mean radiation field is expressed as a weighted
sum of locally produced line radiation and an external background
field,

.. math::

   \langle J_{ij} \rangle = \left[ 1 - \beta_{ij} \right] S_{ij}
   + \beta_{ij} B(\nu_{ij}),

where :math:`S_{ij}` is the line source function, :math:`B(\nu_{ij})`
represents the background radiation at the transition frequency
:math:`\nu_{ij}`, and :math:`\beta_{ij}` is the escape probability.

The source function for a transition between levels :math:`i` and
:math:`j` is determined by the local level populations and statistical
weights,

.. math::

   S_{ij} = \frac{2 h \nu_{ij}^3}{c^2}
   \left( \frac{n_i g_j}{n_j g_i} - 1 \right)^{-1},

where :math:`g_i` and :math:`g_j` are the statistical weights of the
upper and lower levels, respectively.

The escape probability :math:`\beta_{ij}` represents the probability
that a photon emitted in the transition :math:`i \rightarrow j`
leaves the system without being reabsorbed. In optically thin
conditions, :math:`\beta_{ij} \rightarrow 1`, while in optically thick
media radiative trapping reduces :math:`\beta_{ij}` and suppresses
the effective radiative cooling.

Optical Depth Dependence
------------------------

The escape probability is primarily determined by the line optical
depth :math:`\tau_{ij}`, which measures the opacity of the medium
to photons of frequency :math:`\nu_{ij}`. For line radiation, and
neglecting continuum absorption, the optical depth along a given
direction is given by

.. math::

   \tau_{ij} = \frac{A_{ij} c^3}{8 \pi \nu_{ij}^3}
   \int \frac{n_i}{u}
   \left( \frac{n_j g_i}{n_i g_j} - 1 \right) \, \mathrm{d}r,

where :math:`A_{ij}` is the Einstein coefficient for spontaneous
emission, :math:`u` is the local velocity dispersion (including
thermal and non-thermal contributions), and the integral is performed
along the photon path.

Analytical Expression
---------------------

The analytical expression of the escape probability (`de Jong et al. 1975 <https://ui.adsabs.harvard.edu/abs/1975ApJ...199...69D/abstract>`_) is given by

.. math::

   \beta_{ij} = \int_{0}^{4\pi} \frac{d\Omega}{4\pi} \left[\frac{1-e^{-\tau_{ij}}}{\tau_{ij}}\right]

For simple geometries, analytic expressions are often used to relate
the escape probability to the optical depth, capturing the essential
physics of photon trapping while avoiding explicit radiative transfer
calculations. These approximations form the basis of many
non-LTE excitation and cooling models used in studies of the
interstellar medium.

For example, commonly used expressions include

.. math::

   \beta_{ij}(\tau_{ij}) = \frac{1 - e^{-\tau_{ij}}}{\tau_{ij}},

for a uniform slab, and

.. math::

   \beta_{ij}(\tau_{ij}) = \frac{1}{1 + \tau_{ij}},

as a commonly used approximation for spherical or isotropic media.

