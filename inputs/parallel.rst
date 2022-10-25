
Parallelism
-----------

Some comments on parallelism and reporting of time taken for parallel
overheads.

.. toctree::
   :maxdepth: 1

A hierarchy of parallelism is employed in the code. At the most coarse
grained level, domain decomposition is implemented via the message
pessing interface (MPI). Within each subdomain, computational kernels
are executed in a threaded model which may involve either OpenMP for
CPU architectures, or an appropriate GPU model. This means the user
may have a number of choices in how to deploy resources for problems
of any given size.


Distributed memory parallelism
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



Halo swap mechanisms
~~~~~~~~~~~~~~~~~~~~

Under usual circumstances, it should not be necessary to select the
type of halo swap used for the various lattice-based fields in the
calculation. The code will select the relevant mechanism. In
particular, relevant CPU/GPU halo swaps will be applied.

If explicit control is required, one can use some of the following
flags.

Lattice distribution halo swaps
"""""""""""""""""""""""""""""""

There a number of different halo swap mechaniams for the lattice Boltzmann
distributions. One or other may be selected explicitly via

.. code-block:: none

  lb_halo_scheme          lb_halo_openmp_full       # host (full)
  lb_halo_scheme          lb_halo_openmp_reduced    # host (reduced)
  lb_halo_scheme          lb_halo_target            # target (full) [default]

The two host options (which, despite their names, will work with or
without OpenMP) are full or reduced. The reduced halo swap may be
quicker as it sends only distributions propagating in the relevant
direction. However, the reduced halo swap should not be used if
solid objects (boundaries or colloids) are present. It is intended
for fluid only computations. The target halo swap handles the extra
complexity in the GPU case: this is always a 'full' halo swap at the
moment.

There is an additional restriction that if two distribitions are
needed (two-distribution binary fluid case), the target halo swap
must be used.

Halo swaps for other fields
"""""""""""""""""""""""""""

There are similar options for other field types.

.. code-block:: none

  field_halo_openmp       yes                     # [no] use host scheme
  field_halo_verbose      yes                     # [no] report information
  hydro_halo_scheme       hydro_u_halo_target     # [default]
  hydro_halo_scheme       hydro_u_halo_openmp     # updated host scheme
  hydro_halo_scheme       hydro_u_halo_host       # older host version

There is no concept of a reduced halo swap for these data; the full
data are always exchanged. The ``field_halo_openmp`` option is only
available in liquid crystal casess at the moment.


Threaded parallelism via OpenMP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The recommended way to run the OpenMP version is to use one MPI
task per NUMA region. Most modern machines have two sockets per
node, and each socket may be one or more NUMA regions. The
number of threads should be set to the number of physical cores
per NUMA region. Many modern processors report the number of
"CPUs" available, which is often two times the number of physical
cores owing to the ability to run two threads in hardware per
physical core.

Typically, for 16 cores per NUMA regions, one might set

.. code-block:: none

  export OMP_NUM_THREADS=16
  export OMP_PLACES=cores

Local details may vary on how exactly to run in this hybrid MPI/OpenMP
mode.

It is possible to specify "first touch" policy for a number of data
structures in the input file:

.. code-block:: none

   lb_data_use_first_touch      yes    # lattice Boltzmann data [no]
   field_data_use_first_touch   yes    # field data: default is [no]

The field option only refers to tensor order parameter at the moment.
All other fields have the default setting.


Threaded parallelism for GPU
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usual mode of opertation envisaged for GPU-based architectures
is one MPI task per GPU. The code will determine internally how to
aportion individual GPUs to each MPI task.

At the moment the code does no checking that the assumption of one
MPI task per GPU device is valid, so some care may be required if
running multi-GPU jobs.


Timing
^^^^^^

Halo swap imbalance time
~~~~~~~~~~~~~~~~~~~~~~~~

A facility is provided to report the imbalance time observed at the
point of the lattice distribition halo swap. This introduces an
explicit synchronisation immediately before the halo swap to
allow the separation of load  imbalance and actually communication
overhead.

.. code-block:: none

  lb_halo_scheme_report_imbalance    yes      # Default no

This appears as part of the breakdown of the lattice halo swap time, e.g.,

.. code-block:: none

     Lattice halos:      0.001      0.013     84.246   0.000842 (100000 calls)
      -> imbalance:      0.000      0.013     14.277   0.000143 (100000 calls)

The introduction of the synchronisation itself introduces an overhead,
so this option should not be used for production runs.
