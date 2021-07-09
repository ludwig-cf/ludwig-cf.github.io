
Building and Testing
====================

This page contains details of how to install *Ludwig*; all that
is required in the most simple case is Gnu make and a functioning
C compiler.

.. toctree::
   :maxdepth: 2


   

Configuration
-------------

Compilation of the main code is controlled by the ``config.mk`` in the
top-level directory. This is a (gnu-) Makefile fragment which is included
in all the relevant Makefiles in the code to describe the local configuration.

A number of example ``config.mk`` files are provided in the ``config``
directory (which can be copied and adjusted as needed). This section
discusses a number of issues which influence compilation.


Parallel Build
^^^^^^^^^^^^^^
You will need to copy one of the existing configuration files from the
``config`` directory to the top level directory. For a parellel build use,
e.g.,

.. code-block:: bash

  $ cp config/unix-mpicc-default.mk config.mk

The configuration file defines a number of variables which include ``MPICC``,
the compiler required for MPI programs:

.. code-block:: make

  BUILD   = parallel                # here "parallel" for message passing
  MODEL   = -D_D3Q19_               # one of _D2Q9_ _D3Q15_ or _D3Q19_

  CC      = mpicc                   # C compiler
  CFLAGS  = -O2 -DNDEBUG            # compiler flags

  AR      = ar                      # standard ar command
  ARFLAGS = -cru                    # flags for ar
  LDFLAGS =                         # additional link time flags

  MPI_INC_PATH      =               # path to mpi.h (if required)
  MPI_LIB_PATH      =               # path to libmpi.a (if required)
  MPI_LIB           =               # -lmpi (if required)

  LAUNCH_SERIAL_CMD =               # serial launch command
  LAUNCH_MPIRUN_CMD = mpirun        # parallel launch command
  MPIRUN_NTASK_FLAG = -np           # flag to set number of MPI tasks


If the MPI compiler wrapper does not require that MPI include and library
paths be explicitly defined, the relevant variables can be left blank
(as they are in the above example).

The test system requires that an MPI program can be started (often via
``mpirun``) so relevant variables are also set. Note the number of MPI
tasks used by the tests is not specified in the configuration.


Serial Build
^^^^^^^^^^^^

If no MPI library is available, or strictly serial execution is wanted,
the ``BUILD`` configuration variable should be set to ``serial``.
In this case, a stub MPI library is compiled as a replacement which allows
the code to operate in serial.

To configure a serial build copy, e.g.,


.. code-block:: bash

  $ cp config/unix-gcc-default.mk config.mk

to the top-level directory.

Again, you may need to edit the file to reflect local conditions.
A minimum configuration might be:

.. code-block:: make

  BUILD   = serial                  # here "serial"
  MODEL   = -D_D3Q19_               # preprocessor macro for model

  CC      = gcc                     # C compiler
  CFLAGS  = -O -g -Wall             # compiler options

  AR      = ar                      # standard ar command
  ARFLAGS = -cru                    # standard ar options
  LDFLAGS =                         # additional link time flags

  MPI_INC_PATH      = ./mpi_s       # stub MPI include location
  MPI_LIB_PATH      = ./mpi_s       # stub MPI library location
  MPI_LIB           = -lmpi         # MPI library link

  LAUNCH_SERIAL_CMD =               # blank


The stub MPI library should be built before the main compilation. To do this,

.. code-block:: bash

  $ make serial
  $ make


Compilation
-----------

With a relevant configuration file in the top-level directory, compilation
proceeds via


.. code-block:: bash

  $ make

This will build the executable, the unit tests, and a small number of
utilities. To remove these files, and other compilation products


.. code-block:: bash

  $ make clean



Preprocessor options
^^^^^^^^^^^^^^^^^^^^

A number of standard C-preprocessor macros are relevant at compilation time,
and should be set in the configuration file. All are introduced to the
compiler in the usual way via the -D flag. (Note this is also the form of
the ``MODEL`` configuration varaible which determines the LB basis set.)
A summary is:

.. code-block:: none

  Macro           Purpose

  _D2Q9_            # Use D2Q9  model
  _D3Q15_           # Use D3Q15 model
  _D3Q19_           # Use D3Q19 model
                    # Set via the MODEL configuration variable. It is
                    # erroneous to define more than one of these three.

  NDEBUG            # Standard C option to disable assertions.
                    # Should be used for all production runs.

  NSIMDVL=4         # Set the SIMD vector length used in inner loops.
                    # The default vector length is 1. The best choice
                    # for performance depends on hardware (2, 4, 8...)

  ADDR_SOA          # Use SOA array addressing (for GPU targets).
                    # Default is AOS (for CPU).

Apart from the choice of ``MODEL`` preprocessor options should be specfied
via the variable ``CFLAGS`` in the normal way.


Target Data Parallel
^^^^^^^^^^^^^^^^^^^^

The code includes a lightweight abstraction of threaded parallelism referred
to as targetDP. This supports either no threads (the default), OpenMP threads
(when the target for production runs is a CPU), or CUDA threads (if the target
device is an NVIDIA GPU). Control of the targetDP layer is via the compiler
and compiler options.

Using OpenMP
""""""""""""

For OpenMP threads, the compiler options ``CFLAGS`` should include the
standard flag for enabling OpenMP; the number of threads is set at runtime
via ``OMP_NUM_THREADS`` in the usual way. For example, for Intel compilers
this might be

.. code-block:: make

  CFLAGS = -fast -qopenmp


Using CUDA
""""""""""

If NVIDIA hardware is available and required, the code should be compiled
with ``nvcc``, which will cause the targetDP layer to make the appropriate
thread model available.

An appropriate configuration file might be:

.. code-block:: make

  BUILD   = parallel
  MODEL   = -D_D3Q19_

  CC      = nvcc
  CFLAGS  = -ccbin=icpc -DADDR_SOA -DNDEBUG -arch=sm_70 -x cu -dc

  AR      = ar
  ARFLAGS = -cr
  LDFLAGS = -ccbin=icpc -arch=sm_70

  MPI_HOME     = /path/to/mpi
  MPI_INC_PATH = -I$(MPI_HOME)/include64
  MPI_LIB_PATH = -L$(MPI_HOME)/lib64 -lmpi

  LAUNCH_SERIAL_CMD =
  LAUNCH_MPIRUN_CMD = mpirun
  MPIRUN_NTASK_FLAG = -np

As this is a parallel build using the ``nvcc`` compiler (with the native
compiler being Intel ``icpc`` in this case), we specify explicitly the
location of MPI include and library files.

Note the ``-DADDR_SOA`` preprocessor macro is set to provide the correct
memory access for coalescing on GPU architectures. The appropriate ``-arch``
flag for ``nvcc`` is also provided to describe the relevant hardware
(at both compile and link time).



Electrokinetics and using PETSc
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is the option to use PETSc to solve the Poisson equation required in
the electrokinetic problem. A rather less efficient in-built method can be
used if PETSc is not available. We suggest using PETSC v3.4 or later
available from Argonne National Laboratory http://www.mcs.anl.gov/petsc/.

If PETSc is required, please enter the additional variables in the
``config.mk`` file:

.. code-block:: make

  HAVE_PETSC = true
  PETSC_INC  = /path/to/petsc/include
  PETSC_LIB  = /path/to/petsc/lib


In addition, there is a choice of finite difference stencil size for the
electrokinetic problem which is determined at compile time. The choices
are via preprocesor options

.. code-block:: none

  -DNP_D3Q6       #  7-point stencil in 3 dimensions (the default)
  -DNP_D3Q18      # 19-point stencil in 3 dimensions
  -DNP_D3Q18      # 27-point stencil in 3 dimensions


Testing
-------

Various tests are found in the ``tests`` subdirectory. Type ``make test``
from the top level to run the default tests, which will take a few minutes. 

.. code-block :: none

  $ make -s test
  PASS     ./unit/test_pe
  PASS     ./unit/test_coords
  ...

Unit tests
^^^^^^^^^^

Unit tests are found in ``./tests/unit`` and report pass or fail for
each case. The unit tests can be run in either serial or parallel,
and run as part of the default test target from the top level. Some
tests may report 'skip' if they are not relevant on a particular
platform.


Regression tests
^^^^^^^^^^^^^^^^

A series of regression tests is available which run the main code with
a given input and compare the answer with a reference output.

Regression tests may be run from the ``tests`` directory, e.g.,

.. code-block :: none

  $ cd tests
  $ make d3q19-short
  
  PASS     ./serial-actv-s01.inp
  PASS     ./serial-actv-s02.inp
  ...

Each test should report pass or fail. Failures will produce a diff-like
output showing how the current result differs from the reference result.
Floating point numbers are checked to within a tolerance set in the
``./tests/awk-fp-diff.sh`` script. Results can be subject to variations
slightly larger than the tolerance depending on the platform/compiler.
The default tests should be run in serial.



