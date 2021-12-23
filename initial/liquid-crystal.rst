Liquid crystal order
--------------------

.. contents:: This section describes some available initial coniditions for
              binary fluid composition
   :depth: 1
   :local:
   :backlinks: none


Various standard initialisations for the tensor order parameter
:math:`Q_{\alpha\beta}` are available and are discussed below.



Nematic initialisation
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none
  
  lc_q_initialisation   nematic          # uniform nematic...
  lc_init_nematic       1.0_0.0_0.0      # director n_alpha

Here, based on the input director :math:`n_\alpha`, the tensor order
parameter will be initialised uniformly in uni-axial fashion:

.. math::

  Q_{\alpha\beta} = 
  {\textstyle \frac{1}{2}} A_0 (3 n_\alpha n_\beta - \delta_{\alpha\beta}).

The value of the amplitude :math:`A_0` is taken from the corresponding
current free energy parameter. If the input director is not a unit
vector, it is normalised to be so before the initialisation. 


Cholesteric initialisation
^^^^^^^^^^^^^^^^^^^^^^^^^^

The following simple cholesteric initialisations are available:

.. code-block:: none

  lc_q_initialisation      cholesteric_x
  lc_q_initialisation      cholesteric_y
  lc_q_initialisation      cholesteric_z

These give, respectively, a helical axis along the :math:`x`-direction,
:math:`y`-direction or :math:`z`-direction. For example, if the
:math:`x`-direction is selected, then a director
:math:`[0,\cos(q_0 x), \sin(q_0 x)]` is computed and the tensor order
parameter initialised from the unixial approximation

.. math::

  Q_{\alpha\beta} = 
  {\textstyle \frac{1}{2}} A_0 (3 n_\alpha n_\beta - \delta_{\alpha\beta}),

with :math:`A_0` taken form the corresponding free energy parameter.


For the :math:`y`-direction the uniaxial director is
:math:`[\cos(q_0 y),0,-\sin(q_0 y)]`, and for the :math:`z`-direction
the director is :math:`[\cos(q_0 z), \sin(q_0 z),0]`.


Blue phase initialisations
^^^^^^^^^^^^^^^^^^^^^^^^^^

The reader is referred to Wright and Mermin [WrightAndMermin1989]_
for the theoretical background on blue phases.

Blue phase I ("o8m")
""""""""""""""""""""

An approximation to the order parameter for BPI (referred to as :math:`O^{8-}`
by Wright and Mermin)  is available in the high-chirality limit,
and can be used as the basis for initialisation.

A suitable initialisation might be:

.. code-block:: none
   
  lc_q_initialisation   o8m      # Initialisation is BPI
  lc_q_init_amplitude   -0.2     # amplitude A

If we write :math:`C_x = \cos(\sqrt{2} q_0 x)`,
:math:`S_x = \sin(\sqrt{2} q_0 x)` etc, then the order parameter
components are:

.. math::

  Q_{xx} = A (-2 C_y S_z + S_x C_z + C_x S_y) 

and so on. A negative amplitude should be selected BECAUSE...

The wavevector is typically selected so that the system length is
:math:`L = \sqrt{2} p`, that is, a unit cell.


Blue phase II ("o2")
""""""""""""""""""""

The approximation in the high chirality limit referred to by Wright and
Mermin as :math:`O^2` is appropriate for blue phase II, and is
introduced via

.. code-block:: none
  
  lc_q_initialisation     o2
  lc_q_init_amplitude     0.3

Again using :math:`C_x` and :math:`S_x` the approximation is:

.. math::

  Q_{xx} = A(C_z - C_y)


and so on. Note the amplitude here is positive.



Rotated blue phase initialisations
""""""""""""""""""""""""""""""""""

An additional rotation can be applied to the following blue phase
initialisations to adjust their orientation:

.. code-block:: none
  
  lc_q_initialisation       o8m       # Available
  lc_q_initialisation       o2        # Available
  lc_q_init_euler_angles    45_45_0   # e.g., Euler angles (a,b,c).

The rotation applied for Euler angles (:math:`\alpha, \beta, \gamma`)
is a standard Euler rotation

.. math::
  
  M_z(\gamma) M_x(\beta) M_z(\alpha)

representing a sequence of rotations: first, around the :math:`z`-axis by angle
:math:`\alpha` to obtain rotated frame :math:`(x', y',z)`; second around the
new :math:`x'`-axis to obtain frame :math:`(x', y'', z')`; and third
around the :math:`z'`-axis to obtain the final result. The rotation is
around the centre of the system.

The angles entered should be in degrees.

Note that an arbitrary rotation of the initial conditions based on the
high chirality limit may introduce frustration at the periodic
boundaries if the disclination lines do not match. No such issue
occurs if boundaries are not periodic.


Miscellaneous liquid crystal initialisations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



.. [WrightAndMermin1989] D.C. Wright and N.D. Mermin, Crystalline liquids:
                         the blue phases,
                         *Rev. Mod. Phys.*, **61** 385-432 (1989).
