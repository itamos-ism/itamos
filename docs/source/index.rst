.. itamos-ism documentation master file, created by
   sphinx-quickstart on Wed Nov  5 12:25:23 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

The ITAMOS project
==================

The **ITAMOS** project (\ **I**\ ntelligent \ **T**\ echniques for **A**\ tomic and **MO**\ lecular **S**\ tudies of the interstellar medium) focuses on developing state-of-the-art numerical codes 
to simulate the distribution of abundances and line emissions of photodissociation regions (PDRs) in both galactic and extragalactic contexts.

PDRs are the sites of rich chemical and physical processes, bridging the fully ionized and molecular phases of the interstellar medium (ISM). They play a critical role in regulating star-formation, chemical evolution, and the energy balance of galaxies.

The emission lines of PDRs provide vital diagnostics for understanding the conditions within molecular clouds, what powers the energy in galaxies and for identifying the ISM 
environmental parameters in general. By studying PDRs we can decode the life cycle of gas and dust in galaxies, shedding new light on the star-formation process and the galaxy 
evolution across all epochs.

However, accurately modeling PDRs is a computationally intensive challenge especially when modelling three-dimensional distributions. It requires solving an extended set of differential 
equations for estimating the abundances of species, for calculating the level populations of coolants to perform radiative transfer, and for calculating the overall heating and cooling 
processes, under conditions of extreme complexity. 

The ITAMOS project addresses this challenge by advancing computational techniques, such as ray-tracing and high-performance programming, to create highly efficient and precise simulation tools. 
These tools are designed to model PDRs at unprecedented levels of chemical detail and spatial resolution, enabling new insights into their structure and behavior.

The ITAMOS project includes the following publicly available codes:

* **3D-PDR**\ : Modelling PDR chemistry in one and three dimensions
* **RAYTHEIA**\ : Stand-alone state-of-the-art ray-tracing algorithm included in 3D-PDR
* **RT-tool**\ : Estimate radiation temperatures from one-dimensional PDR models
* **RT-synth**\ : Construct synthetic observations from three-dimensional PDR models
* `PDFchem <https://github.com/tbisbas/PDFchem>`_: Estimate PDR diagnostics from entire column density distributions as inputs (new version to be released soon)


.. toctree::
   :maxdepth: 1
   :caption: INTRODUCTION:

   introduction/pdrs
   introduction/escapeprobability
   introduction/rt

.. toctree::
   :maxdepth: 1
   :caption: 3D-PDR:

   3d-pdr/3dpdr
   3d-pdr/gallery
   3d-pdr/installation
   3d-pdr/pdrstudio
   3d-pdr/makefile
   3d-pdr/params
   3d-pdr/species
   3d-pdr/ics
   3d-pdr/outputs
   3d-pdr/convert
   3d-pdr/examples
   3d-pdr/heating
   3d-pdr/escapeprob
   3d-pdr/dust
   3d-pdr/h2form
   3d-pdr/rates
   3d-pdr/changeTgas
   3d-pdr/shield
   3d-pdr/levelpop
   3d-pdr/ramestimate

.. toctree::
   :maxdepth: 1
   :caption: RAYTHEIA:

   raytheia/raytheia

.. toctree::
   :maxdepth: 1
   :caption: RT-tool:

   rt-tool/RT-tool
   rt-tool/radex

.. toctree::
   :maxdepth: 1
   :caption: RT-synth:

   rt-synth/RT-synth
   rt-synth/makefile
   rt-synth/params
   rt-synth/run
   rt-synth/vel2fits
   rt-synth/gallery
