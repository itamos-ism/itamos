Gas Temperature Update
======================

Overview
--------

The ``changetemperature`` subroutine updates the **gas temperature** in each
PDR grid point once the **level populations have converged**.  
It enforces **thermal balance** by iteratively adjusting the gas temperature
until the total heating and cooling rates are equal within a prescribed tolerance.

The routine operates point-by-point over the full PDR domain and is
parallelised using OpenMP.

Its main responsibilities are:

* Compute reaction rates at the current gas temperature.
* Compute all relevant heating processes.
* Compare total heating and total cooling rates.
* Adjust the gas temperature using a hybrid strategy:
  
  * Exponential temperature stepping (±30%)
  * Binary chop (bisection) once a valid temperature bracket is found

* Detect convergence and enforce physical temperature bounds.

Thermal Balance Condition
-------------------------

Thermal equilibrium is defined as:

.. math::

   \Gamma_{\mathrm{tot}} = \Lambda_{\mathrm{tot}}

where:

* :math:`\Gamma_{\mathrm{tot}}` is the total heating rate
* :math:`\Lambda_{\mathrm{tot}}` is the total cooling rate

The imbalance is quantified by:

Mean Imbalance
^^^^^^^^^^^^^^

.. math::

   F_{\mathrm{mean}} = \Gamma_{\mathrm{tot}} - \Lambda_{\mathrm{tot}}

Relative Imbalance
^^^^^^^^^^^^^^^^^^

.. math::

   F_{\mathrm{ratio}} =
   \frac{2\,|F_{\mathrm{mean}}|}
        {|\Gamma_{\mathrm{tot}} + \Lambda_{\mathrm{tot}}|}

Thermal balance is considered achieved when:

.. math::

   F_{\mathrm{ratio}} \le F_{\mathrm{crit}}

Heating and Cooling
-------------------

Reaction Rates
^^^^^^^^^^^^^^

For each PDR point, reaction rates are computed using the current gas and dust
temperatures:

* Gas temperature :math:`T_{\mathrm{gas}}`
* Dust temperature :math:`T_{\mathrm{dust}}`
* Local radiation field and column densities along all rays

These rates are passed to the heating routines.

Heating Processes
^^^^^^^^^^^^^^^^^

The routine computes up to **12 distinct heating channels**, including (but not
limited to):

* Photoelectric heating
* Cosmic-ray heating
* Chemical heating
* H₂ formation heating
* Gas–dust thermal coupling

The total heating rate is stored as:

.. math::

   \Gamma_{\mathrm{tot}} = \texttt{heating(12)}

Cooling Processes
^^^^^^^^^^^^^^^^^

Cooling rates are assumed to have already been computed (e.g. via LVG line
cooling) and stored as:

.. math::

   \Lambda_{\mathrm{tot}} = \sum_i \Lambda_i

Temperature Update Strategy
---------------------------

The temperature update proceeds **only after level populations have converged**
(``level_conv = .true.``).

Initial Bracketing Phase
^^^^^^^^^^^^^^^^^^^^^^^^

During the first thermal iteration:

* If :math:`F_{\mathrm{mean}} > 0` (net heating):
  
  .. math::

     T_{\mathrm{new}} = 1.3\,T_{\mathrm{old}}

  and the lower bound is updated:

  .. math::

     T_{\mathrm{low}} = T_{\mathrm{old}}

* If :math:`F_{\mathrm{mean}} < 0` (net cooling):

  .. math::

     T_{\mathrm{new}} = 0.7\,T_{\mathrm{old}}

  and the upper bound is updated:

  .. math::

     T_{\mathrm{high}} = T_{\mathrm{old}}

This phase searches for a temperature interval that brackets the thermal
equilibrium solution.

Binary Chop Phase
^^^^^^^^^^^^^^^^^

Once heating and cooling change sign between iterations, the algorithm switches
to **binary chopping** (bisection):

* If heating dominates:

  .. math::

     T_{\mathrm{low}} = T_{\mathrm{current}}, \quad
     T_{\mathrm{new}} = \frac{T_{\mathrm{current}} + T_{\mathrm{high}}}{2}

* If cooling dominates:

  .. math::

     T_{\mathrm{high}} = T_{\mathrm{current}}, \quad
     T_{\mathrm{new}} = \frac{T_{\mathrm{current}} + T_{\mathrm{low}}}{2}

From this point onward, only bisection is used.

Convergence Criteria
--------------------

A PDR point is marked as **fully converged** if **any** of the following are met:

1. Thermal balance criterion:

   .. math::

      F_{\mathrm{ratio}} \le F_{\mathrm{crit}}

2. Temperature stagnation without balance:

   .. math::

      |T_{\mathrm{new}} - T_{\mathrm{old}}| \le T_{\mathrm{diff}}
      \quad \text{and} \quad
      F_{\mathrm{ratio}} > F_{\mathrm{crit}}

3. Physical bounds are reached consistently:

   * :math:`T \le T_{\min}` while cooling dominates
   * :math:`T \ge T_{\max}` while heating dominates

Temperature Bounds
------------------

The gas temperature is strictly constrained to lie within:

.. math::

   T_{\min} \le T_{\mathrm{gas}} \le T_{\max}

If convergence is reached at a boundary:

* The temperature is clamped.
* Further iteration may be forced if level populations must be recomputed.

Parallelisation
---------------

The routine is OpenMP-parallelised over PDR points:

* Each PDR point is independent.
* Shared arrays are read-only or explicitly indexed by point.
* Private variables include:
  
  * Reaction rates
  * Heating arrays
  * Temporary temperature variables

This ensures scalability for large 3D models.

Summary
-------

The ``changetemperature`` routine provides a robust and physically controlled
method to enforce **local thermal balance** in PDR models after level population
convergence. Its hybrid stepping–bisection strategy ensures:

* Rapid initial exploration of temperature space
* Guaranteed convergence once a valid bracket is found
* Stability in extreme heating or cooling regimes
* Strict adherence to physical temperature limits

This routine forms the thermal closure of the coupled
**chemistry–radiative transfer–thermal balance** problem in 3D-PDR.
