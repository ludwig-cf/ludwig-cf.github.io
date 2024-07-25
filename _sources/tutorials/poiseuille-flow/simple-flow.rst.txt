
Poisseuille Flow
----------------

.. contents:: A simple exercise in one-dimensional Newtonian flow
   :depth: 2
   :local:
   :backlinks: none


A simple exercise: Poisseuille flow in a Newtonian fluid in a cavity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is assumed that you have built the code (using here ``-D_D3Q19_``)
and have the executable available. It is further assumed that the
code has been compiled with both OpenMP and MPI available. The
exact details of how an MPI program should be started may vary; for
illustration we assume the fairly common case where OpenMPI is used
and there is an associated command ``mpirun`` to launch the parallel
program.

The following steps are needed:

1. Copy the sample input file located in the repository at
   ``docs/tutorial/test1/input`` to your working directory.
   By default, the executable will look for a file named ``input``
   in the current directory.

2. Run the code in your working directory using, e.g.,

   .. code-block:: none

      $ export OMP_NUM_THREADS=1
      $ ./Ludwig.exe

3. You should now see various outputs and statistics being reported to
   standard output (the screen). Statistics are reported every 200
   time steps, and one set of output is written to file at the end
   of the run.

4. When execution has completed (you should see lastly
   ``Ludwig finished normally.``) the following files should exist.

   .. code-block:: none

     $ ls
     dist-000010000.001-001 rho-000010000.001-001 vel-000010000.001-001
     dist-metadata.001-001  rho-metadata.001-001  vel-metadata.001-001

   The ``dist-`` files are the lattice Boltzmann distributions, the
   ``rho-`` files are the fluid density, and the ``vel-`` files are
   the fluid velocity. The timestep (here 10000) is encoded in the filename
   to 9 figures. The metadata files contain a JSON description of the
   data and the files with extension ``.001-001`` are the data
   themselves. The ``.001-001`` is related to parallel output and
   encodes the fact that this is a single file (1 of 1).

   These are all ASCII files (as requested in the input file) and can be
   inspected using standard utilities.

5. The velocity file consists of three columns, with three components
   of the velocity. For this problem, flow is only in the
   :math:`y`-direction which is the second column.
   For example, using ``gnuplot``, one could use

   .. code-block:: none

     gnuplot> p "vel-000010000.001-001" u 2

   to obtain a simple picture.

.. figure:: gnuplot.svg
   :alt: A basic output from gnuplot
   :figwidth: 80%
   :align: center

6. Rename or remove the existing output files. We may run the same problem
   using OpenMP threads via

   .. code-block:: none

     $ export OMP_NUM_THREADS=2
     $ ./Ludwig.exe

7. One should be able to compare the output and see it is the same except
   for the report:

   .. code-block:: none

     OpenMP threads: 2; maximum number of threads: 8.

   The content of the output files will be the same.
   For such a small problem size there will probably be no
   very significant difference in the time taken to run.
   [SeeNoteHardwareThreads]_


8. Likewise, one can run the MPI version via, e.g.,

   .. code-block:: none

     $ export OMP_NUM_THREADS=1
     $ mpirun -np 2 ./Ludwig.exe

   This will report

   .. code-block:: none

     Welcome to: Ludwig v0.19.0 (MPI version running on 2 processes)

   and will also report the decomposition of the total of 64 lattice
   sites into two processes worth of 32 each:

   .. code-block:: none

     System size:    64 1 1
     Decomposition:   2 1 1
     Local domain:   32 1 1

9. Note that if you have not renamed or deleted the previous set of output
   files, existing output files will be overwritten without warning or
   error. Again one can inspect the results to see that the output is
   the same.

10. No further processing of the output files is required. The data is
    always in the same format independent of the number of MPI tasks
    or threads.

|

.. [SeeNoteHardwareThreads] This is example is taken from a laptop with
   four hardware cores.
   Many platforms will allow two hardware threads per core: the maximum
   number of threads reported is therefore 8 for this laptop. In general,
   one would usually seek to run one thread per core at most. Running
   two threads per core will probably lead to contention for hardware
   resources and no net increase in performance.
