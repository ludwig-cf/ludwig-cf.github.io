
Simulation parameters
---------------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none


System size and parallel decomposition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


The system size is specified via the key ``size``, e.g.:

.. code-block:: none

  size         64_64_64

If a two-dimensional system is required, the :math:`z`-direction extent
should be set to 1.

In parallel, the domain decomposition is closely related to the
system size, and is specified as follows:

.. code-block:: none

  grid         4_2_1

The ``grid`` key specifies the number of MPI tasks required in
each coordinate direction. In the above example, the decomposition
is into 4 in the :math:`x`-direction, into 2 in the :math:`y`-direction, while
the :math:`z`-direction is not decomposed. In this example, the local domain
size per MPI
task would then be :math:`16\times32\times64`. The total number of MPI tasks
available must match the total implied by ``grid`` (8 in the
example).

If the requested decomposition is not valid, or ``grid`` is
omitted, the code will try to supply a decomposition based on
the number of MPI tasks available and ``MPI_Dims_create()``;
this may be implementation dependent.


Simulation time steps
^^^^^^^^^^^^^^^^^^^^^

Basic parameters controlling the number of time steps are:

.. code-block:: none

  N_start      0                              # Default: 0
  N_cycles     100                            # Default: 0

A typical simulation will start from time zero (key ``N_start``)
and run for a certain number of time steps (key ``N_cycles``).

If a restart from a previous run is required, the choice of parameters
may be as follows:

.. code-block:: none

  N_start      100
  N_cycles     400

This will restart from data previously saved at time step 100, and
run a further 400 cycles, i.e., to time step 500.

Fuild parameters
^^^^^^^^^^^^^^^^

Control parameters for a Newtonian fluid include:

.. code-block:: none

  fluid_rho0                 1.0
  viscosity                  0.166666666666666
  viscosity_bulk             0.166666666666666

The mean fluid density is :math:`\rho_0` (key ``fluid_rho0``) which
defaults to unity in lattice units; it is not usually necessary to
change this. The shear viscosity is
``viscosity`` and as default value 1/6 to correspond to
unit relaxation time in the lattice Boltzmann picture. Reasonable
values of the shear viscosity are :math:`0.2 > \eta > 0.0001` in lattice
units. Higher values move further into the over-relaxation region, and can
result in poor behaviour. Lower
values increase the Reynolds number and tend to cause
problems with stability. The bulk
viscosity has a default value which is equal to whatever shear
viscosity has been selected. Higher values of the bulk viscosity
may be set independently and can help to suppress large deviations
from incompressibility and maintain numerical stability
in certain situations.


Fluctuating hydrodynamics
"""""""""""""""""""""""""

If fluctuating hydrodynamics is wanted, set the switch
``lb_fluctuations`` to ``on`` or ``yes``. The associated
temperature is in lattice units: reasonable values (at :math:`\rho_0 = 1`)
are :math:`0 < kT < 0.0001`. If the temperature is too high, local
velocities will rapidly exceed the Mach number constraint and
the simulation will be unstable.

.. code-block:: none

  lb_fluctuations            yes        # fluctuations on
  temperature                0.00001    # "kT"

Note this "temperature" is the thermal energy per dimension, so
that :math:`<u_x^2> = kT` etc. Statistics on the equipartition
of energy will be reported if the fluctuations are present, e.g..

.. code-block:: none

  Isothermal fluctuations
  [eqipart.]  9.9477576e-06  9.9684327e-06  9.9476364e-06
  [measd/kT]  2.9863827e-05  3.0000000e-05

The first line gives the values of :math:`<u_x^2>`,
:math:`<u_y^2>`, and :math:`<u_z^2>` averaged over the whole system.
The second line gives the total measured for all three dimensions, and
the expected value (:math:`3kT`) based on the input temperature.
