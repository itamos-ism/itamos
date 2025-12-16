Running RT-synth
================

Before running ``RT-synth``, make sure that the output directory specified in the parameters file exists (e.g. ``simsRTsynth/``). 
Execute ``RT-synth`` as follows:

.. code-block:: bash

   cd RT-synth
   make clean; make
   cd ..
   ./RTsynth

or specify a particular parameters file as:

.. code-block:: bash

   ./RTsynth -p=paramUser.dat


Outputs
~~~~~~~

All outputs are written in the ``simsRTsynth`` directory specified in the RT-synth parameters file. In particular:

* ``RT_[prefix]_[los_direction]_cds.dat``: Column densities (H₂, HI, C⁺, C, CO, HCO⁺, total)
* ``RT_[prefix]_[los_direction]_[line].dat``: Velocity-integrated maps
* ``RT_vel_[prefix]_[los_direction]_[line].dat``: Radiation temperature per velocity channel
