
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

The choice of free energy will control automatically a number of factors
related to choice of order parameter, the degree of parallel communication
required, and so on. Each free energy has a number of associated parameters
discussed in the following sections.

Details of general (Newtonian) fluid parameters, such as viscosity,
are discussed in :ref:`Simulation parameters`.



Symmetric Binary Fluid
^^^^^^^^^^^^^^^^^^^^^^

We recall the free energy is a functional of composition :math:`\phi`
and its density may be written:

.. math::

  f(\phi) = {\textstyle \frac{1}{2}} A \phi^2
          + {\textstyle \frac{1}{4}} B \phi^4
          + {\textstyle \frac{1}{2}}\kappa (\partial_\alpha \phi)^2

Parameters are set in the input file via

.. code-block:: none

  # Binary fluid
  free_energy  symmetric
  A            -0.0625                         # Default: -0.003125
  B            +0.0625                         # Default: +0.003125
  K            +0.04                           # Default: +0.002

Common usage has :math:`A < 0` and :math:`B = -A` so that the separated phase
has values :math:`\phi^\star = (-A/B)^{1/2} = \pm 1`. The parameter
:math:`\kappa` (key ``K``) controls the interfacial energy penalty
and is usually positive. The combination of parameters determines
the interfacial width :math:`\xi = (-2\kappa/A)^{1/2}` and the interfacial
tension :math:`\sigma = 4\kappa\phi^{\star 2}/3\xi`.

.. attention::

  Add something on hybrid


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
parameter in the highest derivative `math:`C > 0`.


Polar active gels
^^^^^^^^^^^^^^^^^


The free energy density is a function of vector order parameter 
:math`P_\alpha`:

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
  lc_active_zeta   0.0                         # Default: 0.0

The first is :math:`\xi` (key ``lc_xi``) is the effective molecular
aspect ratio and should be in the range :math:`0 < \xi < 1`. The rotational
diffusion constant is :math:`\Gamma` (key ``lc_Gamma``; not to be
confused with ``lc_gamma``). The (optional) apolar activity
parameter is :math:`\zeta` (key ``lc_active_zeta``).

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






