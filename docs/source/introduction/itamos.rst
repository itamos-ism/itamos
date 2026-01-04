ITAMOS
======

The **ITAMOS** project (\ **I**\ ntelligent \ **T**\ echniques for **A**\ tomic and **MO**\ lecular **S**\ tudies of the interstellar medium) focuses on developing state-of-the-art numerical codes 
to simulate the distribution of abundances and line emissions of photodissociation regions (PDRs) in both galactic and extragalactic contexts.

The emission lines of PDRs provide vital diagnostics for understanding the conditions within molecular clouds, what powers the energy in galaxies and for identifying the ISM 
environmental parameters in general. By studying PDRs we can decode the life cycle of gas and dust in galaxies, shedding new light on the star-formation process and the galaxy 
evolution across all epochs.

However, accurately modeling PDRs is a computationally intensive challenge especially when modelling three-dimensional distributions. It requires solving an extended set of differential 
equations for estimating the abundances of species, for calculating the level populations of coolants to perform radiative transfer, and for calculating the overall heating and cooling 
processes, under conditions of extreme complexity. 

The ITAMOS project addresses this challenge by advancing computational techniques, such as ray-tracing and high-performance programming, to create highly efficient and precise simulation tools. 
These tools are designed to model PDRs at unprecedented levels of chemical detail and spatial resolution, enabling new insights into their structure and behavior.

The ITAMOS project includes the following publicly available codes:

* :ref:`3D-PDR <sec-3dpdr>`: Modelling PDR chemistry in one and three dimensions
* :ref:`RAYTHEIA <sec-raytheia>`: Stand-alone state-of-the-art ray-tracing algorithm included in 3D-PDR
* :ref:`RT-tool <sec-rttool>`: Estimate radiation temperatures from one-dimensional PDR models
* :ref:`RT-synth <sec-rtsynth>`: Construct synthetic observations from three-dimensional PDR models
* `PDFchem <https://github.com/tbisbas/PDFchem>`_: Estimate PDR diagnostics from entire column density distributions as inputs (new version to be released soon)
