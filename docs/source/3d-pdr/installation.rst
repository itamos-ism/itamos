Getting started
===============

You can download the **3D-PDR** code from this `GitHub link <https://github.com/itamos-ism/3D-PDR>`_.

To install **3D-PDR**, you must first install **SUNDIALS**.  
You will need the **gfortran** and **gcc** compilers, as well as **CMake**.

Before proceeding, create a home directory where the code will be installed, for example:

.. code-block:: console

    $ cd
    $ mkdir PDRmodels
    $ cd PDRmodels


Installing CMake
----------------

First, check whether ``cmake`` is already installed on your system.  
If it is, you may skip this section.

To install ``cmake``, download it within your ``~/PDRmodels/`` directory:

.. code-block:: console

    $ wget https://github.com/Kitware/CMake/releases/download/v3.23.0/cmake-3.23.0-linux-x86_64.tar.gz

Extract the tarball:

.. code-block:: console

    $ tar -xzvf cmake-3.23.0-linux-x86_64.tar.gz

Move the extracted directory to ``/opt``:

.. code-block:: console

    $ sudo mv cmake-3.23.0-linux-x86_64 /opt/cmake-3.23.0

Create a symbolic link to make it accessible system-wide:

.. code-block:: console

    $ sudo ln -sf /opt/cmake-3.23.0/bin /usr/bin/

Verify that ``cmake`` is properly installed:

.. code-block:: console

    $ cmake --version

For more information about CMake, visit the `CMake official website <https://cmake.org>`_.


Installing SUNDIALS
-------------------

Clone the **SUNDIALS** repository:

.. code-block:: console

    $ git clone https://github.com/LLNL/sundials.git sundials

Create and enter a build directory:

.. code-block:: console

    $ mkdir sundials/build
    $ cd sundials/build

Configure SUNDIALS using ``cmake``:

.. code-block:: console

    $ cmake -DCMAKE_INSTALL_PREFIX=/YOUR-HOMEPATH/PDRmodels/sundials ../

Build and install:

.. code-block:: console

    $ make
    $ make install

Next, edit your shell configuration file (e.g. ``~/.bashrc``) and add the following lines:

.. code-block:: console

    export LD_LIBRARY_PATH=/home/USERNAME/PDRmodels/sundials/lib
    export SUNDIALS_DIR=/home/USERNAME/PDRmodels/sundials

Then, reload your shell configuration:

.. code-block:: console

    $ source ~/.bashrc

.. note::

   If you are using a different shell, adjust the above commands accordingly.

For further details on SUNDIALS, see the `SUNDIALS GitHub repository <https://github.com/LLNL/sundials>`_.


Installing 3D-PDR
-----------------

Within the directory ``~/PDRmodels/``, extract the 3D-PDR package:

.. code-block:: console

    $ cd ~/PDRmodels/
    $ tar xvzf 3DPDR.tgz

Navigate to the source directory:

.. code-block:: console

    $ cd 3D-PDR-main/src


Compiling the Code and Running a Test Model
-------------------------------------------

Before compiling, open the file ``config.mk`` and ensure that the ``F90`` (Fortran) and ``CC`` (C++) compiler paths are correctly set.

.. note::

   **For macOS users:** Compilation may fail if you have a Conda environment activated.  
   Deactivate Conda before compiling:

   .. code-block:: console

       $ conda deactivate

To compile **3D-PDR**:

.. code-block:: console

    $ make

If compilation completes successfully, an executable named ``3DPDR`` will appear in your ``~/PDRmodels/`` directory.

You can now run a test model to verify that the installation is successful:

.. code-block:: console

    $ ./3DPDR

The test run should complete within a few seconds (depending on your system).  

To confirm that the run was successful, compare the **gas temperature vs. visual extinction** results with the provided benchmark model.

Plot columns **3 (visual extinction)** and **4 (gas temperature)** of your output file ``sims/test.pdr.fin`` against those in ``benchmark/model.pdr.fin``.

For example, using **gnuplot**:

.. code-block:: console

    set log
    plot 'sims/test.pdr.fin' u 3:4 w l t 'my model', \
         'benchmark/model.pdr.fin' u 3:4 w l t 'benchmark'

If the curves match, your installation is complete and functioning correctly.

---

Congratulations!  
You are now ready to begin running your own 3D-PDR simulations.
