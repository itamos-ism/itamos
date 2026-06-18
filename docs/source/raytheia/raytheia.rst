The RAYTHEIA Algorithm
=======================

**RAYTHEIA** is a standalone, three-dimensional, hybrid MPI/OpenMP reverse ray-tracing tool that computes the angle-averaged effective visual extinction, :math:`A_{V,\rm eff}`, at every cell of a Cartesian density grid. It runs independently of **3D-PDR** as a fast post-processing/pre-processing step on simulation outputs (e.g. SILCC, AREPO) before they are fed into **3D-PDR**.

The core ray-tracing engine (linear octree and adaptive DDA traversal) was written by Zhengping Zhu.

Algorithm
~~~~~~~~~

For every grid cell, **RAYTHEIA** casts a set of rays distributed isotropically using the `HEALPix <https://healpix.sourceforge.io/>`_ pixelization scheme (NESTED ordering), and integrates the local density along each ray to obtain a column density. The number of rays is set by the HEALPix resolution parameter ``level`` in ``params.dat``:

.. math::
   N_{\rm side} = 2^{\rm level}, \qquad N_{\rm rays} = 12\,N_{\rm side}^2

To avoid tracing rays cell-by-cell through the full resolution grid, each MPI rank compresses its local sub-domain into a **linear (Morton-code) octree**:

* The local grid is recursively subdivided in blocks of 8 octants (Z-order/Morton curve).
* A block is kept as a leaf (not split further) if it is a single cell, if it is effectively vacuum (peak density below a fixed threshold), or if its density is sufficiently uniform (max/min spread below 1%); otherwise it is split into 8 children at the next level.
* Each leaf stores a 64-bit Morton code, its octree level, and its (single-precision) average density — this is what makes the ``RAYTHEIA = 1`` mode of **3D-PDR** memory-efficient: large uniform or empty regions collapse into a single node instead of one entry per cell.

Rays are then marched through this compressed structure with an **adaptive DDA (Digital Differential Analyser)**: at each step the ray's current position is mapped to a local cell index, Morton-encoded, and located in the sorted leaf array via binary search; the ray then advances directly to the exit point of the (possibly large) leaf it landed in, rather than one grid cell at a time. Each rank only holds the octree for its own MPI sub-domain, so a ray crossing multiple ranks accumulates partial column densities from each rank's broadcasted tree.

The column density accumulated along ray :math:`i` (in :math:`\rm cm^{-2}`) is converted to extinction and the angle-averaged effective extinction is

.. math::
   A_{V,\rm eff} = -\frac{1}{\gamma}\ln\left\langle \exp(-\gamma\, A_{V,\rm fac}\, N_{{\rm H},i})\right\rangle_{i=1,N_{\rm rays}}

with :math:`\gamma = 3.02` and :math:`A_{V,\rm fac} = 6.29\times10^{-22}\,{\rm cm^{2}}` hard-coded in ``m_calc_columndens.f90``.

.. note::

   ``src/m_Ray_box.f90`` implements an earlier, alternative per-ray AMR approach (pointer-based octree, ``raytheia_amr``) by the same author. It is **not** included in the ``Makefile``'s source list and is not compiled or used — keep this in mind if browsing the source tree.

Compiling RAYTHEIA
~~~~~~~~~~~~~~~~~~~

Edit the top of the ``Makefile``:

.. code-block:: text

   CMP     = intel
   OPENMP  = 1
   HAMMER  = 1

- **CMP**: compiler suite.

  - ``intel`` — uses ``mpiifx`` (Intel MPI Fortran compiler).
  - ``gcc`` — uses ``mpif90`` (GNU MPI Fortran wrapper).

- **OPENMP**: ``1`` enables hybrid MPI+OpenMP parallelism over rays/cells (recommended); ``0`` disables it.
- **HAMMER**: selects the run mode at compile time.

  - ``1`` — builds the single-cell diagnostic mode (see :ref:`raytheia-modes` below).
  - ``0`` — builds the production mode that computes :math:`A_{V,\rm eff}` for the whole grid.

Then build with:

.. code-block:: bash

   make clean; make

.. important::

   The produced executable is named ``cd`` (set by ``exeName`` in the ``Makefile``). Run it as ``./cd`` — note that this name shadows the ``cd`` shell builtin if you ever try to source the Makefile directly.

The ``params.dat`` File
~~~~~~~~~~~~~~~~~~~~~~~~

``params.dat`` must be present in the run directory and follows a fixed line order, split into two blocks:

::

   ===========================|
   Grid parameters            |
   ===========================|
   125.0                       !xlx - domain size in x (pc)
   125.0                       !yly - domain size in y (pc)
   125.0                       !zlz - domain size in z (pc)
   128                         !nxc - global grid cells in x
   128                         !nyc - global grid cells in y
   128                         !nzc - global grid cells in z
   1                           !npx - MPI processes in x
   1                           !npy - MPI processes in y
   1                           !npz - MPI processes in z
   0                           !level - HEALPix refinement level
   ===========================|
   Input/Output - Densities   |
   ===========================|
   ics                         !indir - ICs directory
   silcc_128.dat                !input - ICs file (inside indir)
   sims                        !outdir - output directory

* **Entries 1-3** (``xlx``, ``yly``, ``zlz``): physical domain size in parsecs.
* **Entries 4-6** (``nxc``, ``nyc``, ``nzc``): number of grid cells along each axis. The cell size is ``dx = xlx/nxc`` (and similarly for ``dy``, ``dz``).
* **Entries 7-9** (``npx``, ``npy``, ``npz``): MPI Cartesian decomposition — number of processes along each axis.
* **Entry 10** (``level``): HEALPix refinement level; sets :math:`N_{\rm rays} = 12 \times 2^{2\,\rm level}`.
* **Entries 11-13**: ICs directory, ICs filename (read as ``indir/input``), and output directory.

.. note::

   The density input file (e.g. ``ics/silcc_128.dat``) is read in full by **every** MPI rank, which then keeps only the cells belonging to its own sub-domain. Each line is expected to contain ``x y z density`` for one cell, with the same fastest-varying axis convention (``z`` fastest) as **3D-PDR**'s ``XYZ = 0`` density format — see :doc:`ics`.

Parallel Decomposition Constraints
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``readparams`` enforces, and stops the run if violated:

* The number of MPI ranks launched must equal ``npx*npy*npz``.
* ``nxc``, ``nyc``, ``nzc`` must each be exactly divisible by ``npx``, ``npy``, ``npz`` respectively.
* The resulting **local** grid dimensions (``nxnp = nxc/npx``, and similarly for y, z) must each be a power of two — required by the linear-octree builder, which recursively bisects each local sub-domain down to single cells.

.. _raytheia-modes:

Running RAYTHEIA
~~~~~~~~~~~~~~~~~

.. code-block:: bash

   mpirun -np <npx*npy*npz> ./cd

Two run modes are selected at compile time via ``HAMMER`` in the ``Makefile``:

- **Production mode** (``HAMMER = 0``): computes :math:`A_{V,\rm eff}` for every cell of the grid (subroutine ``calc_columndens``) and writes ``Aveff.dat`` (see Outputs below). Can be run with any valid MPI decomposition.
- **HAMMER diagnostic mode** (``HAMMER = 1``): traces all :math:`N_{\rm rays}` from a single cell at the centre of the full domain (``nxc/2``, ``nyc/2``, ``nzc/2``) out to the domain boundary, and writes the raw per-ray column densities to ``cdMaps.dat``. This mode requires exactly one MPI rank (``nproc = 1``); running it with more ranks aborts with ``"Error: hammermap only for OpenMP"``.

Outputs
~~~~~~~

- ``Aveff.dat`` (production mode): plain ASCII file written by rank 0 in the **current working directory** (note: unlike ``cdMaps.dat``, this filename is hard-coded and does **not** use the ``outdir`` entry of ``params.dat``). One line per cell, looping ``i`` (x), then ``j`` (y), then ``k`` (z) fastest, with two columns:

  ::

     rho(i,j,k)   Aveff(i,j,k)

- ``cdMaps.dat`` (HAMMER mode): written to ``outdir/cdMaps.dat``. One line per HEALPix ray, in ray order (``ipix = 0 .. nrays-1``), with three columns:

  ::

     theta   phi   single_cd(ipix)

  where ``theta``, ``phi`` are the ray direction angles from ``pix2ang_nest``, and ``single_cd`` is the raw (un-converted-to-extinction) column density accumulated along that ray.

Implementation Notes
~~~~~~~~~~~~~~~~~~~~~

- ``healpix_types.f90``, ``bit_manipulation.f90``, and most of ``m_Healpix.f90`` are standard `HEALPix <https://healpix.sourceforge.io/>`_ Fortran library routines (Gorski, Hivon, Wandelt et al.), carried in the source tree for self-containment. Of these, **RAYTHEIA** itself only calls ``pix2ang_nest`` to generate the ray directions; the remaining routines are unused library boilerplate.
- Leaf densities in the linear octree (``LinearDensity``) are stored in single precision to reduce memory footprint, consistent with the goal of the memory-optimized (``RAYTHEIA = 1``) mode in **3D-PDR**.
