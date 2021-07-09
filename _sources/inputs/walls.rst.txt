
Plane walls
-----------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Plane boundary walls
^^^^^^^^^^^^^^^^^^^^

As a convenience, it is possible to specifiy that sets of plane walls
are present in the system in one or more co-ordinate directions. For
example a set of walls inthe :math:`y-z` plane are placed at each end
of the system in positions
:math:`x = 0.5\Delta x` and :math:`x = L + 0.5\Delta x` if the input
file contains

.. code-block:: none

  boundary_walls_on    yes
  periodicity          0_1_1
  boundary_walls       1_0_0

The extent of the system :math:`L_x` is then just the number of points in
the x-direction as specfied by key ``size``. By default, walls have
no-slip boundary conditions implemented via bounce-back on links.

Walls may be specifed in two dimensions to form a rectangular
"duct", or in all three dimensions to form an enclosed box.

Moving wall
"""""""""""

If one sets of walls (only) is present, is possible to specify
a boundary speed (in the positive :math:`x`-direction) which will impart
momemtum to the fluid. Use, e.g.,

.. code-block:: none

  boundary_walls          1_0_0
  boundary_speed_bottom  -0.001
  boundary_speed_top     +0.001

will set the corresponsing speed of the wall :math:`u_x` at the lower
at upper ends of the system, 
respectively. Note that these speeds should be selected with reference
to the Mach number constraint :math:`u < c_s`. Momentum transfer here
is again implemented via a no-slip condition via bounce-back on links.

Note the wall speed has only one non-zero component :math:`u_x`, which
should be tangential to the place of the wall (i.e., moving walls
should be in either `y` or `'z` dimensions).

Slip and no-slip
""""""""""""""""

It is possible to specify a linear combination of slip and no-slip
conditions on a per-wall basis. The fraction of slip is specified.
E.g.,

.. code-block :: none

  boundary_walls                    0_0_1
  boundary_walls_slip_active        yes
  boundary_walls_slip_fraction_bot  0.0_0.0_0.0
  boundary_walls_slip_fraction_top  0.0_0.0_1.0

gives a no-slip condition at the lower wall and a free-slip condition
at the top wall (walls in the :math:`x-y` plane). Values of the slip
:math:`s` must satisfy :math:`0 \leq s \leq 1` for all six faces. 
If slip is active, no moving walls can be used.


Colloids and plane walls
""""""""""""""""""""""""

Systems with colloids and walls can be accommodated. To prevent colloids
impinging on the plane walls, a lubrication correction can be added by
setting

.. code-block:: none

  boundary_lubrication_rcnormal   0.25

For surface-surface separations below this cut-off value specified in
lattice units, a normal lubrication correction based on the analytical
expression for the lubrication force between a sphere (of the appropriate
hydrodynamic radius) and a plane wall is added to the force on
the colloid.








