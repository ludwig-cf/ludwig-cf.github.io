
Viscosity Models
----------------

A number of different viscosity models are available.

.. toctree::
   :maxdepth: 1


Newtonian Fluid
^^^^^^^^^^^^^^^

The ...

Arrhenius
^^^^^^^^^

For free energies involing a compositional order parameter :math:`\phi`
one can define a local viscosity following a relation owing
to Arrhenius:

.. math::
   \eta(\mathbf{r})
   = \eta_{-}^{(1-\phi/\phi^\star)/2} \eta_{+}^{(1+\phi/\phi^\star)/2}.

For example for a symmetric binary fluid with
:math:`\phi^\star = \pm (-A/B)^{1/2}`, :math:`\eta_+` is the viscosity
when the composition is :math:`\phi = +\phi^\star` and :math:`\eta_-`
is the viscosity when :math:`\phi = -\phi^\star`.

This may be specified in the input file via

.. code-block:: none

     viscosity_model                  arrhenius
     viscosity_arrhenius_eta_plus     0.1
     viscosity_arrhenius_eta_minus    0.5
     viscoisty_arrhenius_phistar      1.0

Values for :math:`\eta_+` and :math:`\eta_-` must be specified, and
set the shear viscosity in the corresponding phases. If the value
for :math:`\phi^\star` is omitted, it will default to that specified
by the free energy parameters :math:`(-A/B)^{1/2}`. However, one is
free to choose a :math:`\phi^\star` for the viscosity model independently
of the value that would be computed from the free energy parameters.

The values of :math:`\eta_+` and :math:`\eta_-` override any Newtonian
viscosity that may also be specified in the input. The local bulk viscosity
is computed via

.. math::

	\eta_\nu (\mathbf{r}) = (\eta_\nu/\eta) \eta (\mathbf{r})

where :math:`\eta` and :math:`\eta_\nu` are the fixed Newtonian shear
and bulk viscosities, respectively.
