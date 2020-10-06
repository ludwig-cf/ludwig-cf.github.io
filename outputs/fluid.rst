
Fluid Output
------------



.. toctree::
   :maxdepth: 1


File I/O
^^^^^^^^

Lattice quantities describing the fluid can be saved to file at regular
intervals. This serves two purposes: (1) it allows fluid quantities to
be analysed or visualised and (2) a complete set of fluid quantities
allows the computation to be restarted (checkpoint/restart).

Types of file I/O
~~~~~~~~~~~~~~~~~

The following types of output may be requested at regular intervals

.. code-block:: none

  freq_rho         ! fluid density (scalar)
  freq_vel         ! fluid velocity (vector)
  freq_phi         ! order parameter (scalar, vector or tensor)
  freq_fed         ! fluid free energy density
  freq_config      ! Full configuration output for restart

Full configuration output includes the lattice Boltzmann distributions,
from which all fluid properties (density, velocity, stress) can be
recovered.

Meta data and data
~~~~~~~~~~~~~~~~~~

Requests for file output will first create a relevant `meta-data` file
for the type of output requested. For example, a request for order
parameter output will generate a single file

.. code-block:: none

  phi.001-001.meta

which contains information on the system size, order parameter fields
and so on. The meta-data file provides a description of the corresponding
data and is useful for post-processing.

Actual data appear in a separate file for each time step (at the
frequency requested), e.g.,

.. code-block:: none

  phi-00000000.001-001
  phi-00020000.001-001  

would be expected for a simulation starting at time step zero, and
producing output at each 20,000 steps. The signficance of the file
extension ``.001-001`` is discussed under parallel output below.

By default, output per time step occurs to a single file with extension
``.001-001`` as seen above.



File Formats
^^^^^^^^^^^^

Data format
~~~~~~~~~~~

Output can be requested in either ASCII or (raw) binary format. While
ASCII output can be appropriate for initial investigations, it is
recommended that binary format is used as it is both significantly
faster and results in smaller files. Binary is therefore the default.

The format is controlled via input keys of the form

.. code-block:: none

  rho_io_format            ASCII
  phi_io_format            BINARY
  distribution_io_format   BINARY

To avoid specifying many individual formats when a number of quantities
are requested

.. code-block:: none

  default_io_format        BINARY

may be requested.


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

Parallel Output
^^^^^^^^^^^^^^^

In parallel, MPI tasks write local domains of data to file in turn.
This means the order of the lattice sites in the file may not be
the same is if the data were written in serial. The order of the
writes per processor is detailed in the met-data file.

To rearrange the data so that the correct order is recovered, the
`extract` utility must be used:

.. code-block:: none

  $ ./extract meta-data-file data-file

For example:

.. code-block:: none

  $ ./extract phi.001-001.meta phi-00020000.001-001

will result in a single reordered file ``phi-00020000`` (without file
extension).
The single output file will be in the correct order (as if written in
serial) for analysis.

This will also 'unroll' any displacement associated with Lees Edwards
sliding periodic boundary conditions for the given time step.

Note that in the case the the processor decomposition is in the
:math:`x-` direction only, the order of the output happens to be
correct, and the extract step in not required (unless Less Edwards
'unrolling' is required).


Multiple file output
~~~~~~~~~~~~~~~~~~~~

For large systems, the requirement that each MPI task write data in turn
is unacceptable. Parallel output should be used so that different groups
of MPI tasks write to different files, avoiding serialisation.

This is done by specifying an I/O grid, which decomposes the system in
a similar way to the processor decomposition (with one or more MPI tasks
per I/O grid group).

The I/O grid is set via

.. code-block:: none

  default_io_grid 2_2_1

which would result is four I/O groups writing to four separate files, e.g.,

.. code-block:: none

  phi-00020000.001-004
  phi-00020000.002-004
  phi-00030000.003-004
  phi-00040000.004-004

with corresponding met-data files.



Special case: single file output in parallel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



Examples
^^^^^^^^

