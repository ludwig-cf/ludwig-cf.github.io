
Force on the fluid
------------------

Forces on the fluid fall into a number of categories which are discussed
in this section.

.. toctree::
   :maxdepth: 1


Body forces
^^^^^^^^^^^

The momentum equation may include a term which is a local body force
density. The origin of such a force will depend on context, but may
include uniform external forces, or local forces arising
from the thermodynamic sector. An example of the former would be
gravity, while an example of the latter would include bending forces
at the interface arising from interfacial tension in the case of a
symmetric binary fluid.

Constant body forces
^^^^^^^^^^^^^^^^^^^^

If a constant body force density on the fluid is required, this may be
included via:

.. code-block:: none

  force fx_fy_fz

where the three components are defined in the usual way. This force density
is applied per fluid site per time step, so there must be some
counter-balancing mechanism to prevent the fluid accelerating
indefinitely. This counter-balancing force might be provided by,
e.g., drag at external walls.

Local forces from the thermodynamic sector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. warning::

   As of version 0.19.0 the input key switches ``fd_force_divergence`` and
   ``fe_use_stress_divergence`` have been replaced by those described
   below. If the keys are present in the input, an error will occur;
   the old key value pairs should be replaced with the corresponding
   new single key ``fe_force_method``.

The choice of free energy determines the form of chemical potential
(and analogous quantities for more complex order parameters), and
the form of the thermodynamic stress exerted locally on the fluid.
There are a number of ways to implement how the force is computed
and how it is applied to the fluid.

The list of available keys is as follows. Not all choices are
relevant for all free energy selections. See the selections
below for further details.

.. code-block:: none

  fe_force_method          no_force
  fe_force_method          stress_divergence
  fe_force_method          phi_gradmu
  fe_force_method          phi_gradmu_correction
  fe_force_method          relaxation_symmetric
  fe_force_method          relaxation_antisymmetric


Generic cases
"""""""""""""

All free energies define a "thermodynamic" stress :math:`P_{\alpha\beta}`
which acts on the fluid. There are a number of methods to implement the
resulting force on the fluid. The first is to compute the divergence of
the stress via a finite difference expression, which results in a local
body force density

.. math::

  f_\alpha (\mathbf{r}; t) = \partial_\beta P_{\alpha\beta}.

The local body force density then enters the hydrodynamics at the lattice
Boltzmann collision stage. This method has the advantage that it is
by construction a divergence in the finite difference picture, and
so momentum is conserved both locally, and therefore globally.

A second approach is to add the thermodynamic stress to the equilibrium
stress :math:`S_{\alpha\beta}^{eq}` at the lattice Boltzmann collision
stage:

.. math::

  S_{\alpha\beta}^{eq} = u_\alpha u_\beta + P_{\alpha\beta}.

The current stress is then relaxed toward the equilibrium at a rate
determined by the viscosity, and we refer to this as the relaxation
method. It requires that the thermodynamic contribution to the stress
is symmetric. The relaxation approach results in an effective
divergence of a stress, which again conserves momentum.

These options are enabled by selections ``stress_divergence`` and
``relaxation_symmetric``, respectively.
If the option ``no_force`` is selected, no force from the thermodynamic
sector will enter the hydrodynamics. This differs from setting no
hydrodynamics at all in that with ``no_force``, forces may still enter
the hydrodynamics from elsewhere (e.g., from external gravity).


Scalar order parameters and Cahn Hilliard
"""""""""""""""""""""""""""""""""""""""""

For scalar order parameters :math:`\phi(\mathbf{r}; t)` with Cahn-Hilliard
dynamics such as the composition in the symmetric free energy, it is
possible to relate the local body force on the fluid to the gradient of
the chemical potential :math:`\mu(\mathbf{r};t)`:

.. math::

  f_\alpha (\mathbf{r};t) =
  - \phi(\mathbf{r};t) \partial_\alpha\mu(\mathbf{r};t).

Formally, this expression is the same as the divergence of the corresponding
stress (as seen above). However, in the finite difference picture, their
discrete analogues do not behave in the same way. Specifically, the
gradient of the chemical potential is not a divergence and therefore
does not, by construction, result in a net force of zero and so
does not conserve momentum.

However, the calculation does have the advantage that it admits an
equilibrium. This is related to the fact that a consistent finite
difference approach to the body force, and the diffusive fluxes
in the Cahn-Hilliard equation :math:`-M\partial_\alpha\mu(\mathbf{r};t)`
where :math:`M` is a uniform mobility, can be adopted. In this way,
with a uniform chemical potential both the force in momentum equation
and diffusive fluxes in the Cahn-Hilliard equation are simultaneously
zero. This admits a state
of no flow where the total flux in the Cahn-Hilliard equation

.. math::

  \phi u_\alpha + M \partial_\alpha \mu = 0.

This is in contrast to the computation of the force based
on :math:`\partial_\beta P_{\alpha\beta}`, where a uniform chemical
does not correspond to zero force in the finite difference picture.
This often results in the generation of residual flows
("spurious currents") in situations where an equilibrium might be
expected.

For scalar order parameters one can choose

.. code-block:: none

  fe_force_method          phi_gradmu
  fe_force_method          phi_gradmu_correction

The first option computes and applies the local force
:math:`-\phi(\mathbf{r};t) \partial_\alpha \mu(\mathbf{r};t)`
without additional action.
The second option computes the same force, but then applies a correction
at every time step based on the current global force on the fluid. Any net
force is subtracted uniformly from fluid sites to enforce global
conservation of momentum before the force is applied.

External chemical potential gradient
""""""""""""""""""""""""""""""""""""

For Cahn-Hilliard dynamics, it is also possible to add a fixed external
chemical potential gradient :math:`\partial_\alpha \mu^\textrm{ex}`
which generates an additional diffusive flux

.. math::

   \partial_t \phi + \partial_\alpha \phi u_\alpha =
   -\partial_\alpha (M\partial_\alpha \mu + M \partial_\alpha \mu^\textrm{ex}),

where :math:`M` is the mobility.
The addition chemical potential gradient also leads to a contribution to the
body force on the fluid which
is locally :math:`-\phi \partial_\alpha \mu^\textrm{ex}`. The external
potential chemical gradient is a vector which may be defined in the
input file as, e.g.,

.. code-block:: none

  grad_mu                  0.00001_0.0_0.0   # x-direction only

The additional force on the fluid may lead to a permanent acceleration
of the fluid, so it may make sense to use this option only when walls
are present, or some other mechanism is present to limit fluid motion.

Note this option is independent of the choice of ``fe_force_method``.


Vector order parameter
""""""""""""""""""""""

For vector order parameters involving the Leslie-Erickson equation,
there are currently no options for ``fe_force_method``: the stress
divergence method is the default.


Forces for tensor order parameter
"""""""""""""""""""""""""""""""""

The liquid crystal tensor order parameter :math:`Q_{\alpha\beta}` gives
rise to a stress which may be written in three parts:

.. math::

   P_{\alpha\beta} =
   P^\mathrm{symm}_{\alpha\beta} + P^\mathrm{anti}_{\alpha\beta}
   - \partial_\alpha Q_{\pi\nu}
     \frac{\delta \cal{F}}{\delta \partial_\beta Q_{\pi\nu}}.

Here the first two parts are related to the molecular field
:math:`H_{\alpha\beta}` and are symmetric and antisymmetric
contributions, respectively. The third part, related
to the functional derivative of the free energy density with
respect to the order parameter gradient, is symmetric.
In this case, the stress divergence may be computed to give a body
force. However, as the stress relaxation requires a symmetric stress,
only :math:`P^\mathrm{symm}_{\alpha\beta}` and the third term may
enter. The remaining
antisymmetric must be treated via the divergence approach
:math:`\partial_\beta P^\mathrm{anti}_{\alpha\beta}`.

For the bare liquid crystal free energy there is currently no
implementation to compute the force via a term of the form

.. math::

   f_\gamma(\mathbf{r};t) =
   -H_{\alpha\beta}(\mathbf{r};t)\partial_\gamma Q_{\alpha\beta}(\mathbf{r};t),

which is the expression analogous to that involving the chemical potential
in the scalar order parameter case.

Relevant choices of ``fe_force_method`` for the bare liquid crystal
free energy are therefore:

.. code-block:: none

  fe_force_method          stress_divergence
  fe_force_method          relaxation_antisymmetric

For the liquid crystal emulsion free energy there is both a chemical
potential and a molecular field. Again, the stress from the
thermodynamic sector may be decomposed into three parts

.. math::

   P_{\alpha\beta} = P^\mathrm{symm}_{\alpha\beta}
                   + P^\mathrm{anti}_{\alpha\beta}
		   + P^\mathrm{drop}_{\alpha\beta}

where the symmetric and antisymmetric parts are treated in the same
way as the bare liquid crystal case (the same input keys are
relevant).

The *additional* contribution :math:`P^\mathrm{drop}_{\alpha\beta}`
arising from the functional derivative of the free energy, gives
rise to a force:

.. math::

   f_\alpha = - H_{\pi\nu} \partial_\alpha Q_{\pi\nu}
              - \phi \partial_\alpha \mu.

This is always computed separately and subject to a correction to
conserve momentum.


Forces in the electrokinetic picture
""""""""""""""""""""""""""""""""""""

For free energies involving electrokinetics and the Nernst-Planck
equation, there are two available options:

.. code-block:: none

  fe_force_method           stress_divergence
  fe_force_method           phi_gradmu_correction

The details of the computation in this case a slightly different to
other free energies (the ``phi`` in ``phi_gradmu_correction`` is a
slight misnomer in the context of electrokinetics).


Force near stationary solid objects (walls)
"""""""""""""""""""""""""""""""""""""""""""

The following methods are available in the case of a static wall:

.. code-block:: none

  fe_force_method           no_force
  fe_force_method           stress_divergence
  fe_force_method           phi_gradmu
  fe_force_method           phi_gradmu_correction

In the stress divergence approach, the stress acting on the wall does
is accounted for as momentum lost by the fluid. If using the chemical
potential, the approximation is made that the normal component of
the gradient of the chemical potential at the wall is zero. Either
the ``phi_gradmu`` or the ``phi_gramu_correction`` with no momentum
conservation or with the correction to conserve momentum, respectively.

Force near moving solid objects
"""""""""""""""""""""""""""""""

For moving colloids, the following methods are available:

.. code-block:: none

  fe_force_method           no_force
  fe_force_method           stress_divergence
  fe_force_method           phi_gradmu_correction

For ``no_force`` there will be no force on the colloids from the
thermodynamic sector. In the ``stress_divergence`` case, the net
force on the colloid arising from the divergence of the thermodynamic
stress is computed by accumulating the sum of the stress on the
solid-fluid interfaces that make up the colloid at any given instant.
This gives a body force on each colloid which goes into the dynamic
update of their velocities. This conserves momentum.

In contrast, if the gradient of the chemical potential is used, there
is no direct force on the colloids from the thermodynamic sector. Any
influence comes indirectly through the hydrodynamic boundary conditions
at the colloid surface in response to the local fluid velocities.
