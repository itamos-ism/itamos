.. _pdr-studio:

PDR-studio
==========

**PDR-studio** is a browser-based control panel for **3D-PDR**: it lets you configure, compile, run, and analyse PDR models from a local web app, without needing to touch the terminal or edit ``config.mk``/``params.dat`` by hand.

Quick start
-----------

Navigate to the ``PDR-studio/`` directory inside your cloned ``3D-PDR/`` repository and run:

.. code-block:: console

   $ cd PDR-studio
   $ chmod 755 run.sh
   $ ./run.sh

``run.sh`` creates a self-contained Python virtual environment (``.venv``) and installs all required dependencies on first run only. Once installation completes, the app launches at ``http://localhost:8501``. If your browser does not open automatically, click the displayed address.

PDR-studio finds the 3D-PDR installation automatically — it assumes ``PDR-studio/`` sits directly inside the 3D-PDR tree (its own parent directory). If you have moved it elsewhere, set the ``PDR_ROOT`` environment variable to point at your 3D-PDR home directory.

.. note::

   To make PDR-studio reachable from other machines on your local network, use ``./run-lan.sh`` instead, which prints the address to share (e.g. ``http://<your-ip>:8501``).

   .. important::

      Anyone who can reach that address gets the full app, which can compile and run 3D-PDR and execute arbitrary Python typed into the **Custom plot** tabs *on this machine*. Only use ``run-lan.sh`` on a trusted internal network.

Features
--------

The app is organised as a set of pages, available from the sidebar:

.. list-table::
   :header-rows: 1
   :widths: 15 45

   * - Page
     - What it does
   * - Home
     - Landing page showing whether 3D-PDR is detected/built, the current ``config.mk`` build (dimensions and network), and quick links to every feature.
   * - Tutorial
     - A hands-on, in-app walkthrough from launching PDR-studio to a finished, analysed 3D-PDR model, following the sidebar top to bottom.
   * - Code Configuration
     - Edit ``src/config.mk``, then **Save & Compile** directly from the browser; **Next** continues to Model Parameters.
   * - Model Parameters
     - Step-by-step wizard: ``params.dat`` → initial abundances → chemical network → run the model → jump into analysis.
   * - Line emission
     - Runs :doc:`../rt-tool/RT-tool` on the finished model and shows, per coolant: line tables (line, frequency, T\ :sub:`ex`, τ, T\ :sub:`r`) at a chosen depth; how T\ :sub:`r` and τ build up with column density; and the CO spectral-line energy distribution (SLED). ``paramsRT.dat`` is filled in automatically from ``params.dat``.
   * - Abundance Profiles
     - A 2×2 log-log figure (T\ :sub:`gas`, HI/H\ :sub:`2`, C\ :sup:`+`/C/CO, and a user-selected species) against A\ :sub:`V` or n\ :sub:`H`, with the HI→H\ :sub:`2` transition marked; plus a **Custom plot** tab — a matplotlib scratchpad pre-loaded with the model's arrays.
   * - Heating & Cooling
     - A 2×2 log-log figure of heating, cooling, emissivities, and level populations against A\ :sub:`V` or n\ :sub:`H`, plus a custom-plot tab.
   * - Chemical Analysis
     - Per-species formation/destruction pathways (reads ``.chemanalysis.fin``, so requires ``CHEMANALYSIS = 1`` — see :doc:`makefile`), hover-linked to depth, with an optional local-LLM natural-language summary.
   * - Network Builder
     - Builds a new chemical network from a master reaction file using an embedded, headless copy of MakeRates, and imports it directly into your 3D-PDR checkout.
   * - Report a bug
     - Bundles the current code configuration, parameters, chemistry files, initial conditions, and model outputs into a single tarball to send to the 3D-PDR team.

Adapting to your checkout
--------------------------

PDR-studio does not hard-code the list of compile flags. On every load it discovers them directly from your ``src/config.mk`` (the ``KEY = VALUE`` lines and their comment-header documentation) and cross-checks the allowed values against the ``ifeq (...)`` branches in ``src/makefile``. This means a checkout with newer or custom flags (for example after using the Network Builder, or after manually adding a flag) works out of the box: unrecognised flags still appear, grouped under an **Advanced** area, and are flagged as *newly imported*.

The same structure-preserving approach applies to ``params.dat``: each value is matched by its inline ``!label`` comment rather than its line number, so adding new parameter lines to a template does not break older wizard state, and saving only rewrites values in place — comments, section order, and any entries PDR-studio doesn't recognise are left untouched.

Network Builder
----------------

The **Network Builder** page creates a new chemical network with an embedded, headless copy of MakeRates and imports it into the 3D-PDR tree:

1. Pick the elements to include (``e-``, ``H``, ``H2``, ``He``, ``C+``, ``O`` are always in; add any of ``Mg+, N, S, Fe, Cl, P, Si, Na, F``), or import your own species file. PDR-studio keeps the species made only of those elements.
2. MakeRates builds ``species_<NAME>.d``, ``rates_<NAME>.d``, and ``odes_<NAME>.c`` (all initial abundances set to zero).
3. **Import** copies species/rates into ``chemfiles/``, the ODEs into ``src/``, and wires ``NETWORK = <NAME>`` into ``config.mk``, the ``makefile``, and the relevant ``#ifdef`` blocks. Chemical heating needs no edit thanks to ``AUTOCHEMHEAT``. Recompile to use the new network — its elements then appear on the Model Parameters page.

.. note::

   The optional Chemical Analysis pathway summaries use a local LLM, run entirely offline, so no model data ever leaves your machine:

   - **Ollama** (recommended): install it and run ``ollama pull llama3.2`` (or any small instruct model, e.g. ``qwen2.5``, ``phi3``); PDR-studio detects it automatically on ``http://localhost:11434``.
   - **llama.cpp**: ``pip install llama-cpp-python`` into the app's virtual environment and point ``PDRSTUDIO_LLM_GGUF`` at a local ``.gguf`` model file.

   Without either configured, the deterministic pathway table still works — only the natural-language summary is unavailable.
