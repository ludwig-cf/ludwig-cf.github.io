
Fluid Output
------------

.. toctree::
   :maxdepth: 1


Many fluid quantities defined at the level of the lattice are available
for output. The fluid quantities include the lattice Boltzmann distributions,
density and velocity, and order parameters if relevant. Output for some
quantities is optional, while output for quantities required for restart
will always be enabled automatically.


I/O for all lattice quantities
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are a significant number of options related to the input and
output of data. These concern the format of the data, the mechanism
by which the data are produced, and reporting of input/output activity.
The recommended options are:

.. code-block:: none

   default_io_mode           mpiio    # Use MPI-IO
   default_io_format         binary   # ascii or binary
   default_io_report         no       # produce a report on i/o

Here, the ``mode`` is the mechanism used to generate the output. The
recommended mode is MPI-IO, which will always work and produce the
same file irrespective of the number of MPI tasks.
The ``format`` refers to the representation of data in files: either
binary or ASCII is available. Binary format is recommended for reasons
of speed and file size; for these reasons it is also the default.

Control for specific lattice quantities
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Input and output for different quantities are controlled by specific
key value pairs. For example, options for the density field ``rho``
may be selected explicitly - overriding any default - using:

.. code-block:: none

   rho_io_mode               mpiio    # overrides default_io_mode
   rho_io_format             binary   # overrides default_io_format
   rho_io_report             yes      # overrides default_io_report

   rho_input_io_format       ascii    # overrides rho_io_mode
   rho_output_io_format      binary   # overrides default

where``rho`` has replaced ``default`` in each case. Available lattice
quantities include (using mode as an example):

.. code-block:: none

  lb_io_mode                          # LB distributions
  rho_io_mode                         # density field (scalar)
  vel_io_mode                         # velocity field (vector)
  phi_io_mode                         # order parameters

Note that all order parameter fields are treated on the same basis
including scalar order parameters ``phi``, vector order parameters ``p``,
and tensor order parameter ``q``. The format of these files for different
numbers of scalar or vector components is discussed below.

Frequency of I/O
~~~~~~~~~~~~~~~~

The following types of output may be requested at regular intervals

.. code-block:: none

  freq_rho         # fluid density
  freq_vel         # fluid velocity
  freq_phi         # order parameter
  freq_config      # Full configuration output for restart

Full configuration output includes the lattice Boltzmann distributions,
the density and velocity fields, and order parameters as required. The
density and velocity fields are required in addition to the lattice
Boltzmann distributions to ensure reproducibility at restarts.

If a full configuration is not required at the end of the run, one can
set

.. code-block:: none

   config_at_end          no      # default is "yes"

The default is that configuration output should be produced at the end.


Meta data and data
~~~~~~~~~~~~~~~~~~

Requests for file output will always create a relevant `metadata` file
for the type of output requested. For example, a request for order
parameter output will generate a single JSON file

.. code-block:: none

  phi-metadata.001-001

which contains information on the system size, order parameter fields
and so on. The meta-data file provides a description of the corresponding
data and is useful for post-processing.

Actual data appear in a separate file for each time step (at the
frequency requested), e.g.,

.. code-block:: none

  phi-000000000.001-001
  phi-000020000.001-001

would be expected for a simulation starting at time step zero, and
producing output at each 20,000 steps. The file
extension ``.001-001`` indicates this is one file in a set of one.


Multiple file output
~~~~~~~~~~~~~~~~~~~~

For the largest systems (probably larger than :math:`512^3`) run on very large
numbers of MPI tasks, it may be favourable to ask for output to be
written to more than one file. This allows a larger overall bandwidth
to disk to be obtained. The downside is that the separate files must
be recombined if a complete view is required for visualisation etc.

Output (and input) to more than one file is requested by specifying an
I/O grid. This decomposes the system in a similar way to the processor
decomposition (with one or more MPI tasks per I/O grid group).

The I/O grid is set via

.. code-block:: none

  default_io_grid 2_2_1

which would result is four I/O groups writing to four separate files, e.g.,

.. code-block:: none

  phi-000020000.001-004
  phi-000020000.002-004
  phi-000020000.003-004
  phi-000020000.004-004

with corresponding metadata files. The metadata files will detail which
portions of the complete system are held by the respective data files.


File Data Formats
^^^^^^^^^^^^^^^^^

Data format
~~~~~~~~~~~

Output can be requested in either ASCII or (raw) binary format. While
ASCII output can be appropriate for initial investigations, it is
recommended that binary format is used. The data is always stored as
8-byte floating point in binary format. In ASCII there are usually
15 decimal places of precision.

Serial storage order
~~~~~~~~~~~~~~~~~~~~

In serial (or with one MPI task with multiple threads), output occurs
to a single file. Data for fluid quantities are written to file on
a per-lattice site basis in the following order:

.. code-block:: none

  x_1 y_1 z_1   q_1 q_2 ...
  x_1 y_1 z_2   q_1 q_2 ...
  ...
  x_1 y_1 z_N   ...
  x_1 y_2 z_1   ...
  ...

Files written in parallel automatically follow this order.


Older-style I/O mode
^^^^^^^^^^^^^^^^^^^^

The "ansi" I/O mechanism was removed at Revision 0.22.0 in favour of
the MPI/IO mechanism as the default. Serial files written with the
old mechanism can be read by the MPI/IO mechanism.
