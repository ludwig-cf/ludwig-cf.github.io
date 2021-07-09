
Lees Edwards SPBCs
------------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Lees Edwards sliding periodic boundary conditions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Constant uniform shear may be introduced via a number of Lees Edwards
planes with given speed. This is appropriate for fluid-only
systems with periodic boundaries.

.. code-block:: none

  N_LE_plane               2           # Number of planes (default: 0)
  LE_plane_vel             0.05        # Constant plane speed

The placing of the planes in the system is as follows.
The number of planes :math:`N` must
divide evenly the lattice size in the :math:`x`-direction to give an integer
:math:`\delta x`. Planes are then placed at 
:math:`\delta x / 2, 3\delta x/2, \ldots`.
All planes have the same, constant, velocity jump associated with them:
this is positive in the positive :math:`x`-direction. (This jump is usually
referred to as the plane speed.) The uniform shear rate
will be :math:`\dot{\gamma} = N U_{LE} / L_x` where :math:`U_{LE}`
is the plane speed, which is always in the :math:`y`-direction.

The velocity gradient or shear direction is :math:`x`, the flow
direction is :math:`y` and the vorticity direction is :math:`z`.

The spacing between planes must not be less than twice the halo size
in lattice units; 8--16 lattice units may be the practical limit in
many cases. In additional, the speed of the planes must not cause a
violation of the
Mach number constraint in the fluid velocity on the lattice, which
will match the plane speed in magnitude directly either side of the
planes. A value of around 0.05 should be regarded as a maximum safe
limit for practical purposes.

Additional keys associated with the Lees Edwards planes are:

.. code-block:: none

  LE_init_profile          1           # Initialise u(x) (off/on)
  LE_time_offset           10000       # Offset time (default 0)

If ``LE_init_profile`` is set, the fluid velocity is initialised
to reflect a steady state shear flow appropriate for the number of
planes at the given speed (ie., shear rate). If set to zero (the default),
the fluid is initialised with zero velocity.

The code works out the current displacement of the planes by computing
:math:`U_{LE} t`, where :math:`t` is the current time step. A shear run
should then
start from :math:`t = 0`, i.e. zero plane displacement.
It is often convenient to run an equilibration with no shear, and
then to start an experiment after some number of steps. This
key allows you to offset the start of the Lees-Edwards motion.
It should then take the value of the start time (in time steps)
corresponding to the restart at the end of the equilibration
period.

There are a couple of additional constraints to use the Lees-Edwards
planes in parallel. In particular, the planes cannot fall at a
processor boundary in the :math:`x`-direction. This means you should
arrange an integer number of planes per process in the :math:`x`-direction.
(For example, use one plane per process; this will also ensure the number
of planes
still evenly divides the total system size.)
This will interleave the planes with the processor decomposition.
The :math:`y`-direction and :math:`z`-direction may be decomposed without
further constraint.

Note that this means a simulation with one plane will only work
if there is one process in the :math:`x` decomposition.










