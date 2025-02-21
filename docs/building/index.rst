
Building and Testing
====================

This page contains details of how to install *Ludwig*; all that
is required in the most simple case is Gnu make and a functioning
C compiler.

.. contents:: In this section
   :depth: 3
   :local:
   :backlinks: none


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
  MODEL   = -D_D3Q19_               # one of _D2Q9_ _D3Q15_, _D3Q19_ or _D3Q27_
  TARGET  =                         # Optional: for GPU builds

  CC      = mpicc                   # C compiler
  CFLAGS  = -O2 -DNDEBUG            # compiler flags

  AR      = ar                      # standard ar command
  ARFLAGS = -cru                    # flags for ar
  LDFLAGS =                         # additional link time flags

  MPI_INC_PATH      =               # path to mpi.h (if required)
  MPI_LIB_PATH      =               # path to libmpi.a (if required)
  MPI_LIB           =               # -lmpi (if required)

  LAUNCH_MPIRUN_CMD = mpirun -np 2  # parallel launch command for tests


If the MPI compiler wrapper does not require that MPI include and library
paths be explicitly defined, the relevant variables can be left blank
(as they are in the above example).

The test system requires that an MPI program can be started (often via
``mpirun``) so relevant variables are also set. The number of MPI
tasks used by the tests must be specified in the configuration.


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
  TARGET  =

  CC      = gcc                     # C compiler
  CFLAGS  = -O -g -Wall             # compiler options

  AR      = ar                      # standard ar command
  ARFLAGS = -cru                    # standard ar options
  LDFLAGS =                         # additional link time flags

  MPI_INC_PATH      = ./mpi_s       # stub MPI include location
  MPI_LIB_PATH      = ./mpi_s       # stub MPI library location
  MPI_LIB           = -lmpi         # MPI library link


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
the ``MODEL`` configuration variable which determines the LB basis set.)
A summary is:

.. code-block:: none

  Macro           Purpose

  _D2Q9_            # Use D2Q9  model
  _D3Q15_           # Use D3Q15 model
  _D3Q19_           # Use D3Q19 model
  _D3Q27_           # Use D3Q27 model
                    # Set via the MODEL configuration variable. It is
                    # erroneous to define more than one of these three.

  NDEBUG            # Standard C option to disable assertions.
                    # Should be used for all production runs.

  NSIMDVL=4         # Set the SIMD vector length used in inner loops.
                    # The default vector length is 1. The best choice
                    # for performance depends on hardware (2, 4, 8...)

  ADDR_SOA          # Use SOA array addressing (for GPU targets).
                    # Default is AOS (for CPU).

Apart from the choice of ``MODEL`` preprocessor options should be specified
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

It is often the case that the OpenMP compiler option needs to
be specified at both compile and link stages. This can be done
conveniently via, e.g.,

.. code-block:: make

  CC = gcc -fopenmp

where we have specified the ``-fopenmp`` option relevant for GNU `gcc`.

The OpenMP implementation is strongly recommended, as it has a number
of advantages over the simple MPI-only implementation. For example, if
the local MPI domain size becomes small, it can limit both flexibility
in parameter choices (particularly for large particles), and improve
efficiency. Performance should remain reasonable as long as threads
are limited to single NUMA regions.

Using CUDA
""""""""""

If NVIDIA hardware is available and required, the code should be compiled
with ``nvcc``, which will cause the targetDP layer to make the appropriate
thread model available.

An appropriate configuration file might be:

.. code-block:: make

  BUILD   = parallel
  MODEL   = -D_D3Q19_
  TARGET  = nvcc         # Must be specified in addition to CC

  CC      = nvcc
  CFLAGS  = -ccbin=icpc -DADDR_SOA -DNDEBUG -arch=sm_70 -x cu -dc

  AR      = ar
  ARFLAGS = -cr
  LDFLAGS = -ccbin=icpc -arch=sm_70

  MPI_HOME     = /path/to/mpi
  MPI_INC_PATH = -I$(MPI_HOME)/include64
  MPI_LIB_PATH = -L$(MPI_HOME)/lib64 -lmpi

  LAUNCH_MPIRUN_CMD = mpirun -np 2

As this is a parallel build using the ``nvcc`` compiler (with the native
compiler being Intel ``icpc`` in this case), we specify explicitly the
location of MPI include and library files.

Note the ``-DADDR_SOA`` preprocessor macro is set to provide the correct
memory access for coalescing on GPU architectures. The appropriate ``-arch``
flag for ``nvcc`` is also provided to describe the relevant hardware
(at both compile and link time).


Using HIP
"""""""""

The targetDP layer supports a HIP implementation which can be used for
AMD GPUs. Compilation is via the standard ``hipcc`` compiler. A configuration
might be

.. code-block:: make

   BUILD   = parallel
   MODEL   = -D_D3Q19_
   TARGET  = hipcc

   CC      = hipcc
   CFLAGS  = -x hip -O3 -fgpu-rdc --amdgpu-target=gfx90a -DADDR_SOA

   AR      = ar
   ARFLAGS = -cr
   LDFLAGS = -fgpu-rdc --hip-link --amdgpu-target=gfx90a

   MPI_HOME     = /path/to/mpi
   MPI_INC_PATH = -I$(MPI_HOME)/include
   MPI_LIB_PATH = -L$(MPI_HOME)/lib -lmpi

The option ``-fgpu-rdc`` requests relocatable device code so that device
code from different translation units (aka source files) can be called.
It is the equivalent of ``nvcc -dc`` for NVIDIA platforms.

The current GPU architecture is specified with the ``--amdgpu-target`` option;
this will vary between platforms.

Note that ROCM Version 6 is required for Ludwig. ROCM Version 5 is known not
to work.

GPU-aware MPI
"""""""""""""

If GPU aware MPI is available, it should be used to improve performance
of GPU to GPU MPI transfers. This must be described
at compile time using the preprocessor option

.. code-block:: bash

  CFLAGS += -DHAVE_GPU_AWARE_MPI

If this option is specified erroneously, the code will fail at run time
with a segmentation violation (SEGV). Consult local documentation to find
out whether GPU aware MPI is available.


Electrokinetics and using PETSc
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is the option to use PETSc to solve the Poisson equation required in
the electrokinetic problem. A rather less efficient in-built method can be
used if PETSc is not available. We suggest a recent version of PETSc,
which is
available from Argonne National Laboratory http://petsc.org/release/.

If PETSc is required, please enter the additional variables in the
``config.mk`` file:

.. code-block:: make

  HAVE_PETSC = true
  PETSC_INC  = /path/to/petsc/include
  PETSC_LIB  = /path/to/petsc/lib


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
