.. _species:

The species_[network].d
=======================

To modify the initial elemental abundances in your PDR model, edit the ``species_NETWORK.d`` file located in the ``chemfiles/`` directory. The specific file name depends on the chemical network you are using.

.. important::
   No re-compilation of 3D-PDR is required when only modifying elemental abundances. Simply edit the appropriate species file and run your simulation.

The ``species_NETWORK.d`` file contains the initial abundance values for all chemical elements included in the network. Modify these values according to your desired metallicity or abundance pattern.

Example modification for the REDUCED network
--------------------------------------------

If using the ``species_reduced.d`` network, you would edit ``chemfiles/species_reduced.d`` and adjust the abundance values for elements such as hydrogen, carbon, oxygen, etc., to reflect your desired chemical composition.

When editing ``species_reduced.d``, pay special attention and change only the following species while leaving all the rest equal to ``0.0``:

- **C+ (entry 11)**: Carbon abundance should be specified here rather than in neutral C (entry 25) to allow the code to start the chemical network from the ionized phase of carbon
- **O (entry 30)**: Oxygen abundance
- **H (entry 32)**: Atomic hydrogen abundance
- **H2 (entry 31)**: Molecular hydrogen abundance
- **He (entry 26)**: Helium abundance
- **Mg+ (entry 10)**: Ionized magnesium abundance (if desired)

Locate the relevant entries in your species file and modify the abundance values (third column):

.. code-block:: text

   11,C+,1.40e-04,12.0    # Carbon abundance
   30,O,3.00e-04,16.0     # Oxygen abundance
   32,H,4.00e-01,1.0      # Atomic hydrogen
   31,H2,3.00e-01,2.0     # Molecular hydrogen
   26,He,1.00e-01,4.0     # Helium abundance
   10,Mg+,0.0,24.0        # Ionized magnesium

.. note::
   - The total hydrogen abundance must satisfy: **HI + 2×H₂ = 1** at all times
   - The default values (H = 0.4, H₂ = 0.3) satisfy this constraint: 0.4 + 2×0.3 = 1
   - Electron abundance (e-, entry 33) is automatically calculated by 3D-PDR from the input ionized elements and should not be modified manually, unless specialized runs need to be performed
   - For specialized runs, you may adjust the HI/H₂ ratio, but always maintain the total hydrogen constraint

After saving your changes to the species file, 3D-PDR will automatically use the updated abundances in your next simulation without requiring recompilation.

Example modification for the MEDIUM network
-------------------------------------------

The ``MEDIUM`` network (``chemfiles/species_medium.d``, 109 species) includes sulphur and magnesium in addition to H, He, C, N and O. Its non-zero initial abundances, which should be the only entries you change, are:

.. code-block:: text

   66,C+,1.40E-04,12.0    # Carbon abundance (start from the ionized phase)
   105,O,3.00E-04,16.0    # Oxygen abundance
   104,N,5.65E-05,14.0    # Nitrogen abundance
   102,S,3.51E-06,32.0    # Sulphur abundance
   59,Mg+,2.70E-06,24.0   # Ionized magnesium (gas-phase metal)
   101,He,1.00E-01,4.0    # Helium abundance
   108,H,4.00E-01,1.0     # Atomic hydrogen
   107,H2,3.00E-01,2.0    # Molecular hydrogen

All other entries, including the electrons (``e-``, entry 109), should be left at ``0.0``.

.. note::
   The gas-phase metal abundance (here Mg\ :sup:`+`) controls the electron fraction in the shielded gas, where C\ :sup:`+` has recombined. It therefore directly affects the abundances of molecular ions such as HCO\ :sup:`+`, which are destroyed mainly by dissociative recombination. The default value, 2.7×10\ :sup:`-6`, corresponds to diffuse-cloud depletion; the previous version of the ``MEDIUM`` network used 2.7×10\ :sup:`-7` and had no sulphur. State the adopted metal and sulphur abundances when reporting results, and test their effect if molecular-ion abundances or line ratios such as HCO\ :sup:`+`/HCN are important for your application.




