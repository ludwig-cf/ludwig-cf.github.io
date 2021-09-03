
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
include uniform external forces ("gravity"), or local forces arising
from the thermodynamic sector. The later would include bending forces
at the interface arising from interfacial tension in the case of a
symmetric binary fluid.

Constant body forces
^^^^^^^^^^^^^^^^^^^^

If a constant body force density on the fluid is required, this may be
included via:

.. code-block:: none

  force fx_fy_fz

where the three components are defined in the usual way. This force desnity
is applied per fluid site per time step, so there must be some
counter-balancing mechanism to prevent the fluid accelerating
indefinitely. This counter-balancing force might be provided by,
e.g., drag at external walls.

Local forces from the thermodynamic sector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Local forces on teh fluid often arise from the thermodynamic sector; as
a concrete example we will consider the symmetric free energy with
compositional order parameter :math:`\phi(\mathbf{r};t)` and corresponding
chemical potential :math:`\mu(\mathbf{r};t)`.



Via the diveregence of the stress
"""""""""""""""""""""""""""""""""

The body force density may be computed via the divergence of the
thermodynamic stress:

.. math::

  f_\alpha (\mathbf{r}; t) = \partial_\beta P_{\alpha\beta}

The key in the input file to use this method is

.. code-block:: none

  fd_force_divergence yes


This method has the advantage that momentum is conserved both locally, and
therefore globally, but the disadvantage that an equillibrium is not possible
owing to the generation of residual ("spurious") flows.
Note that this is the default method.

Via the gradient of the chemical potential
""""""""""""""""""""""""""""""""""""""""""

The local force on the fluid may be computed as

.. math::

  f_\alpha (\mathbf{r};t) = - \phi(\mathbf{r};t) \nabla_\alpha \mu(\mathbf{r};t)

To use this method set the switch

.. code-block:: none

  fd_force_divergence no

This method has the advantage that an equilibrium is admittted, but
the disadvantage that momentum is not conserved (it can only be so
via a divergence form).

External chemical potential gradient
""""""""""""""""""""""""""""""""""""

For Cahn-Hilliard dynmaics, it is possible to add a fixed external
chemical potential gradient :math:`\nabla_\alpha^\textrm{ex} \mu`
which generates an additional diffusive flux

.. math::

   \partial_t \phi + \nabla_\alpha \phi u_\alpha =
   -\nabla_\alpha (M\nabla_\alpha \mu + M \nabla_\alpha^\textrm{ex}\mu),

where :math:`M` is the mobility.
The addition chemical potential gradient also leads to a contribution to the
body force on the fluid which
is locally :math:`-\phi \nabla_\alpha^\textrm{ex}\mu`. The external
potential chemical gradient is a vector which may be defined in the
input file as, e.g.,

.. code-block:: none

  grad_mu              0.00001_0.0_0.0   # External chemical potential gradient
  fd_force_divergence  no                # Currently required 

The additional force on the fluid may lead to a permanent acceleration
of the fluid, so it may make sense to use this option only when walls
are present, or some other mechanism is present to limit fluid motion.


Local force via the equilibrium stress
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to introduce an effective body force at the lattice Boltzmann
collision stage by making an ad-hoc addition to the equilibrium stress
:math:`S_{\alpha\beta}^{eq}`:

.. math::

  S_{\alpha\beta}^{eq} = u_\alpha u_\beta + P_{\alpha\beta}

where :math:`P_{\alpha\beta}` is the thermodynamic contribution to the
stress. To do this set the switch

.. code-block:: none

  fe_use_stress_relaxation yes

Note that this may be done when the thermodynamic contribution is symmetric.
If the thermodynamic contribution is anti-symmetric (e.g., for the liquid
crystal case) then only the symmetric part is implememnted in this way.
The contribution to the force is computed as a divergence of the
anti-symmetric stress as above.
