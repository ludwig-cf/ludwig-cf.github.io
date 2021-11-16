
Open boundaries
---------------

.. contents:: This section contains some details of open boundary conditions
	      available to drive channel-like flows.
   :depth: 2
   :local:
   :backlinks: none

Open boundary conditions and walls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Open boundary conditions may be used with a number of provisos:

1. An inflow must be have a corresponding outflow, and vice-versa;
2. The non-flow directions should have plane walls;
3. We assume that the inflow is at the lower, or 'left-hand', edge
   of the domain, and the outflow is at the upper, or 'right-hand' end;
4. All three dimensions must be non-periodic.

A suitable set of input keys would therefore be, e.g.:

.. code-block:: none

  boundary_walls       0_1_1
  periodicity          0_0_0
  lb_bc_open           yes

The switch ``lb_bc_open`` introduces open boundary conditions of a type
which is specified by further keys. It must be present for open
boundaries to be recognised.


Hydrodynamic open boundaries
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Hydrodynamic boundary conditions mean specifying appropriate density
and velocities boundary conditions so that appropriate incoming
distributions can be computed at both inflow and outflow. Outgoing
distributions are lost.

In the following, we assume that the flow direction is the
:math:`x`-direction. Inflow conditions are indicated by
:math:`x=0`, and outflow by :math:`x=L_x`.
Gernealisations to the
:math:`y`-direction or :math:`z`-direction may be made is
a straightforward manner.

      
Type ``rhou`` inflow
""""""""""""""""""""

The following section:

.. code-block:: none

  lb_bc_inflow_type          rhou
  lb_bc_inflow_profile       uniform
  lb_bc_inflow_rhou_u0       0.001_0.0_0.0

will request an inflow condition which imposes a uniform velocity
:math:`u(0,y,z) = u_0`. The density is
:math:`\rho(0,y,z) = \rho(1,y,z)`. The incoming distributions
are then computed as :math:`f_i = f^{eq}_i (\rho, u)`. The velocity
specifed at the inflow should be in the positive :math:`x`-direction.


Type ``rhou`` outflow
"""""""""""""""""""""

Correspinding outflow conditions are introduced via, e.g.:

.. code-block:: none

  lb_bc_outflow_type         rhou
  lb_bc_outflow_rhou_rho0    1.0

Here the outflow density :math:`\rho(L_x,y,z) = \rho_0`, and the
velocity :math:`u(L_x,y,z) = u(L_x-1,y,z)`; incoming distributions
are again computed via :math:`f_i = f^{eq}_i(\rho_0, u)`.

  
Composition and the Cahn-Hilliard equation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Pending.
