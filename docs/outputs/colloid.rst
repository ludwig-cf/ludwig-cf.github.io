
Colloid Output
--------------


.. toctree::
   :maxdepth: 1


Colloid file I/O
^^^^^^^^^^^^^^^^

If colloids are present, information can be saved at the required
interval for analysis, and for the purposes of restarting the
calculation.

Colloid output frequency is controlled by the input key/value

.. code-block:: none

  colloid_io_freq          1000     # every 1000 time steps

There are two modes of output: `ansi` and `mpiio`. The former is an older
method which produces decomposition-dependent files. The MPI/IO method is
preferred, and should be selected via

.. code-block: none

  colloid_io_options_mode      mpiio    # note two `i`s in mpiio

For backwards compatibility, the default mode is `ansi`, which will
ensure older colloid files (produced by version before 0.24.0) can
be read correctly.

For each mode, for record formats are available: `ascii` and `binary`.
The default is `ascii`. The record format can be set using, e.g.,

.. code-block: none

  colloid_io_options_format    binary


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

  int type;             /* Particle type NO LONGER USED; see "shape" etc */
  int bond[2]        ;  /* Bonded neighbours ids (index) */

  int rng;              /* Random number state */

  int isfixedrxyz[3];   /* Position update in specific coordinate directions */
  int isfixedvxyz[3];   /* Velocity update in specific coordinate directions */

  int inter_type;       /* Interaction type of a particle */

  int ioversion;        /* For internal use */
  int bc;               /* Broadly, boundary condition (bbl, subgrid) */
  int shape;            /* Particle shape (2d disk, sphere, ellipsoid) */
  int active;           /* Particle is active */
  int magnetic;         /* Particle is magnetic */
  int attr;             /* Additional attributes bitmask */

  int intpad[7];        /* Unused */

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

                        /* Ellipsoids */
  double elabc[3];      /* Semi principal axes a,b,c */
  double quat[4];       /* Quaternion describing current orientation */
  double quatold[4];    /* Quaternion at previous time step */

  double dpad[4];       /* Unused */


Note that the bare colloid output files may be converted to different
format (csv) with a subset of useful information if required. See
``util/extract_colloid.c``.

Formats for versions before 0.21.0
""""""""""""""""""""""""""""""""""

Version 0.21.0 introduced ellipsoids, which required the replacement
of the ``type`` entry in the colloid structure by separate entries to
describe different properties.

However, it should still be possible to read older colloid state files
as the old ``type`` entry can be translated to the new structure in
most cases. This is done automatically when the file is read at run
time. Ellipsoids must follow the new structure.

Support for files generated before version 0.21.0 will be removed at
the next release (0.25.0).

Colloid parallel output
^^^^^^^^^^^^^^^^^^^^^^^

For very large systems, it may be necessary to use the parallel output
facility to prevent performance and/or memory bottlenecks. Use

.. code-block: none

  colloid_io_options_mode      mpiio
  colloid_io_options_format    binary

This always produces a single file which is decomposition-independent.
