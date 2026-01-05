Escape Probability and Line Cooling
===================================

This routine computes molecular line cooling and the mean radiation field
using the **Large Velocity Gradient (LVG) escape probability approximation**
along multiple rays (see description :ref:`here <sec-ep>`). It is designed for use within the 3D-PDR framework and
operates on a given grid cell, using level populations obtained from the
statistical equilibrium solver.

The routine performs the following tasks:

* Computes the optical depth :math:`\tau_{ij}` for each radiative transition
  :math:`i \rightarrow j` along all rays.
* Computes the ray-by-ray escape probability :math:`\beta_{ij}^{(r)}`.
* Averages the escape probability over rays to obtain :math:`\beta_{ij}`.
* Evaluates the source function :math:`S_{ij}`.
* Computes line cooling rates.
* Updates the mean radiation field :math:`\langle J_{ij} \rangle`.
* Constructs the radiative transition matrix used by the level population solver.

The LVG approximation assumes that velocity gradients Doppler-shift photons
out of resonance, allowing radiative transfer to be treated locally via
escape probabilities.

Physical Assumptions
--------------------

* Statistical equilibrium holds for all molecular levels.
* Line transfer is treated locally using escape probabilities.
* Velocity broadening includes both thermal and microturbulent components.
* The radiation field includes contributions from:
  
  * Cosmic Microwave Background (CMB)
  * Thermal dust emission

* Strong maser action is regulated to avoid numerical divergence.

Key Quantities
--------------

Velocity Dispersion
^^^^^^^^^^^^^^^^^^^

The effective inverse velocity dispersion entering the LVG optical depth is

.. math::

   \phi^{-1} = \frac{1}{\sqrt{ \frac{2 k_B T}{m} + v_{\mathrm{turb}}^2 }}

where:

* :math:`T` is the gas temperature
* :math:`m` is the molecular mass
* :math:`v_{\mathrm{turb}}` is the microturbulent velocity

This factor is stored in the code as ``frac2``.

Einstein Prefactor
^^^^^^^^^^^^^^^^^^

For each transition :math:`i \rightarrow j`, the LVG optical depth prefactor is

.. math::

   f_{ij} = \frac{A_{ij} c^3}{8 \pi \nu_{ij}^3}

implemented as ``frac1``.

Optical Depth Calculation
-------------------------

The optical depth for a transition :math:`i \rightarrow j` along ray :math:`r`
is computed as a sum over ray segments:

.. math::

   \tau_{ij}^{(r)} = \sum_k f_{ij} \, \phi^{-1}
   \left[
     \frac{n_j g_i - n_i g_j}{g_j}
   \right]_k
   \Delta s_k

where:

* :math:`n_i`, :math:`n_j` are the level populations
* :math:`g_i`, :math:`g_j` are the statistical weights
* :math:`\Delta s_k` is the path length of segment :math:`k`
* The populations are evaluated at each ray integration point

In the code, this is accumulated as ``tau_ij(j)``.

One-dimensional models
^^^^^^^^^^^^^^^^^^^^^^

When the ``ONEDIMENSIONAL`` flag is enabled:

* Only one ray (ray index 6) contributes to radiative transfer.
* All other rays are assigned extremely large optical depths:

.. math::

   \tau_{ij}^{(r \neq 6)} \rightarrow 10^{50}

This effectively enforces 1D slab geometry while preserving the ray-based
infrastructure.

Escape Probability
------------------

The escape probability for each ray is computed using the standard LVG form:

.. math::

   \beta(\tau) = \frac{1 - e^{-\tau}}{\tau}

Special Cases
^^^^^^^^^^^^^

To ensure numerical stability and physical realism, the following limits
are applied:

* **Strong masing** (:math:`\tau < -5`):

  .. math::

     \beta = \frac{1 - e^{5}}{-5}

* **Weak masing** (:math:`-5 \le \tau < 0`):

  Standard LVG expression is used.

* **Optically thin limit** (:math:`|\tau| < 10^{-8}`):

  .. math::

     \beta = 1

These limits prevent floating-point overflow and unphysical cooling rates.

Mean Escape Probability
^^^^^^^^^^^^^^^^^^^^^^

The total escape probability for a transition is the average over all rays:

.. math::

   \beta_{ij} =
   \begin{cases}
     \sum_r \beta_{ij}^{(r)} & \text{(ONEDIMENSIONAL)} \\
     \frac{1}{N_{\mathrm{rays}}} \sum_r \beta_{ij}^{(r)} & \text{(3D)}
   \end{cases}

Source Function
---------------

The line source function is computed as

.. math::

   S_{ij} =
   \begin{cases}
     \dfrac{2 h \nu_{ij}^3}{c^2}
     \left[
       \dfrac{1}{\dfrac{n_j g_i}{n_i g_j} - 1}
     \right] & n_i \neq 0 \\
     0 & n_i = 0
   \end{cases}

This follows the formulation used in UCL\_PDR and is valid for arbitrary
population inversions.

Background Radiation Field
--------------------------

The background radiation field consists of:

CMB Contribution
^^^^^^^^^^^^^^^^

.. math::

   B_\nu(T_{\mathrm{cmb}}) =
   \frac{2 h \nu^3}{c^2}
   \left[
     \frac{1}{e^{h\nu / k_B T_{\mathrm{cmb}}} - 1}
   \right]

Dust Emission
^^^^^^^^^^^^^

Thermal dust emission is included as

.. math::

   B_\nu(T_{\mathrm{dust}}) \, \epsilon_\nu

with emissivity

.. math::

   \epsilon_\nu = \rho_{\mathrm{grain}} \, n_{\mathrm{grain}}
   \left( 0.01 \frac{1.3 \nu}{3 \times 10^{11}} \right)

The total background field is

.. math::

   B_{ij} = B_\nu(T_{\mathrm{cmb}}) + B_\nu(T_{\mathrm{dust}})\,\epsilon_\nu

Mean Radiation Field
--------------------

The angle-averaged radiation field entering the rate equations is

.. math::

   \langle J_{ij} \rangle =
   (1 - \beta_{ij}) S_{ij} + \beta_{ij} B_{ij}

This interpolates between optically thick trapping and optically thin escape.

Line Cooling Rate
-----------------

The cooling rate contributed by transition :math:`i \rightarrow j` is

.. math::

   \Lambda_{ij} =
   A_{ij} h \nu_{ij} \, n_i \,
   \beta_{ij}
   \frac{S_{ij} - B_{ij}}{S_{ij}}

The total cooling rate is the sum over all downward transitions:

.. math::

   \Lambda = \sum_{i>j} \Lambda_{ij}

Radiative Transition Matrix
---------------------------

The routine updates the radiative transition matrix:

.. math::

   R_{ij} = A_{ij} + B_{ij} \langle J_{ij} \rangle + C_{ij}

This matrix is used in the statistical equilibrium solver to update level
populations.

Memory Management
-----------------

Temporary arrays allocated within the routine:

* ``tau_ij`` – optical depth per ray
* ``field`` – mean radiation field matrix

These arrays are deallocated before returning.

Summary
-------

This routine provides a self-consistent LVG-based treatment of line transfer,
including dust and CMB backgrounds, ray-based anisotropy, and numerical
safeguards for maser action. It is suitable for both 3D and 1D geometries and
forms a core component of the 3D-PDR radiative cooling framework.
