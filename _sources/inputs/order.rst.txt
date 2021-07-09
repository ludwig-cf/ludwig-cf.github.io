
Order Parameters
----------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Order parameter initialisations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Composition :math:`\phi`
""""""""""""""""""""""""


The following initialisations are available.

.. code-block:: none

  phi_initialisation          spinodal    # spinodal
  phi0                        0.0         # mean
  noise                       0.05        # noise amplitude
  random_seed                 102839      # +ve integer

Suitable for initialising isothermal spinodal decomposition, the order
parameter may be set at random at each position via
:math:`\phi = \phi_0 + A(\phi_r - 1/2)` with the random variate
:math:`\phi_r` selected uniformly on the interval :math:`[0,1)`. For symmetric
quenches (mean order parameter :math:`\phi_0 = 0` and
:math:`\phi^\star = \pm 1`),
a value of :math:`A` in the range 0.05-0.1 is usually appropriate.

For off-symmetric quenches, larger patches of fluid may be required to
initiate decomposition:

.. code-block:: none

  phi_initialisation          patches     # patches of phi = +/- 1
  phi_init_patch_size         2           # patch size
  phi_init_patch_vol          0.1         # volume fraction phi = -1 phase
  random_seed                 13          # +ve integer

The initialises cubics patches of fluid of given size with :math:`\phi= \pm 1`
at random. The requested overall volume fractions may be met approximately.


A uniform value of the order parameter may be apprpropriate for
some situations. This is arranged using a single value :math:`\phi_0`:

.. code-block:: none

  phi_initialisation          uniform     # same everywhere
  phi0                        0.2         # the uniform value

A spherical drop can be initialised at the centre of the system.

.. code-block:: none

  phi_initialisation          drop        # spherical droplet
  phi_init_drop_radius        16.0        # radius
  phi_init_drop_amplitude    -1.0         # phi value inside

The drop is initialised with a :math:`\tanh(r/\xi)` profile where the
interfacial width :math:`\xi` is computed via the current free energy
parameters.

A pair of plane interfaces at :math:`z = L_z/4` and :math:`z=3L_z/4` may
be intialised via

.. code-block:: none

  phi_initialisation          block

The interfacial width is again set via the current free energy parameters.
The centre of the system has order parameter :math:`\phi = +\phi^\star`.


For restarted simulations, the default position is to read order
parameter information from file

.. code-block:: none

  phi_initialisation          from_file

in which case a file or files for the appropriate time step should
be present in the working directory.

Tensor order parameter
^^^^^^^^^^^^^^^^^^^^^^


A number of different initialisations are available for the liquid
crystal order parameter :math:`Q_{\alpha\beta}`. Some care may be required
to ensure consistency between the choice and the free energy
parameters, the system size, and so on (particularly for the blue phases).

A summary of choices is:

.. code-block:: none

  lc_q_initialisation   nematic          # uniform nematic...
  lc_init_nematic       1.0_0.0_0.0      # ...with given director

  lc_q_initialisation   cholesteric_x    # cholesteric with helical axis x
  lc_q_initialisation   cholesteric_y    # cholesteric with helical axis y
  lc_q_initialisation   cholesteric_z    # cholesteric with helical axis z

  lc_q_initialisation   o8m              # BPI high chirality limit
  lc_q_initialisation   o2               # BPII high chirality limit
  lc_q_initialisation   o5
  lc_q_initialisation   h2d              # 2d hexagonal
  lc_q_initialisation   h3da             # 3d hexagonal BP A
  lc_q_initialisation   h3db             # 3d hexagonal BP B
  lc_q_initialisation   dtc              # double twist cylinders

  lc_q_initialisation   bp3

  lc_q_initialisation   cf1_x            # cholesteric ``finger'' axis x
  lc_q_initialisation   cf1_y            # cholesteric ``finger'' axis y
  lc_q_initialisation   cf1_z            # cholesteric ``finger'' axis z

  lc_q_initialisation   cf1_fluc_x       # as cf1_x with random perterbations
  lc_q_initialisation   cf1_fluc_y       # as cf1_y with random perturbations
  lc_q_initialisation   cf1_flux_z       # as cf1_z with random perturbations

  lc_q_initialistion    random           # with randomly chosen unit director

Note many of the initialiations require an initial amplitude of order,
which should be set via

.. code-block:: none

  lc_q_init_amplitude   0.01             # initial amplitude of order A

For example, if an initial uniform nematic is requested with
unit director :math:`n_\alpha`, the corresponding initial tensor will be

.. math::

  Q_{\alpha\beta} = 
  {\textstyle \frac{1}{2}} A (3 n_\alpha n_\beta - \delta_{\alpha\beta}).





















