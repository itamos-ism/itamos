.. _sec-3dpdr:

The 3D-PDR code
===============

3D-PDR (`Bisbas et al. 2012 <https://ui.adsabs.harvard.edu/abs/2012MNRAS.427.2100B/abstract>`_) is a state-of-the-art numerical code designed to model photodissociation regions (PDRs) in arbitrary three-dimensional geometries. It self-consistently solves the chemistry, thermal balance, and radiative transfer within the interstellar medium, accounting for the interaction between far-ultraviolet (FUV) radiation and gas and dust. The code employs a ray-tracing scheme based on the HEALPix algorithm to evaluate the attenuation of the radiation field in all directions from each grid cell, allowing realistic treatment of complex density structures. It includes a comprehensive chemical network, heating and cooling processes, and line radiative transfer for key atomic and molecular species. 

The upgraded 3D-PDR uses the ray-tracing scheme RAYTHEIA (Zhu et al., *submitted*), a highly parallelized algorithm that achieves significant speed-up allowing to model high-resolution PDRs in full three-dimensions. In particular, the MPI-parallelized version enables large-scale simulations on distributed-memory systems, making 3D-PDR a powerful tool for investigating PDRs in both Galactic and extragalactic environments.

You can download the 3D-PDR code from this `Github link <https://github.com/itamos-ism/3D-PDR>`_.

Papers using 3D-PDR
-------------------

Since its development, 3D-PDR and its coupled tools for synthetic observations have been used to demonstrate the power of full three-dimensional PDR modelling for both interpreting observations and for probing fundamental ISM physics. Below is a growing list of papers that used 3D-PDR in their analysis (in reversed chronological order):


* `The SOMA Atomic Outflow Survey. I. An Atomic OI and Highly Ionized OIII Outflow from Massive Protostar G11.94-00.62` (`Oakey et al., 2025 <https://ui.adsabs.harvard.edu/abs/2025arXiv250920551O/abstract>`_)
* `NeuralPDR: neural differential equations as surrogate models for photodissociation regions` (`Vermarien et al., 2025 <https://ui.adsabs.harvard.edu/abs/2025MLS%26T...6b5069V/abstract>`_)
* `Metallicity dependence of the CO-to-H2 and the [CI]-to-H2 conversion factors in galaxies` (`Bisbas et al., 2025 <https://ui.adsabs.harvard.edu/abs/2025A%26A...697A.115B/abstract>`_)
* `3D-PDR Orion dataset and NeuralPDR: Neural Differential Equations for Photodissociation Regions` (`Vermarien et al., 2024 <https://ui.adsabs.harvard.edu/abs/2024arXiv241200758V/abstract>`_)
* `A new analytic approach to infer the cosmic-ray ionization rate in hot molecular cores from HCO+, N2H+, and CO observations` (`Luo et al., 2024 <https://ui.adsabs.harvard.edu/abs/2024A%26A...690A.293L/abstract>`_)
* `Reevaluation of the Cosmic-Ray Ionization Rate in Diffuse Clouds` (`Obolentseva et al., 2024 <https://ui.adsabs.harvard.edu/abs/2024ApJ...973..142O/abstract>`_)
* `α-enhanced astrochemistry: the carbon cycle in extreme galactic conditions` (`Bisbas et al., 2024 <https://ui.adsabs.harvard.edu/abs/2024MNRAS.527.8886B/abstract>`_)
* `SUNRISE: The rich molecular inventory of high-redshift dusty galaxies revealed by broadband spectral line surveys` (`Yang et al., 2023 <https://ui.adsabs.harvard.edu/abs/2023A%26A...680A..95Y/abstract>`_)
* `Abundance Ratios of OH/CO and HCO+/CO as Probes of the Cosmic-Ray Ionization Rate in Diffuse Clouds` (`Luo et al., 2023 <https://ui.adsabs.harvard.edu/abs/2023ApJ...946...91L/abstract>`_)
* `PDFCHEM: A new fast method to determine ISM properties and infer environmental parameters using probability distributions` (`Bisbas et al., 2023 <https://ui.adsabs.harvard.edu/abs/2023MNRAS.519..729B/abstract>`_)
* `Dependence of Chemical Abundance on the Cosmic-Ray Ionization Rate in IC 348` (`Luo et al., 2023 <https://ui.adsabs.harvard.edu/abs/2023ApJ...942..101L/abstract>`_)
* `Cosmic-ray-induced H2 line emission. Astrochemical modeling and implications for JWST observations` (`Gaches et al., 2022 <https://ui.adsabs.harvard.edu/abs/2022A%26A...664A.150G/abstract>`_)
* `The Origin of the [C II] Deficit in a Simulated Dwarf Galaxy Merger-driven Starburst` (`Bisbas et al., 2022 <https://ui.adsabs.harvard.edu/abs/2022ApJ...934..115B/abstract>`_)
* `Insights into the collapse and expansion of molecular clouds in outflows from observable pressure gradients` (`Dasyra et al., 2022 <https://ui.adsabs.harvard.edu/abs/2022NatAs...6.1077D/abstract>`_)
* `The impact of cosmic-ray attenuation on the carbon cycle emission in molecular clouds` (`Gaches et al., 2022 <https://ui.adsabs.harvard.edu/abs/2022A%26A...658A.151G/abstract>`_)
* `Photodissociation region diagnostics across galactic environments` (`Bisbas et al., 2021 <https://ui.adsabs.harvard.edu/abs/2021MNRAS.502.2701B/abstract>`_)
* `Star cluster formation in Orion A` (`Lim et al., 2021 <https://ui.adsabs.harvard.edu/abs/2021PASJ...73S.239L/abstract>`_)
* `The Astrochemical Impact of Cosmic Rays in Protoclusters. II. CI-to-H2 and CO-to-H2 Conversion Factors` (`Gaches et al., 2019 <https://ui.adsabs.harvard.edu/abs/2019ApJ...883..190G/abstract>`_)
* `Simulating the atomic and molecular content of molecular clouds using probability distributions of physical parameters` (`Bisbas et al. 2019 <https://ui.adsabs.harvard.edu/abs/2019MNRAS.485.3097B/abstract>`_)
* `The interstellar medium properties of heavily reddened quasarsand companions at z~2.5 with ALMA and JVLA` (`Banerji et al., 2018 <https://ui.adsabs.harvard.edu/abs/2018MNRAS.479.1154B/abstract>`_)
* `New places and phases of CO-poor/C I-rich molecular gas in the Universe` (`Papadopoulos et al., 2018 <https://ui.adsabs.harvard.edu/abs/2018MNRAS.478.1716P/abstract>`_)
* `The inception of star cluster formation revealed by [C II] emission around an Infrared Dark Cloud` (`Bisbas et al., 2018 <https://ui.adsabs.harvard.edu/abs/2018MNRAS.478L..54B/abstract>`_)
* `A Model for Protostellar Cluster Luminosities and the Impact on the CO-H2 Conversion Factor` (`Gaches & Offner, 2018 <https://ui.adsabs.harvard.edu/abs/2018ApJ...854..156G/abstract>`_)
* `Deriving a multivariate a_co conversion function using the [C II]/CO (1-0) ratio and its application to molecular gas scaling relations` (`Accurso et al., 2017 <https://ui.adsabs.harvard.edu/abs/2017MNRAS.470.4750A/abstract>`_)
* `GMC Collisions as Triggers of Star Formation. V. Observational Signatures` (`Bisbas et al., 2017 <https://ui.adsabs.harvard.edu/abs/2017ApJ...850...23B/abstract>`_)
* `Cosmic-ray Induced Destruction of CO in Star-forming Galaxies` (`Bisbas et al., 2017 <https://ui.adsabs.harvard.edu/abs/2017ApJ...839...90B/abstract>`_)
* `ALMA observations of atomic carbon in z ∼ 4 dusty star-forming galaxies` (`Bothwell et al., 2017 <https://ui.adsabs.harvard.edu/abs/2017MNRAS.466.2825B/abstract>`_)
* `Radiative transfer meets Bayesian statistics: where does a galaxy's [C II] emission come from?` (`Accurso et al., 2017 <https://ui.adsabs.harvard.edu/abs/2017MNRAS.464.3315A/abstract>`_)
* `Photochemical-dynamical models of externally FUV irradiated protoplanetary discs` (`Haworth et al., 2016 <https://ui.adsabs.harvard.edu/abs/2016MNRAS.463.3616H/abstract>`_)
* `External photoevaporation of protoplanetary discs in sparse stellar groups: the impact of dust growth` (`Facchini et al. 2016 <https://ui.adsabs.harvard.edu/abs/2016MNRAS.457.3593F/abstract>`_)
* `TORUS-3DPDR: a self-consistent code treating three-dimensional photoionization and photodissociation regions` (`Bisbas et al. 2015 <https://ui.adsabs.harvard.edu/abs/2015MNRAS.454.2828B/abstract>`_)
* `Effective Destruction of CO by Cosmic Rays: Implications for Tracing H2 Gas in the Universe` (`Bisbas et al. 2015 <https://ui.adsabs.harvard.edu/abs/2015ApJ...803...37B/abstract>`_)
* `Astrochemical Correlations in Molecular Clouds` (`Gaches et al. 2015 <https://ui.adsabs.harvard.edu/abs/2015ApJ...799..235G/abstract>`_)
* `A photodissociation region study of NGC 4038` (`Bisbas et al. 2014 <https://ui.adsabs.harvard.edu/abs/2014MNRAS.443..111B/abstract>`_)
* `An alternative accurate tracer of molecular clouds: the 'XCi-factor'` (`Offner et al. 2014 <https://ui.adsabs.harvard.edu/abs/2014MNRAS.440L..81O/abstract>`_)
* `Modeling the Atomic-to-molecular Transition and Chemical Distributions of Turbulent Star-forming Clouds` (`Offner et al. 2013 <https://ui.adsabs.harvard.edu/abs/2013ApJ...770...49O/abstract>`_)
