.. _sec-rttool:

The RT-tool Algorithm
=====================

**RT-tool** solves the radiative transfer equation for one-dimensional **3D-PDR** models. You can download the algorithm from this `GitHub link <https://github.com/itamos-ism/RT-tool>`_.

The algorithm can be used with the Python wrapper ``pyRTtool.py``. Refer to the ``HandsOn_RTtool.ipynb`` Jupyter notebook for usage instructions found on GitHub.

Running from Terminal
~~~~~~~~~~~~~~~~~~~~~

After running **3D-PDR**, compile **RT-tool**:

.. code-block:: bash

   cd RT-tool/
   make clean; make
   cd ..

Edit the ``paramsRT.dat`` file:

::

   1) sims
   2) test
   3) 1
   4) 0

* **Entry 1**: PDR output directory
* **Entry 2**: Model prefix
* **Entry 3**: Turbulent velocity (default 1 km/s)
* **Entry 4**: Redshift (to control the CMB temperature)

Run the tool:

.. code-block:: bash

   ./RTtool

Outputs are written to ``sims/`` (where ``PREF`` is the prefix of the model and ``LINE`` the calculated transition line):

* ``Tr_PREF_LINE.dat``: Radiation temperatures for coolant transitions
* ``tau_PREF_LINE.dat``: Optical depths for line transitions

Format of ``Tr_PREF_LINE.dat``::

   Ntot, NH2, Tgas, Ncool, Tr(transitions)

Format of ``tau_PREF_LINE.dat``::

   tau1, tau2, ...
