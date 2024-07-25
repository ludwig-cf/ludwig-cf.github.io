
Porous Media
------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Porous media calculations can be undertaken when appropriate
solid/fluid status information is supplied. A number of standard
structures are available via the input file; more generally, a
file with solid/fluid status and (optionally) wetting data is
required.

Standard porous media structures
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Details of various structures follow. Note that uniform wetting parameters
can be selected via the relevant free energy parameters (as discussed in
the section on :doc:`walls`).
If a general wetting pattern is required, a file must be
prepared. See below.

Cylindrical pipe
""""""""""""""""

.. code-block:: none

  porous_media_init      circle_xy

A cylindrical structure allowing flow in the z-direction is generated
with diameter :math:`(L_x - 2)/2`. The dimensions :math:`L_x` and
:math:`L_y` must be equal.

Rectanular pipe
"""""""""""""""

.. code-block:: none

  porous_media_init      square_xy

A rectangular (or square) pipe allowing flow in the z-direction is
generated with extent :math:`(L_x-2)` in the x-direction, and
:math:`(L_y - 2)` in the y-direction.

Walls in one coordinate direction
"""""""""""""""""""""""""""""""""

.. code-block:: none

  porous_media_init      wall_x
  porous_media_init      wall_y
  porous_media_init      wall_z

Generates a structure with, e.g., solid plane walls at :math:`x = 1`
and :math:`x = L_x` (channel width :math:`L_x-2`), and periodic in the
other two dimensions.

Simple cubic crystal of spheres
"""""""""""""""""""""""""""""""

.. code-block:: none

  porous_media_init      simple_cubic
  porous_media_acell     10

This will initialise an array of sphere in a simple cubic array with
lattice constant :math:`a`. The lattice constant must exactly divide
all three coordinate lengths (i.e., there should be an integer number
of unit cells).

The sphere radius is :math:`a/2`, and the solid volume fraction should be
close to the theoretical value of :math:`\pi/6 \approx 0.52` (with
some discretisation error).


Body centred cubic crystal of spheres
"""""""""""""""""""""""""""""""""""""

.. code-block:: none

  porous_media_init      body_centred_cubic
  porous_media_acell     10

As for the simple cubic case, but here body-centred cubic.
The sphere radius is :math:`a\sqrt{3}/4`
The solid fraction should be close to the theoretical value of
:math:`\pi \sqrt{3}/8 \approx 0.68`.


Face centred cubic crystal of spheres
"""""""""""""""""""""""""""""""""""""

.. code-block:: none

  porous_media_init      face_centred_cubic
  porous_media_acell     10

Again, as for the simple cubic case, but here face-centred cubic.
The sphere radius is approx :math:`a\sqrt{2}/4`, and the
solid fraction should be
close to the theoretical figure of :math:`\pi \sqrt{2}/6 \approx 0.74`.




Porous media structures from file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none

  porous_media_file     yes            # "map-000000000.001-001" is present
  porous_media_data     status_with_h  # file contents [status_only]


In this case, the run time will expect to find the file
``map-000000000.001-001`` in the current directory. The
``porous_media_data`` key describes the contents of the file, with
a number of possible values. These are illustrated below. The
solid/fluid status must be present, while additional wetting
information is also permitted:


.. code-block:: none

  porous_media_data     status_only             # solid/fluid status only
  porous_media_data     status_with_h           # Free energy H
  porous_media_data     status_with_c_h         # Free energy C and H

Only one ``porous_media_type`` can be defined at one time. The default
is ``status_only``, i.e., no wetting information.


File format
"""""""""""

In all cases the status (fluid/solid) information is represented by a
single ``char`` (or integer), which must be supplied via the
porous media file. A single file should contain data matching the
current system size, and have the :math:`z`-direction running fastest,
followed by the :math:`y`-direction. Note that in non-periodic directions,
the structure must be 'closed', i.e., all the points at the edge
should be solid.


Fluid sites are designated by ``0`` and boundary or solid sites
by ``1``. These data should be of type ``char`` in binary,
and may be integer in ASCII.

Where wetting information is required, the relevant free energy parameters
should be provided. These are of data type `double`. For example, in the
symmetric free energy picture, values
:math:`C` and :math:`H` are required at each site. The stoarge order is
then :math:`s_1, c_1, h_1, s_2, c_2, h_2, \ldots`.

An example of how to construct a porous media file is provided in
``util/capillary.c``, which writes an appropriate file for
a square or circular capillary tube. Note that as the standard
output mechanism is used, it is only necessary to make the relevant
assignments to the data structure.

Please see the comments in
the file for further details. Note that the allowed
solid/fluid status values are defined in ``src/map.h``.
A solid boundary is ``MAP_BOUNDARY``, while fluid is ``MAP_FLUID``.
