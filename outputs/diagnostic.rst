
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
for ``[phi]`` should be fixed after initialisation (see further
comments below). The maximumm and
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

The total momentum from the fluid is computed as a compensated sum, so
should be robust to parallel decomposition. Other contributions to the
total momumtum (e.g., from colloids) are not currently treated in the
same way.


Conserved order parameter
~~~~~~~~~~~~~~~~~~~~~~~~~

In cases where composition is locally and globally conserved (as in the
symmretic binary fluid), some care can be taken to ensure the total
reported order parameter (ie., the composition) remains fixed as a function
of time. There
are at least two sources of round-off error that can lead to apparent
drift in the total reported as

.. code-block:: none

  Scalars - total mean variance min max
  [rho] ...
  [phi]  3.1484764e+00  1.2010484e-05  3.7820523e-04 -4.7270149e-02  4.6821679e-02

The first is in the evaluation of the total itself (and particularly the
order in which the sum is accumulated in parallel). This is handled in
all cases via a compensated sum, which should ensure the results are
consistent independent of parallel decomposition (in both MPI/thread sense).
The second is the time evolution via the Cahn-Hilliard equation. Additional
measures can be taken if required here via the option:

.. code-block:: none

  cahn_hilliard_options_conserve  1   # integer value 0, 1, or 2

The choices are 0 (the default) which means no special action; 1, which
indroduces an additional compensation at each point of the lattice in
the Cahn Hilliard update; and 2, which introduces a post-hoc correction
to maintain the initial value of the total order parameter at each
time step. These options may be of interest if strict conservation at
machine precision is wanted. This may be most noticable if the total
is exactly zero or very close to zero. For most purposes, the default
should be acceptable.
