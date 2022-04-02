
Colloids
--------

.. contents:: In this section
   :depth: 2
   :local:
   :backlinks: none

Introducing colloids
^^^^^^^^^^^^^^^^^^^^

If no relevant key words are present, no colloids will be
expected at run time. The simulation will progress in the
usual fashion with fluid only.

If colloids are required, the ``colloid_init``
key word must be present to allow the code to determine where
colloid information will come from. The options for the
``colloid_init`` key word are summarised below:

.. code-block:: none

  colloid_init             none
  
  #  none                  no colloids [DEFAULT]
  #  input_one             one colloid from input file
  #  input_two             two colloids from input file
  #  input_three           three colloids from input file
  #  input_random          Small number at random
  #  from_file             Read a separate file (including all restarts)

For idealised simulations which require 1, 2, or 3 colloids, relevant
initial state information 
for each particle can be included in the input file. In principle, most
of the colloid state as defined in the colloid
state structure in ``colloid.h`` may be specified. (Some state is
reserved for internal use only and cannot be set from the input file.)
Furthermore, not all the state is relevant in all simulations ---
quantities such as charge and wetting parameters may not be required,
in which case they a simply ignored.

A minimal initialisation is shown in the following example:

.. code-block:: none

  colloid_init              input_one
  
  colloid_one_a0            1.25
  colloid_one_ah            1.25
  colloid_one_r             12.0_12.0_12.0

This initialises a single colloid with an input radius :math:`a_0=1.25`,
and a hydrodynamic radius :math:`a_h=1.25`; in general both are required,
but they do not have to be equal.
A valid position is required within the extent of the system
:math:`0.5 < x,y,z < L + 0.5` as specified by the ``size`` key word.
State which is not explicitly defined is initialised to zero.


Single colloid initialisation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A full list of colloid state-related key words is as follows. All
the quantities are floating point numbers unless explicitly stated
to be otherwise:

.. code-block:: none

  # colloid_one_nbonds      (integer) number of bonds
  #   colloid_one_bond1     (integer) index of bond partner 1
  #   colloid_one_bond2     (integer) index of bond partner 2
  # colloid_one_isfixedr    colloid has fixed position (integer 0 or 1)
  # colloid_one_isfixedv    colloid has fixed velocity (integer 0 or 1)
  # colloid_one_isfixedw    colloid has fixed angular velocity (0 or 1)
  # colloid_one_isfixeds    colloid has fixed spin (0 or 1)
  # colloid_one_type        ``default'' COLLOID_TYPE_DEFAULT
  #                         ``active''  COLLOID_TYPE_ACTIVE
  #                         ``subgrid'' COLLOID_TYPE_SUBGRID
  # colloid_one_a0          input radius
  # colloid_one_ah          hydrodynamic radius
  # colloid_one_al          subgrid offset parameter (subgrid only)
  # colloid_one_r           position vector
  # colloid_one_v           velocity (vector)
  # colloid_one_w           angular velocity (vector)
  # colloid_one_s           spin (unit vector)
  # colloid_one_m           direction of motion (unit) vector for swimmers 

Note that for magnetic particles, the appropriate initialisation involves
the spin key word ``colloid_one_s`` which relates to the dipole
moment :math:`\mu \mathbf{s}_i`, while ``colloid_one_m`` relates to the
direction of motion vector. Do not confuse the two.
It is possible in principle to have magnetic active particles,
in which case the dipole direction or spin (:math:`\mathbf{s}_i`) and the
direction of swimming motion :math:`\mathbf{m}_i` are allowed to be distinct. 

.. code-block:: none

  # colloid_one_b1          Squirmer parameter B_1
  # colloid_one_b2          Squirmer parameter B_2
  # colloid_one_rng         (integer) random number generator state
  # colloid_one_q0          charge (charge species 0)
  # colloid_one_q1          charge (charge species 1)
  # colloid_one_epsilon     Permeativity
  # colloid_one_c           Wetting parameter C
  # colloid_one_h           Wetting parameter H


Example: Single active particle (a squirmer)
""""""""""""""""""""""""""""""""""""""""""""

The following example shows a single active particle with initial
swimming direction along the :math:`x`-axis.

.. code-block:: none

  colloid_init              input_one

  colloid_one_type          active
  colloid_one_a0            7.25
  colloid_one_ah            7.25
  colloid_one_r             32.0_32.0_32.0
  colloid_one_v             0.0_0.0_0.0
  colloid_one_m             1.0_0.0_0.0
  colloid_one_b1            0.05
  colloid_one_b2            0.05

Fixed position or velocity
""""""""""""""""""""""""""

It is possible to fix the position or velocity of a colloid via

.. code-block:: none

  colloid_one_isfixedr       1
  colloid_one_isfixedv       1

It is also possible to do this on a per co-ordinate direction basis
using

.. code-block:: none

  colloid_one_isfixedrxyz    0_0_1
  colloid_one_isfixedvxyz    0_0_1

to, for example, fix the :math:`z`-poisition and velocity components only.

Many colloid initialisation at random
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For suspensions with more than few colloids, but still at
relatively low volume fraction (10--20% by volume), it is
possible to request initialisation at random positions.

The additional key word value pair ``colloid_random_no``
determines the total number of particles to be placed in
the system. To prevent particles being initialised very
close together, which can cause problems in the first few
time steps if strong potential interactions are present,
a ``grace'' distance or minimum surface-surface separation
may also be specified (``colloid_random_dh``).

The following example asks for 100 colloids to be initialised
at random positions, with a minimum separation of 0.5 lattice
spacing.

.. code-block:: none

  colloid_init              input_random

  colloid_random_no         100             # Total number of colloids
  colloid_random_dh         0.5             # ``Grace'' distance

  colloid_random_a0         2.30
  colloid_random_ah         2.40

An input radius and hydrodynamic radius must be provided: these
are the same for all colloids.
If specific initialisations of the colloid state (excepting the
position) other than the radii are wanted, values should be provided
as for the single particle case in the preceding section, but using
``colloid_random_a0`` in place of ``colloid_one_a0`` and so on.

The code will try to initialise the requested number in the current
system size, but only makes a finite number of attempts to place
particles at random with no overlaps. (The initialisation will also
take into account the presence of any solid walls, using the same
grace distance.) If the the number of particles is too large, the
code will halt with a message to that effect.

In general, colloid information for a arbitrary configuration with many
particles should be read from a pre-prepared file. See the section on
File I/O for further information on reading files.

.. attention::

  Section on File I/O required

Colloid interactions
^^^^^^^^^^^^^^^^^^^^

Note that two-body pair-potential interactions are defined uniformly for
all colloids in a simulation. The same is true for lubrication corrections.
There are a number of constraints related to the computation of
interactions discussed below.

Boundary-colloid lubrication correction
"""""""""""""""""""""""""""""""""""""""

Lubrication corrections (here the normal force) between a flat wall
are required to prevent overlap between colloid  and the wall.
The cutoff distance is set via the key word value pair

.. code-block:: none

  boundary_lubrication_rcnormal    0.5

It is recommended that this is used in all cases involving walls.
A reasonable value is in the range :math:`0.1 < r_c < 0.5` in lattice
units, and should be calibrated for particle hydrodynamic radius
and fluid viscosity if exact results are important.

Boundary-colloid soft sphere potential
""""""""""""""""""""""""""""""""""""""

In some circumstances it may be desirable to use a conservative potential
at a boundary wall in place of the lubrication correction. In this case a
cut-and-shifted soft sphere potential is available. Foe example:

.. code-block:: none

  wall_ss_cut_on       yes                    # Switch
  wall_ss_cut_epsilon  0.001                  # Energy scale
  wall_ss_cut_sigma    0.1                    # Length scale
  wall_ss_cut_nu       2.0                    # Exponent
  wall_ss_cut_hc       0.5                    # wall-surface cut off

Both the exponent and the wall-surface cut off should be positive. The
potential will take effect at boundary walls in all directions.


Colloid-colloid lubrication corrections
"""""""""""""""""""""""""""""""""""""""

The key words to activate the calculation of lubrication corrections
are:

.. code-block:: none

  lubrication_on                   1
  lubrication_normal_cutoff        0.5
  lubrication_tangential_cutoff    0.05

Soft-sphere potential
"""""""""""""""""""""

A cut-and-shifted soft-sphere potential of the form
:math:`v \sim \epsilon (\sigma/r)^\nu` is
available. Some trial-and-error with the parameters may be required in
any given situation to ensure simulation stability in the long run. The
following gives an example of the relevant input key words:

.. code-block:: none

  soft_sphere_on            1                 # integer 0/1 for off/on 
  soft_sphere_epsilon       0.0004            # energy units
  soft_sphere_sigma         1.0               # a length
  soft_sphere_nu            1.0               # exponent is positive
  soft_sphere_cutoff        2.25              # a surface-surface separation

Soft-sphere potential (type-specific)
"""""""""""""""""""""""""""""""""""""

This potential is of the same form as the basic cut-and-shifted
soft-sphere potential
described above, but allows different parameters to be specified for
colloids with different *interaction type*. The interaction type is
an integer specifed by the appropriate element of the colloid
structure, e.g., via input

.. code-block:: none

  colloid_one_interact_type   0
  ...
  colloid_two_interact_type   1


specifying two different types (0 and 1). The first type must have
index 0. Interactions between different pairs then all have the form
:math:`v_{ij} \sim \epsilon_{ij} (\sigma_{ij}/r)^{\nu_{ij}}`.

The type specific pair interaction is then introduced via

.. code-block:: none

  pair_ss_cut_ij          yes
  pair_ss_cut_ij_ntypes   2

the second key value pair giving the number of types expected. The parameters
then form a symmetric matrix, for which we specific the upper triangle as
a flattened vector. In the case of two types, there are three independent
parameters, e.g.,

.. code-block:: none

  pair_ss_cut_ij_epsilon  0.2_0.1_0.05  # epsilon_00, _01, _11 in order

where we specify :math:`\epsilon_{00}, \epsilon_{01}` and
:math:`\epsilon_{11}`,
being the interaction energies for interactions bewtween pairs of type
(0,0), (0,1), and (1,1) respectively. The value :math:`\epsilon_{10}` is
set to be the same as :math:`\epsilon_{01}` internally.
A full set of key value pairs might be

.. code-block:: none

  pair_ss_cut_ij          yes           # Switch on
  pair_ss_cut_ij_ntypes   2             # Number of types n
  pair_ss_cut_ij_epsilon  0.0_0.1_0.0   # n(n+1)/2 epsilon parameters
  pair_ss_cut_ij_sigma    0.0_2.0_0.0   # n(n+1)/2 sigma parameters
  pair_ss_cut_ij_nu       1.0_1.0_3.0   # n(n+1)/2 nu exponents
  pair_ss_cut_ij_hc       0.1_0.4_0.1   # n(n+1)/2 surface-surface cut offs

The user must ensure all colloids have appropriate interaction types, i.e.,
the interaction type does not exceed 1 in theis case.


Lennard-Jones potential
"""""""""""""""""""""""

The Lennard-Jones potential is controlled by the following key words:

.. code-block:: none

  lennard_jones_on          1                 # integer 0/1 off/on
  lj_epsilon                0.1               # energy units
  lj_sigma                  2.6               # potential length scale
  lj_cutoff                 8.0               # a centre-centre separation

Yukawa potential
""""""""""""""""

A cut-and-shifted Yukawa potential of the form
:math:`v \sim \epsilon \exp(-\kappa r)/r` is
available using the following key word value pairs:

.. code-block:: none

  yukawa_on                 1                 # integer 0/1 off/on
  yukawa_epsilon            1.330             # energy units
  yukawa_kappa              0.725             # an inverse length
  yukawa_cutoff             16.0              # a centre-centre cutoff

Dipole-dipole interaction and the Ewald sum
"""""""""""""""""""""""""""""""""""""""""""

The Ewald sum is completely specified in the input file
by the uniform dipole strength $\mu$ and the real-space cut off :math:`r_c`.  

.. code-block:: none

  ewald_sum                 1                 # integer 0/1 off/on
  ewald_mu                  0.285             # dipole strength mu
  ewald_rc                  16.0              # real space cut off

If short range interactions are required, particle information is stored
in a cell list, which allows efficient computation of the potentially
:math:`N^2` interactions present. This gives rise to a constraint that the
width of the cells must be large enough that all relevant interactions
are included. This generally means that the cells must be at least
:math:`2a_h + h_c` where :math:`h_c` is the largest relevant cut off
distance.

The requirement for at least two cells per local domain in parallel
means that there is a associated minimum local domain size. This is
computed at run time on the basis of the input. If the local domain
is too small, the code will terminate with an error message. The
local domain size should be increased.

External forces
"""""""""""""""

The following example requests a uniform body force in the negative
:math:`z`-direction on all particles.

.. code-block:: none

  colloid_gravity           0.0_0.0_-0.001    # vector

The counterbalancing body force on the fluid which enforces the
constraint of momentum conservation for the system as a whole is
computed automatically by the code at each time step.

Note: in a real system, a gravitation force on a colloid is
related to buoyancy :math:`F \propto \Delta\rho g`, where the density
difference is that between the colloid and the surrounding fluid,
and :math:`g` is an acceleration.
In a system where there is no density contrast, as we have here
(typically), the "gravity" is the product :math:`\Delta\rho g`. Formally,
this may be viewed as the limit that :math:`\Delta\rho \rightarrow 0`,
combined with the limit :math:`g \rightarrow \infty`, but the limit of
the product is finite.














