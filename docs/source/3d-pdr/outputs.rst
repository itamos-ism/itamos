Understanding the outputs
=========================

Output files are written to the ``sims/`` directory (unless specified otherwise) after the code has fully converged. The code generates multiple output files with different suffixes:

- **.pdr.fin**: Species abundances
- **.line.fin**: Line emission data
- **.cool.fin**: Cooling functions
- **.heat.fin**: Heating functions
- **.COOL.spop.fin**: Level populations for each coolant (where ``COOL`` is the coolant name)
- **.species**: Species index/name key used to interpret the abundance columns of ``.pdr.fin``
- **.params**: Global model parameters (radiation field, cosmic-ray rate, metallicity, turbulent velocity, grid size)
- **.RTspop.fin**: Per-cell level populations for every coolant in a single file, used as input to :doc:`../rt-tool/RT-tool`

.. note::
   Outputs are in ASCII format and can be plotted using standard tools like GNUPLOT or PYTHON.

One-dimensional Outputs
-----------------------

OUTPUT.pdr.fin
~~~~~~~~~~~~~~

+--------+---------------------+-----------------------------------+
| Column | Value               | Comments                          |
+========+=====================+===================================+
| 1      | ID                  | Cell identity (integer)           |
+--------+---------------------+-----------------------------------+
| 2      | x                   | Position (pc)                     |
+--------+---------------------+-----------------------------------+
| 3      | Av                  | Visual extinction (mag)           |
+--------+---------------------+-----------------------------------+
| 4      | Tgas                | Gas temperature (K)               |
+--------+---------------------+-----------------------------------+
| 5      | Tdust               | Dust temperature (K)              |
+--------+---------------------+-----------------------------------+
| 6      | etype               | Type of cell (integer)            |
+--------+---------------------+-----------------------------------+
| 7      | nH                  | Number density (cm⁻³)             |
+--------+---------------------+-----------------------------------+
| 8      | UV                  | FUV intensity (Draine units)      |
+--------+---------------------+-----------------------------------+
| 9+     | ...                 | Species abundances                |
+--------+---------------------+-----------------------------------+

Species abundances follow the sequence in ``species_NETWORK.d``.

OUTPUT.cool.fin
~~~~~~~~~~~~~~~

+--------+---------------------+-----------------------------------+
| Column | Value               | Comments                          |
+========+=====================+===================================+
| 1      | ID                  | Cell identity (integer)           |
+--------+---------------------+-----------------------------------+
| 2      | x                   | Position (pc)                     |
+--------+---------------------+-----------------------------------+
| 3      | Av                  | Visual extinction (mag)           |
+--------+---------------------+-----------------------------------+
| 4      | CO                  | CO cooling function               |
+--------+---------------------+-----------------------------------+
| 5      | C+                  | C+ cooling function               |
+--------+---------------------+-----------------------------------+
| 6      | C                   | C cooling function                |
+--------+---------------------+-----------------------------------+
| 7      | O                   | O cooling function                |
+--------+---------------------+-----------------------------------+
| 8+     | ...                 | Additional coolant cooling        |
+--------+---------------------+-----------------------------------+
| Last   | Total               | Total cooling (sum of all)        |
+--------+---------------------+-----------------------------------+

Coolant sequence matches the order in ``params.dat``. Without additional coolants, total cooling is column 8.

OUTPUT.heat.fin
~~~~~~~~~~~~~~~

+--------+---------------------+-----------------------------------+
| Column | Value               | Comments                          |
+========+=====================+===================================+
| 1      | ID                  | Cell identity (integer)           |
+--------+---------------------+-----------------------------------+
| 2      | x                   | Position (pc)                     |
+--------+---------------------+-----------------------------------+
| 3      | Av                  | Visual extinction (mag)           |
+--------+---------------------+-----------------------------------+
| 4      | HR01                | Not contributing                  |
+--------+---------------------+-----------------------------------+
| 5      | HR02                | Photoelectric heating             |
+--------+---------------------+-----------------------------------+
| 6      | HR03                | Not contributing                  |
+--------+---------------------+-----------------------------------+
| 7      | HR04                | Carbon ionization heating         |
+--------+---------------------+-----------------------------------+
| 8      | HR05                | H₂ formation heating              |
+--------+---------------------+-----------------------------------+
| 9      | HR06                | H₂ photodissociation heating      |
+--------+---------------------+-----------------------------------+
| 10     | HR07                | FUV pumping heating               |
+--------+---------------------+-----------------------------------+
| 11     | HR08                | Cosmic-ray heating                |
+--------+---------------------+-----------------------------------+
| 12     | HR09                | Turbulent heating                 |
+--------+---------------------+-----------------------------------+
| 13     | HR10                | Chemical heating                  |
+--------+---------------------+-----------------------------------+
| 14     | HR11                | Gas-grain heating                 |
+--------+---------------------+-----------------------------------+
| 15     | HR12                | Total heating (sum of all)        |
+--------+---------------------+-----------------------------------+

Three-dimensional Outputs
-------------------------

3D-OUTPUT.pdr.fin
~~~~~~~~~~~~~~~~~

+------------------+---------------------+-----------------------------------+
| Column           | Value               | Comments                          |
+==================+=====================+===================================+
| 1                | ID                  | Cell identity (integer)           |
+------------------+---------------------+-----------------------------------+
| 2                | x                   | x position (pc)                   |
+------------------+---------------------+-----------------------------------+
| 3                | y                   | y position (pc)                   |
+------------------+---------------------+-----------------------------------+
| 4                | z                   | z position (pc)                   |
+------------------+---------------------+-----------------------------------+
| 5                | Tgas                | Gas temperature (K)               |
+------------------+---------------------+-----------------------------------+
| 6                | Tdust               | Dust temperature (K)              |
+------------------+---------------------+-----------------------------------+
| 7                | etype               | Type of cell (integer)            |
+------------------+---------------------+-----------------------------------+
| 8                | nH                  | Number density (cm⁻³)             |
+------------------+---------------------+-----------------------------------+
| 9                | UV                  | FUV intensity (Draine units)      |
+------------------+---------------------+-----------------------------------+
| 10-10+species    | ...                 | Species abundances                |
+------------------+---------------------+-----------------------------------+
| 11+species-...   | Av                  | Visual extinction per HEALPix ray |
+------------------+---------------------+-----------------------------------+

Additional Outputs (1D and 3D)
-------------------------------

The three files below are written for every model, regardless of ``DIMENSIONS``.

OUTPUT.species
~~~~~~~~~~~~~~

A plain index/name key for the chemical network used, one species per line:

.. code-block:: text

    1 Mg
    2 He+
    3 HNCO
    ...

The line number (first column) matches the species ordering in ``species_NETWORK.d`` (see :doc:`species`), which is the same order the abundance columns appear in ``OUTPUT.pdr.fin`` (column 9 onward for 1D, column 10 onward for 3D). Use this file to map an abundance column back to its species name without having to count columns manually.

OUTPUT.params
~~~~~~~~~~~~~

A compact record of the global parameters the model was run with, written as three lines:

.. list-table::
   :header-rows: 1
   :widths: 8 40

   * - Line
     - Contents
   * - 1
     - FUV field, CR ionization rate (or the CR attenuation model choice if ``CRATTENUATION=1``), metallicity, turbulent velocity
   * - 2
     - Grid dimensions ``nxc nyc nzc`` (number of cells along x, y, z)
   * - 3
     - Physical box size ``xlx yly zlz`` (pc)

.. note::

   The exact contents of line 1 depend on the ``CRATTENUATION`` flag (see :doc:`makefile`):

   - ``CRATTENUATION = 0``: FUV field, :math:`\zeta` (CR ionization rate), metallicity, turbulent velocity.
   - ``CRATTENUATION = 1``: FUV field, the CR attenuation model choice (``L``/``H``/``U``), metallicity, turbulent velocity.

This file is primarily consumed by downstream analysis and plotting tools (including PDR-studio, see :doc:`pdrstudio`) that need the model's global setup without re-parsing ``params.dat``.

OUTPUT.RTspop.fin
~~~~~~~~~~~~~~~~~

A single file collecting the level populations of **every** coolant for **every** grid cell, intended as the direct input to :doc:`../rt-tool/RT-tool` for computing synthetic radiation/excitation temperatures. Its structure is:

.. code-block:: text

    NETWORK
    LMAX=N
    coolant_file_1.dat
    coolant_file_2.dat
    ...
    ENDCOOLFILES
    <level populations for cell 1, all coolants concatenated>
    <level populations for cell 2, all coolants concatenated>
    ...

- **Line 1**: the compiled chemical network name (``REDUCED``, ``MEDIUM``, or ``FULL``).
- **Line 2**: ``LMAX=N``, the value of ``lmax_levels`` used at runtime — ``0`` if the :ref:`levmax-flag` cap was not applied (all levels from each LAMDA file were used), or the ``-lmax=N`` value otherwise.
- **Coolant file list**: one line per coolant, terminated by the ``ENDCOOLFILES`` marker.
- **Data rows**: one row per grid cell; for each coolant, the populations of its levels (``1`` to ``cnlev``, capped by ``LMAX`` if active) are written consecutively on that row, in the same coolant order as the file list above.

.. important::

   Because the number of levels per coolant can be truncated by ``LMAX``, downstream tools must read the ``LMAX=N`` line (and the coolant list) to know how many columns to expect for each coolant — the row width is not fixed across differently-configured runs.
