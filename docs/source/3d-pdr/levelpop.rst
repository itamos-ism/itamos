Level Population Solver
======================

Overview
--------

This module solves the **statistical equilibrium equations** for the
excitation of atomic or molecular species with an arbitrary number of
energy levels.  The solver is used after the chemistry and gas
temperature have converged (or during thermal balance iterations) and
provides non-LTE level populations for the calculation of cooling rates
and emergent line intensities.

The implementation is general and applies to all multi-level species
treated in the code (e.g. CO, CI, CII, OI, SI).

The solution assumes steady state (time-independent) excitation and
includes both **collisional** and **radiative** processes.

Statistical Equilibrium Equations
---------------------------------

For a species with :math:`N_\mathrm{lev}` energy levels, the population
:math:`n_i` of level :math:`i` is obtained by solving the system:

.. math::

   \sum_{j \ne i} n_j P_{ji}
   \;-\;
   n_i \sum_{j \ne i} P_{ij}
   \;=\; 0
   \qquad \forall i

where:

- :math:`P_{ij}` is the total transition rate from level :math:`i` to
  level :math:`j`,
- radiative (spontaneous, stimulated) and collisional processes are
  included in :math:`P_{ij}`.

Because these equations are linearly dependent, they must be supplemented
by the **particle conservation constraint**:

.. math::

   \sum_{i=1}^{N_\mathrm{lev}} n_i = n_\mathrm{species}

where :math:`n_\mathrm{species}` is the total number density of the
species.

Transition Rate Matrix
----------------------

The solver receives as input the **transition matrix**
:math:`P_{ij}`, stored as:

- ``TRANSITION(i,j)`` = rate from level :math:`i \rightarrow j`

The statistical equilibrium equations are written in matrix form:

.. math::

   \mathbf{A} \cdot \mathbf{n} = \mathbf{b}

with matrix elements:

.. math::

   A_{ii} = -\sum_{j \ne i} P_{ij}

.. math::

   A_{ij} = P_{ji} \qquad (i \ne j)

and right-hand side:

.. math::

   \mathbf{b} = \mathbf{0}

except for the normalization equation (see below).

Normalization Constraint
------------------------

To remove the singularity of the rate matrix, the final row of the system
is replaced by the population conservation condition:

.. math::

   \sum_i n_i = n_\mathrm{species}

Numerically, this is implemented by replacing the final equation with:

.. math::

   \sum_i \epsilon \, n_i = \epsilon \, n_\mathrm{species}

where :math:`\epsilon = 10^{-8}` is a small scaling factor used to:

- avoid division by zero,
- preserve numerical conditioning,
- leave the physical solution unchanged.


Numerical Solution
------------------

The resulting linear system is solved using a **Gauss–Jordan elimination**
algorithm with full pivoting, adapted from *Numerical Recipes*.

Key properties of the solver:

- Full row and column pivoting for numerical stability
- Detection of singular matrices
- Direct solution without iteration
- Suitable for small dense matrices (:math:`N_\mathrm{lev} \lesssim 100`)

On output, the solution vector contains the level populations
:math:`n_i`.

Negative Population Handling
----------------------------

Due to finite numerical precision, very small negative level populations
can occasionally arise near zero. These values are non-physical and are
removed by enforcing:

.. math::

   n_i \ge 0 \qquad \forall i

This correction does not affect the solution when populations are well
converged.

Subroutine: ``solvlevpop``
--------------------------

**Purpose**
  Solve the statistical equilibrium equations for level populations.

**Inputs**

- ``NLEV``  
  Number of energy levels.

- ``density``  
  Total number density of the species.

- ``TRANSITION(NLEV,NLEV)``  
  Total transition rates between levels.

**Outputs**

- ``SOLUTION(NLEV)``  
  Level populations :math:`n_i`.

Subroutine: ``GAUSS_JORDAN``
----------------------------

This routine performs the Gauss–Jordan elimination used to solve the
linear system. The matrix is inverted in place, and the right-hand side
vector is replaced by the solution.

The solver stops if a singular matrix is detected, which generally
indicates an unphysical state (e.g. vanishing density or temperature).

Debugging Variant: ``GAUSS_JORDAN_writes``
------------------------------------------

A diagnostic version of the solver is provided for debugging numerical
instabilities. In addition to solving the system, it:

- Prints the rate matrix and right-hand side
- Outputs intermediate elimination steps
- Reports the gas temperature and grid point on failure

This routine is intended for debugging purposes only and is not used in
production runs.

Physical Assumptions
--------------------

The level population solver assumes:

- Statistical equilibrium (time-independent excitation)
- Fixed species density
- Linear radiative and collisional rates
- No explicit maser saturation
- Radiative trapping already included in the transition rates
  (e.g. via LVG escape probabilities)

Role in the PDR Iteration Scheme
--------------------------------

The solver is called as part of the iterative PDR solution loop:

1. Chemistry update
2. Temperature estimate
3. Rate coefficient calculation
4. **Level population solution (this module)**
5. Cooling rate evaluation
6. Thermal balance iteration

Accurate convergence of the level populations is essential for reliable
cooling rates and thermal balance.
