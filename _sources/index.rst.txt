.. Ludwig documentation master file, created by
   sphinx-quickstart on Mon Feb 17 21:40:24 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.



Ludwig: a lattice Boltzmann code for complex fluids
---------------------------------------------------

Overview
^^^^^^^^

This introduction provides a brief overview of what the *Ludwig* code does.
It is assumed that the reader has at least a general knowledge of
Navier-Stokes hydrodynamics, complex fluids, and to some extent statistical
physics. This knowledge will be required to make sense of the input and
output involved in using the code. Those wanting to work on or develop the
code itself will need knowledge of ANSI C, and perhaps message passing and
CUDA.

Purpose of the code
"""""""""""""""""""

The *Ludwig* code has been developed over a number of years to address specific
problems in complex fluids. The underlying hydrodynamic model is based on the
lattice Boltzmann equation (LBE, or just "LB"). This itself may be used to
study simple (Newtonian) fluids in a number of different scenarios, including
porous media and particle suspensions. However, the code is more generally
suited to complex fluids, where a number of options are available, among
others: symmetric binary fluids and Brazovskii smectics, polar gels,
liquid crystals, or charged fluid via a Poisson-Boltzmann equation approach.
These features are added in the framework of a free energy approach, where
specific compositional or orientational order parameters are evolved according
to the appropriate coarse-grained dynamics, but also interact with the fluid
in a fully coupled fashion.


How the code works
""""""""""""""""""

The code is written in ANSI C, and can be used to perform serial and scalable
parallel simulations of complex fluid systems based around hydrodynamics via
the lattice Boltzmann method. Time evolution of modelled quantities takes
place on a fixed regular discrete lattice. The preferred method of dealing
with the corresponding order parameter equations is by using finite
difference. However, for the case of a binary fluid, a two-distribution
lattice Boltzmann approach is also maintained for historical reference.

Users control the operation of the code via a plain text input file; output
for various data are available. These data may be visualised using appropriate
third-party software (e.g., Paraview). Specific diagnostic output may require
alterations to the code.

Potential users should note that the complex fluid simulations enabled by
*Ludwig* can be time consuming, prone to instability, and provide results
which are difficult to interpret.

License
"""""""

The code is developed largely at The University of Edinburgh, and is freely
available under a BSD-style license. See https://github.com/ludwig-cf/ludwig. 

Contents
^^^^^^^^

.. toctree::
   :maxdepth: 2
   :numbered:

   building/index
   inputs/index
   initial/index
   outputs/index


Index
"""""

* :ref:`genindex`
* :ref:`search`
