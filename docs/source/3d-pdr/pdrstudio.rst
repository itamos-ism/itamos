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

The first run creates a self-contained Python virtual environment and installs all required dependencies; this step only happens once. Once installation completes, the app launches and you will be shown one or more local IP addresses (typically ``http://localhost:8501``). If your browser does not open automatically, click one of the displayed addresses.

PDR-studio locates the 3D-PDR installation automatically (it assumes ``PDR-studio/`` sits directly inside the 3D-PDR tree). If you have moved it elsewhere, set the ``PDR_ROOT`` environment variable to point at your 3D-PDR home directory.

Features
--------

The app is organised as a set of pages, available from the sidebar:

.. list-table::
   :header-rows: 1
   :widths: 15 45

   * - Page
     - What it does
   * - Home
     - Landing page and quick links.
   * - Code Configuration
     - Edit ``src/config.mk``, then **Save & Compile** directly from the browser.
   * - Model Parameters
     - Step-by-step wizard: ``params.dat`` → initial abundances / dust / gas → network selection → run the model → jump to Analysis I.
   * - Analysis I
     - 2×2 log-log plots of key quantities against A\ :sub:`V`, N\ :sub:`tot`, and n\ :sub:`H`, plus a tab for custom Python plots.
   * - Analysis II
     - Heating, cooling, emissivities, and level populations, plus a custom-plot tab.
   * - RT-tool
     - Runs :doc:`../rt-tool/RT-tool` directly; line tables, T\ :sub:`r` and τ build-up, and CO SLED plots.
   * - Chemical Analysis
     - Per-species formation/destruction pathways (reads ``.chemanalysis.fin``, so requires ``CHEMANALYSIS = 1`` — see :doc:`makefile`), hover-linked to depth, with an optional local-LLM natural-language summary.
   * - Network Builder
     - Builds a new chemical network from a master reaction file using an embedded, headless copy of MakeRates, and imports it directly into your 3D-PDR checkout.
   * - Report a bug
     - Bundles ``config.mk``, ``params.dat``, initial conditions, and outputs into a tarball you can send to the 3D-PDR team.

Adapting to your checkout
--------------------------

PDR-studio does not hard-code the list of compile flags. On every load it discovers them directly from your ``src/config.mk`` (the ``KEY = VALUE`` lines and their comment-header documentation) and cross-checks the allowed values against the ``ifeq (...)`` branches in ``src/makefile``. This means a checkout with newer or custom flags (for example after using the Network Builder, or after manually adding a flag) works out of the box: unrecognised flags still appear, grouped under an **Advanced** area, and are flagged as *newly imported*.

The same structure-preserving approach applies to ``params.dat``: each value is matched by its inline ``!label`` comment rather than its line number, so adding new parameter lines to a template does not break older wizard state, and saving only rewrites values in place — comments, section order, and any entries PDR-studio doesn't recognise are left untouched.

.. note::

   The **Chemical Analysis** page's optional pathway summaries use a local LLM (via `Ollama <https://ollama.com>`_ or ``llama-cpp-python``), run entirely offline. Without one configured, the deterministic pathway table still works; instructions for installing and configuring Ollama are provided within that page.
