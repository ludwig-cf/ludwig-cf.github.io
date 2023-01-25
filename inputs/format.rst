
The input file
--------------

.. contents:: In this section
   :depth: 1
   :local:
   :backlinks: none

Using the input file
^^^^^^^^^^^^^^^^^^^^

By default, the run time expects to find user input in a file
``input`` in the current working directory. If a different
file name is required, its name should be provided as the sole
command line argument, e.g.,

.. code-block:: none

  ./Ludwig.exe input_file_name

If the input file is not located in the current working directory
the code will terminate immediately with an error message.

When an input file is located, its content is read by a single MPI
task, and its contents then broadcast to all MPI relevant tasks.
The format of the file is plain ASCII text, and its contents are
parsed on a line by line basis. Lines may contain the following:

.. code-block:: none

  # Comments are introduced by a # and are ignored
  # Here is a key value pair
  key value

Blank lines are treated as comments. The behaviour of the code is
determined by a set of key value pairs. Any given key may appear
only once in the input file. If the key value pairs are not correctly
formed, the code will usually terminate
with an error message and indicate the offending input line.

Key value pairs
^^^^^^^^^^^^^^^

Key value pairs are made up of a key --- an alphanumeric string with no
white space --- and corresponding value following white space. Values
may take on the follow forms:

.. code-block:: none

  key_string           value_string

  key_integer_scalar   1
  key_integer_vector   1_2_3

  key_double_scalar    1.0
  key_double_vector    1.0_2.0_3.0

  # keys intended for logical switches may take a number of different values
  key_switch_off       [0 | no  | off]
  key_switch_on        [1 | yes | on ]

Values which are strings should contain no white space. Scalar parameters
may be integer values, or floating point values with a decimal point
(scientific notation is also allowed).  Vector parameters are introduced
by a set of three values (to be interpreted as :math:`x,y,z` components of the
vector in Cartesian coordinates) separated by an underscore. The identity
of the key will specify what type of value is expected. Keys and (string)
values are case sensitive.


Most keys have an associated default value which will be used if
the key is not present. Some keys must be specified: an error will
occur if they are missing. The remainder of this part
of the guide details the various choices for key value pairs,
along with any default values, and any relevant constraints.

Unused key value pairs
^^^^^^^^^^^^^^^^^^^^^^

Key value pairs which appear in the input file, but are not used by
the code at run time, are reported at the end of the run. This may
indicate that the key string is incorrect or unrecognised.
