
Diagnostic Output
-----------------



.. toctree::
   :maxdepth: 1


Global quantities
^^^^^^^^^^^^^^^^^

To provide information on the progress of execution, a number of fluid
quanities are summarised to standard output at a frequency determined
by

.. code-block:: none

  freq_statistics    1000

Here the frequency would be once every 1000 time steps. Note that these
global statistics require collective communication and can be relatively
expensive to compute (compared with a single time step).

The details of the exact global statistics produced depends upon the
choice of free energy, and the presence of other features in the
particular simulation. All output is reported in lattice units.

Diagnostic Examples
^^^^^^^^^^^^^^^^^^^

Fluid-only system
~~~~~~~~~~~~~~~~~

For fluid only problems, the global sum, mean and variance, minimum and
maximum of the scalar density field are reported. For free energy cases,
corresponding order parameter statistics are included,
e.g. for a binary fluid
case with compositional order :math:`\phi`, we might have:

.. code-block:: none

  Scalars - total mean variance min max
  [rho]      262144.00  1.00000000000  1.5449642e-11  0.99998006808  1.00001625877
  [phi]  3.1484764e+00  1.2010484e-05  3.7820523e-04 -4.7270149e-02  4.6821679e-02

Here the total density reported against ``[rho]`` should be :math:`\rho_0` per
fluid site (the total
fluid volume for the system in the case that :math:`\rho_0` is unity). Maximum
and minimum density should alert the user to any signifigant deviation from
the Mach number constraint which means that the density should be close to
:math:`\rho_0` throughout the fluid. The same
statistics are reported for compositional order ``[phi]``. Net composition
should not change for a conserved order parameter, so the total reported
for ``[phi]`` should be fixed after initialisation. The maximumm and
minimum order parameter may extend slightly beyond the equilibrium values
:math:`\pm \phi^\star`.

The total free energy is reported:

.. code-block:: none

  Free energy density - timestep total fluid
  [fed]             10 -9.7510518349e-07 -9.7510518349e-07

The exact format of the report again depends on the choice of free energy
and simulation details, but in this example gives the total integrated over
all fluid sites in a fluid only system (the fluid contribution and the
total being identical).


The total fluid momentum is reported in each coordinate direction:

.. code-block:: none

  Momentum - x y z
  [total   ]  3.6427701e-12 -1.5761698e-14 -4.5102810e-15
  [fluid   ]  3.6427701e-12 -1.5761698e-14 -4.5102810e-15

If initialised with a fluid at rest, the total momentum should be zero
(to within machine precision of around :math:`10^{-16}`). In the absence
of external forces, the total momentum should be conserved. However, in
practice, the figures reported will drift owing to the accumulation of
round offs errors (as seen in the above example). However, these errors
should remain small compared with unity.