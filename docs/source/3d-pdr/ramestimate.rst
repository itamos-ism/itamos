.. _ram-estimate:

The RAM estimator
==================

Large 3D models can require a substantial amount of memory, and finding this out only after a job has been queued (or has crashed out-of-memory) is costly. ``ram_estimate.py``, located in the repository root next to ``params.dat``, estimates the peak RAM a 3D model will need **before you run it**, so a job can be sized correctly for a cluster or workstation ahead of time.

The script reads your ``params.dat`` (coolant list, HEALPix level) and ``src/config.mk`` (``RAYTHEIA`` mode, ``NETWORK``, ``CHEMANALYSIS``, and the other compile flags that affect the per-cell memory layout), fetches ``NLEV``/``NTEMP`` from each coolant's LAMDA file, and reports a per-component RAM breakdown for a grid of the resolution you specify. It is a plain Python 3 script with no dependencies beyond the standard library (``gfortran`` is used opportunistically, to measure exact Fortran struct sizes — see ``--no-probe`` below).

Usage
-----

.. code-block:: console

   $ python3 ram_estimate.py --res 256 --raytheia 1 --lmax 13

``params.dat`` in the current directory is read by default; pass ``--params /path/to/params.dat`` to point at a different model. ``--res N`` (required) sets the grid resolution (:math:`N^3` cells). ``--raytheia``, ``--lmax``, and the chemistry network are normally read automatically from ``src/config.mk``, but can all be overridden on the command line to explore "what if" scenarios — for example, checking the RAM at a higher HEALPix level, or with :ref:`levmax-flag` capping the number of coolant energy levels, before committing to a run.

Useful options:

.. list-table::
   :header-rows: 1
   :widths: 15 40

   * - Option
     - Purpose
   * - ``--res N``
     - Grid resolution, :math:`N^3` cells (required).
   * - ``--params FILE``
     - Path to ``params.dat`` (default: ``params.dat``).
   * - ``--healpix L``
     - HEALPix level override (default: read from ``params.dat``).
   * - ``--lmax N``
     - Cap energy levels per coolant, matching the runtime ``-lmax=N`` flag (default: no cap). See :ref:`levmax-flag`.
   * - ``--raytheia {0,1,2}``
     - Ray-tracing mode matching ``config.mk RAYTHEIA=`` (default: read from ``config.mk``).
   * - ``--threads T``
     - OpenMP thread count, for the peak temporary (``evalpop``) estimate.
   * - ``--chemanalysis {0,1}``
     - Override the ``CHEMANALYSIS`` flag (default: read from ``config.mk``).
   * - ``--srcdir DIR``
     - Path to ``src/``, used to locate ``config.mk`` and to measure exact struct sizes (default: ``<params dir>/src``).
   * - ``--no-probe``
     - Skip compiling the struct-size probe with ``gfortran``; use a rougher fallback instead.

Run ``python3 ram_estimate.py --help`` for the full list of options.

What it reports
----------------

For the requested resolution, the tool prints:

- A per-coolant table of energy levels used (after any ``-lmax`` cap).
- A persistent-RAM breakdown across every component that scales with grid size: coolant level-population arrays, ray-tracing path arrays, chemical abundances, the Fortran struct and pointer-descriptor overhead (measured directly via ``gfortran`` when available), static LAMDA data, and — when relevant — the ``CHEMANALYSIS`` ``temp_rate(nreac,pdr_ptot)`` array, which is disproportionately large and is explicitly flagged as "not recommended for 3D models" in ``config.mk`` (see :doc:`makefile`).
- The peak *temporary* RAM used during cooling-function evaluation (``evalpop``, scaled by ``--threads``).
- A HEALPix-level comparison table, since ray-tracing memory scales as :math:`12 \times 4^L` per cell and is usually the first thing worth reconsidering if a model does not fit in the available memory.

Notes on accuracy
-----------------

The estimate is derived directly from the Fortran source (``initialization.F90``, ``allocations.F90``, ``modules.F90``, ``3DPDR.F90``), not from a fixed formula, so it stays correct as the code evolves:

- Per-cell struct overhead (the array descriptors inside the internal grid-point and coolant data structures) is measured by compiling a small probe against your actual ``src/modules.F90`` with the same preprocessor flags as your build, rather than guessed.
- The script inspects ``3DPDR.F90`` to check whether the per-cell cooling/heating arrays are allocated efficiently, and adjusts the estimate accordingly (overridable with ``--coolheat-fixed``).

.. note::

   Expect the reported total to be within a few percent of observed usage. The remainder is typically OS/allocator overhead (heap alignment and bookkeeping across the many small per-cell allocations) that isn't practical to model exactly, plus MPI buffers, Fortran runtime overhead, and CVODE solver workspace — none of which are included in the estimate.
