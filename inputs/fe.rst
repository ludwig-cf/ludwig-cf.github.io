
The free energy
---------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Available free energies
^^^^^^^^^^^^^^^^^^^^^^^

The choice of free energy is determined as follows:

.. code-block:: none

  free_energy   none

The default value is ``none``, i.e., a simple Newtonian fluid is used.
Possible values of the ``free_energy`` key are:

.. code-block:: none

  #  none                     Newtonian fluid [DEFAULT]
  #  symmetric                Binary fluid (finite difference)
  #  symmetric_lb             Binary fluid (two distributions)
  #  brazovskii               Brazovskii smectics
  #  polar_active             Polar active gels
  #  lc_blue_phase            Liquid crystal (nematics, ...)
  #  lc_droplet               Liquid crystal emulsions
  #  fe_electro               Single fluid electrokinetics
  #  fe_electro_symmetric     Binary fluid electrokinetics
  #  ternary                  Three-component fluid

The choice of free energy will control automatically a number of factors
related to choice of order parameter, the degree of parallel communication
required, and so on. Each free energy has a number of associated parameters
discussed in the following sections.

Details of general (Newtonian) fluid parameters, such as viscosity,
are discussed in :ref:`Simulation parameters`.



Symmetric binary fluid
^^^^^^^^^^^^^^^^^^^^^^

We recall the free energy is a functional of composition :math:`\phi`
and its density may be written:

.. math::

  f(\phi) = {\textstyle \frac{1}{2}} A \phi^2
          + {\textstyle \frac{1}{4}} B \phi^4
          + {\textstyle \frac{1}{2}}\kappa (\partial_\alpha \phi)^2.

Parameters are set in the input file via

.. code-block:: none

  # Binary fluid
  free_energy            symmetric         # Use free energy
  symmetric_a            -0.0625           # Bulk term          [required]
  symmetric_b            +0.0625           # Bulk term          [required]
  symmetric_kappa        +0.04             # Interfacial term   [required]
  symmetric_c             0.00             # Surface term       [optional]
  symmetric_h             0.00             # Surface term       [optional]

Common usage has :math:`A < 0` and :math:`B = -A` so that the separated phase
has values :math:`\phi^\star = (-A/B)^{1/2} = \pm 1`. The parameter
:math:`\kappa` (key ``K``) controls the interfacial energy penalty
and is usually positive. The combination of parameters determines
the interfacial width :math:`\xi = (-2\kappa/A)^{1/2}` and the interfacial
tension :math:`\sigma = 4\kappa\phi^{\star 2}/3\xi`.

The surface terms are discussed further in :doc:`walls`.

In this approach, the fluid is treated using lattice Boltzmann, while the
order parameter evolves according to a Cahn-Hilliard equation treated
numerically via finite difference. For historical interest, the symmetric
free energy problem can also be treated using two lattice Boltzmann
distributions:

.. code-block:: none

  free_energy   symmetric_lb

Other parameters have the same meaning. This approach was used in an
earlier implementation, is and
discussed at some length in work including [Kendon2001]_.

.. [Kendon2001] V.M. Kendon, M.E. Cates, I. Pagonabarraga, J.-C. Desplat,
                and P. Bladon,
                Inertial effects in three-dimensional spinodal decomposition
                of a symmetric binary fluid mixture: a lattice Boltzmann study,
                *J. Fluid Mech.*, **440** 147-203 (2001).


Brazovskii smectics
^^^^^^^^^^^^^^^^^^^


The free energy density is a function of the composition

.. math::

  f(\phi) = {\textstyle \frac{1}{2}} A \phi^2
          + {\textstyle \frac{1}{4}} B \phi^4
          + {\textstyle \frac{1}{2}} \kappa (\partial_\alpha \phi)^2
          + {\textstyle \frac{1}{2}} C (\partial_\alpha^2 \phi)^2


Parameters are introduced via the keys

.. code-block:: none

  free_energy  brazovskii
  A             -0.0005                        # Default: 0.0
  B             +0.0005                        # Default: 0.0
  K             -0.0006                        # Default: 0.0
  C             +0.00076                       # Default: 0.0


For :math:`A < 0`, phase separation occurs with a result depending on
:math:`\kappa`:
one gets two symmetric phases for :math:`\kappa >0` (cf. the symmetric case)
or a lamellar phase for :math:`\kappa < 0`. Typically, :math:`B = -A` and the
parameter in the highest derivative :math:`C > 0`.


Polar active gels
^^^^^^^^^^^^^^^^^


The free energy density is a function of vector order parameter 
:math:`P_\alpha`:

.. math::

  f(P_\alpha) = {\textstyle \frac{1}{2}} A P_\alpha P_\alpha
              + {\textstyle \frac{1}{4}} B (P_\alpha P_\alpha)^2
              + {\textstyle \frac{1}{2}} \kappa (\partial_\alpha P_\beta)^2

There are no default parameters:

.. code-block:: none

  free_energy        polar_active
  polar_active_a    -0.1                       # Default: 0.0
  polar_active_b    +0.1                       # Default: 0.0
  polar_active_k     0.01                      # Default: 0.0

It is usual to choose :math:`B > 0`, in which case :math:`A > 0` gives
an isotropic phase, whereas :math:`A < 0` gives a polar nematic phase.
The elastic constant :math:`\kappa` (key ``polar_active_k``)
is positive.


Liquid crystals
^^^^^^^^^^^^^^^


The free energy density is a function of tensor order parameter
:math:`Q_{\alpha\beta}`:

.. math::

  f(Q_{\alpha\beta}) =
  {\textstyle\frac{1}{2}} A_0 (1 - \gamma/3)Q^2_{\alpha\beta} -
  {\textstyle\frac{1}{3}} A_0 \gamma
  Q_{\alpha\beta}Q_{\beta\delta}Q_{\delta\alpha}
  + {\textstyle\frac{1}{4}} A_0 \gamma (Q^2_{\alpha\beta})^2
  + {\textstyle\frac{1}{2}} \big(
  \kappa_0 (\epsilon_{\alpha\delta\sigma} \partial_\delta Q_{\sigma\beta} +
  2q_0 Q_{\alpha\beta})^2 + \kappa_1(\partial_\alpha Q_{\alpha\beta})^2 \big)

The corresponding ``free_energy`` value, despite its name, is
suitable for nematics and cholesterics, and not just blue phases:

.. code-block:: none

  free_energy      lc_blue_phase
  lc_a0            0.01                       # Deafult: 0.0
  lc_gamma         3.0                        # Default: 0.0
  lc_q0            0.19635                    # Default: 0.0
  lc_kappa0        0.00648456                 # Default: 0.0
  lc_kappa1        0.00648456                 # Default: 0.0

The bulk free energy parameter :math:`A_0` is positive and controls the
energy scale (key ``lc_a0``); :math:`\gamma` is positive and
influences the position in the phase diagram relative to the
isotropic/nematic transition (key ``lc_gamma``).
The two elastic constants must be equal, i.e., we enforce the
single elastic constant approximation (both keys ``lc_kappa0`` and
``lc_kappa1`` must be specified).

Other important parameters in the liquid crystal picture are:

.. code-block:: none

  lc_xi            0.7                         # Default: 0.0
  lc_Gamma         0.5                         # Default: 0.0

The first is :math:`\xi` (key ``lc_xi``) is the effective molecular
aspect ratio and should be in the range :math:`0 < \xi < 1`. The rotational
diffusion constant is :math:`\Gamma` (key ``lc_Gamma``; not to be
confused with ``lc_gamma``).

Liquid crystal activity
"""""""""""""""""""""""

There exists an option to model contractile or extensile active fluids
via the addition of an "active" stress in the
liquid crystal free energy. For historical reasons, this is
additional active stress written as

.. math::
  
  S_{\alpha\beta} = \zeta_0 \delta_{\alpha\beta} - \zeta_1 Q_{\alpha\beta} 
                  - \zeta_2 (\partial_\beta P_\alpha + \partial_\alpha P_\beta)

where :math:`P_\alpha = Q_{\alpha\gamma} \partial_\beta Q_{\beta\gamma}`.
The first term in :math:`\zeta_0` is included for completeness: it
should only influence compressability and one may safely leave
:math:`\zeta_0  = 0`. The second term models active force dipoles and
sets the force density (:math:`\zeta_1 < 0` for a contractile fluid,
and :math:`\zeta_1 > 0` for an extensile fluid).
The third term in :math:`\zeta_2` is experimental. 

The relevant input keys and values are, e.g.,

.. code-block:: none

  lc_activity      yes                         # Required for activity
  lc_active_zeta0  0.0                         # Default: 0.0
  lc_active_zeta1  0.001                       # Default: 0.0
  lc_active_zeta2  0.0                         # Default: 0.0



Liquid crystal anchoring
""""""""""""""""""""""""

Different types of anchoring are available at solid surfaces, with
one or two related free energy parameters depending on the type.
The type of anchoring may be set independently for stationary
boundaries (walls) and colloids.

.. code-block:: none

  lc_anchoring_strength     0.01               # free energy parameter w1
  lc_anchoring_strength_2   0.0                # free energy parameter w2
  lc_wall_anchoring         normal             # ``normal'' or ``planar''
  lc_coll_anchoring         normal             # ``normal'' or ``planar''


Liquid crystal emulsion
^^^^^^^^^^^^^^^^^^^^^^^

This an interaction free energy which combines the symmetric and liquid
crystal free energies. The liquid crystal free energy constant
:math:`\gamma` becomes a function of composition via
:math:`\gamma(\phi) = \gamma_0 + \delta(1 + \phi)`.
Typically, we might choose :math:`\gamma_0` and :math:`\delta` so that
:math:`\gamma(-\phi^\star) < 2.7` and the :math:`-\phi^\star` phase is
isotropic, while :math:`\gamma(+\phi^\star) > 2.7` and the
:math:`+\phi^\star` phase is ordered (nematic, cholesteric, or blue phase).
Experience suggests that a suitable choice is :math:`\gamma_0 = 2.5` and
:math:`\delta = 0.25`.

A coupling term is added to the free energy density:

.. math::

  W Q_{\alpha\beta} \partial_\alpha \phi \partial_\beta \phi.

For anchoring constant :math:`W > 0`, the liquid crystal anchoring at the
interface is planar, while for :math:`W < 0` the anchoring is normal. This
is set via key ``lc_droplet_W``.

Relevant keys (with default values) are:

.. code-block:: none

  free_energy            lc_droplet
  
  A                      -0.0625
  B                      +0.0625
  K                      +0.053
  
  lc_a0                   0.1
  lc_q0                   0.19635
  lc_kappa0               0.007
  lc_kappa1               0.007
  
  lc_droplet_gamma        2.586                # Default: 0.0
  lc_droplet_delta        0.25                 # Default: 0.0
  lc_droplet_W           -0.05                 # Default: 0.0

Note that key ``lc_gamma`` is not used in this case.

Liquid crystal emulsion activity
""""""""""""""""""""""""""""""""

An option for an additional active stress in the case of an emulsion
is present. The form of the stress allows for activity in the
ordered phase (:math:`\phi = +1`):

.. math::

  S_{\alpha\beta} = {\textstyle\frac{1}{2}} (1 + \phi)
                    [ \zeta_0 \delta_{\alpha\beta} - \zeta_1 Q_{\alpha\beta} ].

The meaning of the active terms is the same as for the bare (active)
liquid crystal case.

The relavant input keys are:

.. code-block:: none

  lc_droplet_active_zeta0    0.0       # Default 0.0
  lc_droplet_active_zeta1    0.001     # Default 0.0

Note that these are separate from the bare liquid crystal activity
parameters (which are not used at the same time).


Ternary free energy
^^^^^^^^^^^^^^^^^^^

An implementation of the ternary model following [Semprebon]_ is
available. This uses a lattice Boltzmann density :math:`\rho` coupled to
two scalar order parameters :math:`\phi` and :math:`\psi` to give three
components. The two scalar order parameters each evolve via a Cahn-Hilliard
equation treated by finite difference.

The basic free energy parameters are:

.. code-block:: none

  free_energy               ternary            # Select ternary free energy
  
  ternary_kappa1            0.01               # Interfacial parameter > 0
  ternary_kappa2            0.02               # Interfacial parameter > 0
  ternary_kappa3            0.05               # Interfacial parameter > 0
  ternary_alpha             1.00               # Interfical width
  
  ternary_mobility_phi      0.15               # Mobility for phi
  ternary_mobility_psi      0.10               # Mobility for psi

All the parameters must be specified.

As the description is rather involved, we do not repeat it here.

.. [Semprebon] C. Semprebon, T. Krueger, and H. Kusumaatmaja,
               Ternary free-energy lattice Boltzmann model with tunable
               contact angles,
               *Phys. Rev. E*, **93** 033305 (2016).
