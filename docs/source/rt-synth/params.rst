The paramsRTsynth.dat
=====================

Specify the parameters of the **RT-synth** model as follows:

::

   1) sims            !Directory of PDR output
   2) simsRTsynth     !Output directory of RT-synth
   3) yourmodelpref   !Model prefix
   4) gasvelocity     !velocity file
   5) 1               !Vturb (km/s)
   6) 1               !dust-to-gas norm. to 1e-2
   7) 0               !Redshift (for Tcmb)

* **Entries 1-7**: General parameters (directory, output directory, prefix, velocity file, turbulent velocity, dust-to-gas ratio, redshift)

.. note::
      * The ``gasvelocity`` file should be inside the ``sims/`` directory
      * Make sure that the output directory ``simsRTsynth/`` (or another name specified by the user) exists before running a model

::

   8)  +z              !direction of integration
   9)  64              !ctot
   10)  1               !coolant
   11) 1               !line
   12) 0               !ignore edge cells
   13) -20             !velmin (km/s)
   14) 20              !velmax (km/s)
   15) 5               !velstep

* **Entries 8-15**: Observation-specific parameters (direction, resolution, coolant, line, edge cells, velocity range)

Custom parameter files (e.g., ``paramsUser.dat``) can be specified using the command-line options ``-p=`` or ``--params``:

.. code-block:: bash

    ./RTsynth -p=paramsUser.dat

.. note::
   Parameter filenames must not exceed 50 characters.
