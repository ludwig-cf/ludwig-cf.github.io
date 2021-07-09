
Colloid Output
--------------


.. toctree::
   :maxdepth: 1


Colloid file I/O
^^^^^^^^^^^^^^^^

If colloids are present, information can be saved at the required
interval for analysis, and for the purposes of restarting the
calculation.

Two input key/value pairs of interest are:

.. code-block:: none

  colloid_io_freq          1000     # every 1000 time steps 
  colloid_io_format        BINARY   # ASCII or BINARY (default is BINARY)

If necessary, different input and output formats may be specified

.. code-block: none

  colloid_io_format_input  ASCII
  colloid_io_format_output BINARY


Colloid file format
^^^^^^^^^^^^^^^^^^^

The form of the file is as follows.

.. code-block:: none

  <integer>      # 4-byte integer number of colloids in file: 0 or more
  <colloid>
  <colloid>
  ...

The information for each colloid follows the same pattern (whether
ASCII or binary). Not all the information is
relevant in all cases; however, the format is the same in all
cases.

.. code-block:: none

  int index;            /* Unique global index for colloid */
  int rebuild;          /* Rebuild flag */
  int nbonds;           /* Number of bonds e.g. fene (to NBOND_MAX) */
  int nangles;          /* Number of angles, e.g., fene (1 at the moment) */
  
  int isfixedr;         /* Set to 1 for no position update */
  int isfixedv;         /* Set to 1 for no velocity update */
  int isfixedw;         /* Set to 1 for no angular velocity update */
  int isfixeds;         /* Set to zero for no s, m update */
  
  int type;             /* Particle type */
  int bond[2]        ;  /* Bonded neighbours ids (index) */
  
  int rng;              /* Random number state */
  
  int isfixedrxyz[3];   /* Position update in specific coordinate directions */
  int isfixedvxyz[3];   /* Velocity update in specific coordinate directions */
  
  int inter_type;       /* Interaction type of a particle */
  
  int intpad[13];       /* Unused */
  
  double a0;            /* Input radius (lattice units) */
  double ah;            /* Hydrodynamic radius (from calibration) */
  double r[3];          /* Position */
  double v[3];          /* Velocity */
  double w[3];          /* Angular velocity omega */
  double s[3];          /* Magnetic dipole, or spin */
  double m[3];          /* Current direction of motion vector (squirmer) */
  double b1;            /* squirmer active parameter b1 */
  double b2;            /* squirmer active parameter b2 */
  double c;             /* Wetting free energy parameter C */
  double h;             /* Wetting free energy parameter H */
  double dr[3];         /* r update (pending refactor of move/build process) */
  double deltaphi;      /* order parameter bbl net; required to restart */
  
  double q0;            /* magnitude charge 0 */
  double q1;            /* magnitude charge 1 */
  double epsilon;       /* permittivity */
  
  double deltaq0;       /* surplus/deficit of charge 0 at change of shape */
  double deltaq1;       /* surplus/deficit of charge 1 at change of shape */
  double sa;            /* surface area (finite difference) */
  double saf;           /* surface area to fluid (finite difference grid) */
  
  double al;            /* Offset parameter used for subgrid particles */
  double dpad[15];      /* Unused */


Note that the bare colloid output files may be converted to different
format (csv) with a subset of useful information if required. See
``util/extract_colloid.c``.


Colloid parallel output
^^^^^^^^^^^^^^^^^^^^^^^

For very large systems, it may be necessary to use the parallel output
facility to prevent performance and/or memory bottlenecks. The parallel
output writes a number of different files (all of the format discussed
above), based on a decomposition of the domain.

.. code-block:: none

  colloid_io_grid      2_2_2    # default is 1_1_1

This I/O grid will produce 8 files based on a Cartesian decomposition
of the system. Each file may contain different numbers of colloids
depending on the current distribution in space.

The ``extract_colloid`` utility may be used to reconstitute such a
set into a single file if required.

Such a set of files may be used as a restart providing the I/O is
the same.


Colloid initialisation from a single file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Independently of any choice of I/O grid, colloid input (e.g., initial
conditions) may be read from a single file via either

.. code-block:: none

  colloid_io_format_input     ASCII_SERIAL
  colloid_io_format_input     BINARY_SERIAL

