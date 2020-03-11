
Porous Media
------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Specifying porous media
^^^^^^^^^^^^^^^^^^^^^^^

Porous media calculations can be undertaken when appropriate
solid/fluid status information is supplied. The are a number
of switches available in the input file:

.. code-block:: none

  porous_media_file     name          # stub
  porous_media_format   ASCII         # ASCII or BINARY
  porous_media_type     status_only   # file context

specifies the file stub name to be read at the start of execution.
(If the stub name is ``file`` then the code will expect to
find ``file.001-001`` in the current directory.
``porous_media_format`` is either ``ASCII`` or ``BINARY``
as appropriate. Note that in parallel, a single data file can be supplied,
but it must be binary. The default is ``BINARY``. The key
<tt>porous_media_format</tt> is either ``status_only``, in which
case the file content is the solid/fluid status alone, or
``status_with_c_h``, in which case two wetting free energy constants
are included.

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

Where wetting information is required, the free energy parameters :math:`C`
and
:math:`H` can be supplied by using the ``status_with_c_h`` switch. In this
case, the ``char`` status is augmented by two ``double``
valued which are the local values of :math:`C` and :math:`H`. The order is then
:math:`s_1, c_1, h_1, s_2, c_2, h_2, \ldots`.

To set a wetting angle other than 90 degrees, it is usual to set
:math:`C = 0`, and set an approriate value of :math:`H`.


An example of how to construct a porous media file is provided in
``util/capillary.c``, which builds an appropriate file for
a square or circular capillary tube. Please see the comments in
the file for further details. Note that the allowed
solid/fluid status values are defined in ``src/map.h``.
A solid boundary is ``MAP_BOUNDARY``, while fluid is ``MAP_FLUID``.















