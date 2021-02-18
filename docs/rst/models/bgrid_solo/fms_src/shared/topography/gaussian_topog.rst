module gaussian_topog_mod
-------------------------

.. container::

   **Contact:**  `Bruce Wyman <mailto:bw@gfdl.noaa.gov>`__
   **Reviewers:** 
   **Change History:**  `WebCVS Log <http://www.gfdl.noaa.gov/fms-cgi-bin/cvsweb.cgi/FMS/>`__
   **Last Modified:** 2002/03/22 01:42:43

--------------

OVERVIEW
^^^^^^^^

Routines for creating Gaussian-shaped land surface topography for latitude-longitude grids.

.. container::

   Interfaces generate simple Gaussian-shaped mountains from parameters specified by either argument list or namelist
   input. The mountain shapes are controlled by the height, half-width, and ridge-width parameters.

| 
| 

--------------

OTHER MODULES USED
^^^^^^^^^^^^^^^^^^

.. container::

   ::

            fms_mod
      constants_mod

--------------

PUBLIC INTERFACE
^^^^^^^^^^^^^^^^

.. container::

   ::

      use gaussian_topog_mod [, only:  gaussian_topog_init,
                                       get_gaussian_topog ]

   `gaussian_topog_init <#gaussian_topog_init>`__:
      Returns a surface height field that consists of the sum of one or more Gaussian-shaped mountains.
   `get_gaussian_topog <#get_gaussian_topog>`__:
      Returns a simple surface height field that consists of a single Gaussian-shaped mountain.

| 
| 

--------------

PUBLIC DATA
^^^^^^^^^^^

.. container::

   None.

--------------

PUBLIC ROUTINES
^^^^^^^^^^^^^^^

a. 

   .. rubric:: gaussian_topog_init
      :name: gaussian_topog_init

   ::

      <B>call gaussian_topog_init </B> ( lon, lat, zsurf )

   **DESCRIPTION**
      Returns a land surface topography that consists of a "set" of simple Gaussian-shaped mountains. The height,
      position, width, and elongation of the mountains can be controlled by variables in namelist
      `&gaussian_topog_nml <#NAMELIST>`__.
   **INPUT**
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``lon   ``                                                | The mean grid box longitude in radians.                   |
      |                                                           |    [real, dimension(:)]                                   |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``lat   ``                                                | The mean grid box latitude in radians.                    |
      |                                                           |    [real, dimension(:)]                                   |
      +-----------------------------------------------------------+-----------------------------------------------------------+

   **OUTPUT**
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``zsurf   ``                                              | The surface height (in meters). The size of this field    |
      |                                                           | must be size(lon) by size(lat).                           |
      |                                                           |    [real, dimension(:,:)]                                 |
      +-----------------------------------------------------------+-----------------------------------------------------------+

b. 

   .. rubric:: get_gaussian_topog
      :name: get_gaussian_topog

   ::

      zsurf = <B> get_gaussian_topog </B> ( lon, lat, height [, olond, olatd, wlond, wlatd, rlond, rlatd ] )

   **DESCRIPTION**
      Returns a single Gaussian-shaped mountain. The height, position, width, and elongation of the mountain is
      controlled by optional arguments.
   **INPUT**
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``lon   ``                                                | The mean grid box longitude in radians.                   |
      |                                                           |    [real, dimension(:)]                                   |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``lat   ``                                                | The mean grid box latitude in radians.                    |
      |                                                           |    [real, dimension(:)]                                   |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``height   ``                                             | Maximum surface height in meters.                         |
      |                                                           |    [real, dimension(scalar)]                              |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``olond, olatd   ``                                       | Position/origin of mountain in degrees longitude and      |
      |                                                           | latitude. This is the location of the maximum height.     |
      |                                                           |    [real, dimension(scalar)]                              |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``wlond, wlatd   ``                                       | Gaussian half-width of mountain in degrees longitude and  |
      |                                                           | latitude.                                                 |
      |                                                           |    [real, dimension(scalar)]                              |
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``rlond, rlatd   ``                                       | Ridge half-width of mountain in degrees longitude and     |
      |                                                           | latitude. This is the elongation of the maximum height.   |
      |                                                           |    [real, dimension(scalar)]                              |
      +-----------------------------------------------------------+-----------------------------------------------------------+

   **OUTPUT**
      +-----------------------------------------------------------+-----------------------------------------------------------+
      | ``zsurf   ``                                              | The surface height (in meters). The size of the returned  |
      |                                                           | field is size(lon) by size(lat).                          |
      |                                                           |    [real, dimension(:,:)]                                 |
      +-----------------------------------------------------------+-----------------------------------------------------------+

   **NOTE**
      Mountains do not wrap around the poles.

--------------

NAMELIST
^^^^^^^^

.. container::

   **&gaussian_topog_nml**
   ``height``
   Height in meters of the Gaussian mountains.
   [real, dimension(mxmtns), units: meter, default: 0.]
   ``olon, olat``
   The longitude and latitude of mountain origins (in degrees).
   [real, dimension(mxmtns), units: degree, default: 0.]
   ``wlon, wlat``
   The longitude and latitude half-width of mountain tails (in degrees).
   [real, dimension(mxmtns), units: degree, default: 0.]
   ``rlon, rlat``
   The longitude and latitude half-width of mountain ridges (in degrees). For a "standard" Gaussian mountain set
   rlon=rlat=0.
   [real, dimension(mxmtns), units: degree, default: 0.]
   ``NOTE``
   The variables in this namelist are only used when routine <TT>gaussian_topog_init</TT> is called. The namelist
   variables are dimensioned (by 10), so that multiple mountains can be generated.
   Internal parameter mxmtns = 10. By default no mountains are generated.
   []

| 
| 

--------------

DATA SETS
^^^^^^^^^

.. container::

   None.

--------------

ERROR MESSAGES
^^^^^^^^^^^^^^

.. container::

   **FATAL in get_gaussian_topog**
      shape(zsurf) is not equal to (/size(lon),size(lat)/)
      Check the input grid size and output field size. The input grid is defined at the midpoint of grid boxes.

--------------

REFERENCES
^^^^^^^^^^

.. container::

   None.

| 
| 

--------------

COMPILER SPECIFICS
^^^^^^^^^^^^^^^^^^

.. container::

   None.

| 
| 

--------------

PRECOMPILER OPTIONS
^^^^^^^^^^^^^^^^^^^

.. container::

   None.

| 
| 

--------------

LOADER OPTIONS
^^^^^^^^^^^^^^

.. container::

   None.

--------------

TEST PROGRAM
^^^^^^^^^^^^

.. container::

   None.

| 
| 

--------------

KNOWN BUGS
^^^^^^^^^^

.. container::

   None.

| 
| 

--------------

NOTES
^^^^^

.. container::

   NAMELIST FOR GENERATING GAUSSIAN MOUNTAINS
   \* multiple mountains can be generated \* the final mountains are the sum of all
   height = height in meters olon, olat = longitude,latitude origin (degrees) rlon, rlat = longitude,latitude half-width
   of ridge (degrees) wlon, wlat = longitude,latitude half-width of tail (degrees)
   Note: For the standard gaussian mountain set rlon = rlat = 0 .
   ::

             height -->   ___________________________
                         /                           \
                        /              |              \
          gaussian     /               |               \
            sides --> /                |                \
                     /               olon                \
               _____/                olat                 \______

                    |    |             |
                    |<-->|<----------->|
                    |wlon|    rlon     |
                     wlat     rlat

   See the `topography <topography.html#TEST%20PROGRAM>`__ module documentation for a test program.

| 
| 

--------------

FUTURE PLANS
^^^^^^^^^^^^

.. container::

   None.

| 

--------------

.. container::

   `top <#TOP>`__