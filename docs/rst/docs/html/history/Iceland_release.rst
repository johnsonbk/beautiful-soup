Iceland
=======

DART Iceland release documentation
----------------------------------

.. attention::

   Iceland is a prior release of DART. Its source code is available via the `DART repository on
   Github <https://github.com/NCAR/DART/tree/Iceland>`__. This documentation is preserved merely for reference. See the
   `DART homepage <https://dart.ucar.edu/>`__ to learn about the latest release.

Overview of DART
----------------

The Data Assimilation Research Testbed (DART) is designed to facilitate the combination of assimilation algorithms,
models, and observation sets to allow increased understanding of all three. The DART programs have been compiled with
several Fortran 90 compilers and run on a linux compute-server and linux clusters. You should definitely read the
Customizations section.

DART employs a modular programming approach to apply an Ensemble Kalman Filter which nudges models toward a state that
is more consistent with information from a set of observations. Models may be swapped in and out, as can different
algorithms in the Ensemble Kalman Filter. The method requires running multiple instances of a model to generate an
ensemble of states. A forward operator appropriate for the type of observation being used is applied to each of the
states to generate the model's estimate of the observation. Comparing these estimates and their uncertainty to the
observation and its uncertainty ultimately results in the adjustments to the model states. Sort of. There's more to it,
described in detail in the tutorial directory of the package.

DART ultimately creates a few netCDF files containing the model states just before the adjustment ``Prior_Diag.nc`` and
just after the adjustment ``Posterior_Diag.nc`` as well as a file ``obs_seq.final`` with the model estimates of the
observations. There is a suite of Matlab® functions that facilitate exploration of the results.

The Iceland release provides several new models and has a greatly expanded capability for **real** observations which
required a fundamentally different implementation of the low-level routines. It is now required to run a preprocessor on
several of the program units to construct the source code files which will be compiled by the remaining units. Due to
the potentially large number of observations types possible and for portability reasons, the preprocessor is actually a
F90 program that uses the namelist mechanism for specifying the observation types to be included. This also prevents
having a gory set of compile flags that is different for every compiler. One very clever colleague also 'built a better
mousetrap' and figured out how to effectively and robustly read namelists, detect errors, and generate meaningful error
messages. HURRAY!

The Iceland release has also been tested with more compilers in an attempt to determine non-portable code elements. It
is my experience that the largest impediment to portable code is the reliance on the compiler to autopromote ``real``
variables to one flavor or another. :doc:`.//DART/PrecisionNotes` Using the F90 "kind" allows for much more flexible
code, in that the use of interface procedures is possible only when two routines do not have identical sets of input
arguments -- something that happens when the compiler autopromotes 32bit reals to 64bit reals, for example.

DART programs can require three different types of input. First, some of the DART programs, those for creating synthetic
observational datasets, require interactive input from the keyboard. For simple cases, this interactive input can be
made directly from the keyboard. In more complicated cases, a file containing the appropriate keyboard input can be
created and this file can be directed to the standard input of the DART program. Second, many DART programs expect one
or more input files in DART specific formats to be available. For instance, ``perfect_model_obs``, which creates a
synthetic observation set given a particular model and a description of a sequence of observations, requires an input
file that describes this observation sequence. At present, the observation files for DART are in a custom format in
either human-readable ascii or more compact machine-specific binary. Third, many DART modules (including main programs)
make use of the Fortan90 namelist facility to obtain values of certain parameters at run-time. All programs look for a
namelist input file called ``input.nml`` in the directory in which the program is executed. The ``input.nml`` file can
contain a sequence of individual Fortran90 namelists which specify values of particular parameters for modules that
compose the executable program. DART provides a mechanism that automatically generates namelists with the default values
for each program to be run.

DART uses the `netCDF <http://www.unidata.ucar.edu/packages/netcdf/>`__ self-describing data format with a particular
metadata convention to describe output that is used to analyze the results of assimilation experiments. These files have
the extension ``.nc`` and can be read by a number of standard data analysis tools. A set of
`Matlab <http://www.mathworks.com/>`__ scripts, designed to produce graphical diagnostics from DART netCDF output files
are available. DART users have also used `ncview <http://meteora.ucsd.edu/~pierce/ncview_home_page.html>`__ to create
rudimentary graphical displays of output data fields. The `NCO <http://nco.sourceforge.net>`__ tools, produced by UCAR's
Unidata group, are available to do operations like concatenating, slicing, and dicing of netCDF files.

Requirements: an F90 compiler
-----------------------------

The DART software has been successfully built on several Linux/x86 platforms with several versions of the `Intel Fortran
Compiler for Linux <http://www.intel.com/software/products/compilers/flin>`__, which (at one point) is/was free for
individual scientific use. It has also been built and successfully run with several versions of each of the following:
`Portland Group Fortran Compiler <http://www.pgroup.com>`__, `Lahey Fortran Compiler <http://www.lahey.com>`__,
`Pathscale Fortran Compiler <http://www.pathscale.com>`__, `Absoft Fortran 90/95 Compiler (Mac
OSX) <http://www.absoft.com>`__. Since recompiling the code is a necessity to experiment with different models, there
are no binaries to distribute.

DART uses the `netCDF <http://www.unidata.ucar.edu/packages/netcdf/>`__ self-describing data format for the results of
assimilation experiments. These files have the extension ``.nc`` and can be read by a number of standard data analysis
tools. In particular, DART also makes use of the F90 interface to the library which is available through the
``netcdf.mod`` and ``typesizes.mod`` modules. *IMPORTANT*: different compilers create these modules with different
"case" filenames, and sometimes they are not **both** installed into the expected directory. It is required that both
modules be present. The normal place would be in the ``netcdf/include`` directory, as opposed to the ``netcdf/lib``
directory.

If the netCDF library does not exist on your system, you must build it (as well as the F90 interface modules). The
library and instructions for building the library or installing from an RPM may be found at the netCDF home page:
http://www.unidata.ucar.edu/packages/netcdf/ Pay particular attention to the compiler-specific patches that must be
applied for the Intel Fortran Compiler. (Or the PG compiler, for that matter.)

The location of the netCDF library, ``libnetcdf.a``, and the locations of both ``netcdf.mod`` and ``typesizes.mod`` will
be needed by the makefile template, as described in the compiling section.

Unpacking the distribution
--------------------------

This release of the `DART source code can be downloaded <https://github.com/NCAR/DART/releases/tag/v5.0.0>`__ as a
compressed zip or tar.gz file. When extracted, the source tree will begin with a directory named ``DART`` and will be
approximately 103.1 Mb. Compiling the code in this tree (as is usually the case) will necessitate much more space.

::


   $ gunzip DART-5.0.0.tar.gz
   $ tar -xvf DART-5.0.0.tar

The code tree is very "bushy"; there are many directories of support routines, etc. but only a few directories involved
with the customization and installation of the DART software. If you can compile and run ONE of the low-order models,
you should be able to compile and run ANY of the low-order models. For this reason, we can focus on the Lorenz \`63
model. Subsequently, the only directories with files to be modified to check the installation are: ``DART/mkmf``,
``DART/models/lorenz_63/work``, and ``DART/matlab`` (but only for analysis).

Customizing the build scripts -- overview
-----------------------------------------

DART executable programs are constructed using two tools: ``make`` and ``mkmf``. The ``make`` utility is a relatively
common piece of software that requires a user-defined input file that records dependencies between different source
files. ``make`` then performs a hierarchy of actions when one or more of the source files is modified. The ``mkmf``
utility is a custom preprocessor that generates a ``make`` input file (named ``Makefile``) and an example namelist
*input.nml.\ program\ \_default* with the default values. The ``Makefile`` is designed specifically to work with
object-oriented Fortran90 (and other languages) for systems like DART.

``mkmf`` requires two separate input files. The first is a \`template' file which specifies details of the commands
required for a specific Fortran90 compiler and may also contain pointers to directories containing pre-compiled
utilities required by the DART system. **This template file will need to be modified to reflect your system**. The
second input file is a \`path_names' file which includes a complete list of the locations (either relative or absolute)
of all Fortran90 source files that are required to produce a particular DART program. Each 'path_names' file must
contain a path for exactly one Fortran90 file containing a main program, but may contain any number of additional paths
pointing to files containing Fortran90 modules. An ``mkmf`` command is executed which uses the 'path_names' file and the
mkmf template file to produce a ``Makefile`` which is subsequently used by the standard ``make`` utility.

Shell scripts that execute the mkmf command for all standard DART executables are provided as part of the standard DART
software. For more information on ``mkmf`` see `the FMS mkmf
description <http://www.gfdl.gov/fms/pubrel/j/atm_dycores/doc/dycore_public_manual.html#mkmf>`__.

One of the benefits of using ``mkmf`` is that it also creates an example namelist file for each program. The example
namelist is called *input.nml.\ program\ \_default*, so as not to clash with any exising ``input.nml`` that may exist in
that directory.

Building and customizing the 'mkmf.template' file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A series of templates for different compilers/architectures exists in the ``DART/mkmf/`` directory and have names with
extensions that identify either the compiler, the architecture, or both. This is how you inform the build process of the
specifics of your system. Our intent is that you copy one that is similar to your system into ``mkmf.template`` and
customize it. For the discussion that follows, knowledge of the contents of one of these templates (i.e.
``mkmf.template.pgf90.ghotiol``) is needed: (note that only the LAST lines are shown here, the head of the file is just
a big comment)

::


   # Makefile template for PGI f90
   FC = pgf90
   LD = pgf90
   CPPFLAGS =
   LIST = -Mlist
   NETCDF = /contrib/netcdf-3.5.1-cc-c++-pgif90.5.2-4
   FFLAGS = -O0 -Ktrap=fp -pc 64 -I$(NETCDF)/include
   LIBS = -L$(NETCDF)/lib -lnetcdf
   LDFLAGS = $(LIBS)

   ...

Essentially, each of the lines defines some part of the resulting ``Makefile``. Since ``make`` is particularly good at
sorting out dependencies, the order of these lines really doesn't make any difference. The ``FC = pgf90`` line
ultimately defines the Fortran90 compiler to use, etc. The lines which are most likely to need site-specific changes
start with ``FFLAGS`` and ``NETCDF``, which indicate where to look for the netCDF F90 modules and the location of the
netCDF library and modules.

Netcdf
^^^^^^

| Modifying the ``NETCDF`` value should be relatively straightforward.
| Change the string to reflect the location of your netCDF installation containing ``netcdf.mod`` and ``typesizes.mod``.
  The value of the ``NETCDF`` variable will be used by the ``FFLAGS, LIBS,`` and ``LDFLAGS`` variables.

FFLAGS
^^^^^^

Each compiler has different compile flags, so there is really no way to exhaustively cover this other than to say the
templates as we supply them should work -- depending on the location of your netCDF. The low-order models can be
compiled without a ``-r8`` switch, but the ``bgrid_solo`` model cannot.

Customizing the 'path_names_*' file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Several ``path_names_*`` files are provided in the ``work`` directory for each specific model, in this case:
``DART/models/lorenz_63/work``.

#. ``path_names_preprocess``
#. ``path_names_create_obs_sequence``
#. ``path_names_create_fixed_network_seq``
#. ``path_names_perfect_model_obs``
#. ``path_names_filter``
#. ``path_names_obs_diag``

Since each model comes with its own set of files, no further customization is needed.

Building the Lorenz_63 DART project
-----------------------------------

Currently, DART executables are constructed in a ``work`` subdirectory under the directory containing code for the given
model. In the top-level DART directory, change to the L63 work directory and list the contents:

::


   $ cd DART/models/lorenz_63/work
   $ ls -1

With the result:

::


   filter_ics
   filter_restart
   input.nml
   mkmf_create_fixed_network_seq
   mkmf_create_obs_sequence
   mkmf_filter
   mkmf_obs_diag
   mkmf_perfect_model_obs
   mkmf_preprocess
   obs_seq.final
   obs_seq.in
   obs_seq.out
   obs_seq.out.average
   obs_seq.out.x
   obs_seq.out.xy
   obs_seq.out.xyz
   obs_seq.out.z
   path_names_create_fixed_network_seq
   path_names_create_obs_sequence
   path_names_filter
   path_names_obs_diag
   path_names_perfect_model_obs
   path_names_preprocess
   perfect_ics
   perfect_restart
   Posterior_Diag.nc
   Prior_Diag.nc
   set_def.out
   True_State.nc
   workshop_setup.csh

There are six ``mkmf_``\ *xxxxxx* files for the programs ``preprocess``, ``create_obs_sequence``,
``create_fixed_network_seq``, ``perfect_model_obs``, ``filter``, and ``obs_diag`` along with the corresponding
``path_names_``\ *xxxxxx* files. You can examine the contents of one of the ``path_names_``\ *xxxxxx* files, for
instance ``path_names_filter``, to see a list of the relative paths of all files that contain Fortran90 modules required
for the program ``filter`` for the L63 model. All of these paths are relative to your ``DART`` directory. The first path
is the main program (``filter.f90``) and is followed by all the Fortran90 modules used by this program (after
preprocessing).

The ``mkmf_``\ *xxxxxx* scripts are cryptic but should not need to be modified -- as long as you do not restructure the
code tree (by moving directories, for example). The only function of the ``mkmf_``\ *xxxxxx* script is to generate a
``Makefile`` and an *input.nml.\ program\ \_default* file. It is not supposed to compile anything:

::


   $ csh mkmf_preprocess
   $ make

| The first command generates an appropriate ``Makefile`` and the ``input.nml.preprocess_default`` file. The second
  command results in the compilation of a series of Fortran90 modules which ultimately produces an executable file:
  ``preprocess``. Should you need to make any changes to the ``DART/mkmf/mkmf.template``, you will need to regenerate
  the ``Makefile``.
| The ``preprocess`` program actually builds source code to be used by all the remaining modules. It is **imperative**
  to actually **run** ``preprocess`` before building the remaining executables. This is how the same code can assimilate
  state vector 'observations' for the Lorenz_63 model and real radar reflectivities for WRF without needing to specify a
  set of radar operators for the Lorenz_63 model!
| ``preprocess`` reads the ``&preprocess_nml`` namelist to determine what observations and operators to incorporate. For
  this exercise, we will use the values in ``input.nml``. ``preprocess`` is designed to abort if the files it is
  supposed to build already exist. For this reason, it is necessary to remove a couple files (if they exist) before you
  run the preprocessor. It is just a good habit to develop.

::


   $ \rm -f ../../../obs_def/obs_def_mod.f90
   $ \rm -f ../../../obs_kind/obs_kind_mod.f90
   $ ./preprocess
   $ ls -l ../../../obs_def/obs_def_mod.f90
   $ ls -l ../../../obs_kind/obs_kind_mod.f90

| This created ``../../../obs_def/obs_def_mod.f90`` from ``../../../obs_kind/DEFAULT_obs_kind_mod.F90`` and several
  other modules. ``../../../obs_kind/obs_kind_mod.f90`` was created similarly. Now we can build the rest of the project.
| A series of object files for each module compiled will also be left in the work directory, as some of these are
  undoubtedly needed by the build of the other DART components. You can proceed to create the other five programs needed
  to work with L63 in DART as follows:

::


   $ csh mkmf_create_obs_sequence
   $ make
   $ csh mkmf_create_fixed_network_seq
   $ make
   $ csh mkmf_perfect_model_obs
   $ make
   $ csh mkmf_filter
   $ make
   $ csh mkmf_obs_diag
   $ make

The result (hopefully) is that six executables now reside in your work directory. The most common problem is that the
netCDF libraries and include files (particularly ``typesizes.mod``) are not found. Edit the ``DART/mkmf/mkmf.template``,
recreate the ``Makefile``, and try again.

============================ =========================================================================================
program                      purpose
============================ =========================================================================================
``preprocess``               creates custom source code for just the observations of interest
``create_obs_sequence``      specify a (set) of observation characteristics taken by a particular (set of) instruments
``create_fixed_network_seq`` specify the temporal attributes of the observation sets
``perfect_model_obs``        spinup, generate "true state" for synthetic observation experiments, ...
``filter``                   perform experiments
``obs_diag``                 creates observation-space diagnostic files to be explored by the Matlab® scripts.
============================ =========================================================================================

Running Lorenz_63
-----------------

This initial sequence of exercises includes detailed instructions on how to work with the DART code and allows
investigation of the basic features of one of the most famous dynamical systems, the 3-variable Lorenz-63 model. The
remarkable complexity of this simple model will also be used as a case study to introduce a number of features of a
simple ensemble filter data assimilation system. To perform a synthetic observation assimilation experiment for the L63
model, the following steps must be performed (an overview of the process is given first, followed by detailed procedures
for each step):

Experiment overview
-------------------

#. Integrate the L63 model for a long time
   starting from arbitrary initial conditions to generate a model state that lies on the attractor. The ergodic nature
   of the L63 system means a 'lengthy' integration always converges to some point on the computer's finite precision
   representation of the model's attractor.
#. Generate a set of ensemble initial conditions
   from which to start an assimilation. Since L63 is ergodic, the ensemble members can be designed to look like random
   samples from the model's 'climatological distribution'. To generate an ensemble member, very small perturbations can
   be introduced to the state on the attractor generated by step 1. This perturbed state can then be integrated for a
   very long time until all memory of its initial condition can be viewed as forgotten. Any number of ensemble initial
   conditions can be generated by repeating this procedure.
#. Simulate a particular observing system
   by first creating an 'observation set definition' and then creating an 'observation sequence'. The 'observation set
   definition' describes the instrumental characteristics of the observations and the 'observation sequence' defines the
   temporal sequence of the observations.
#. Populate the 'observation sequence' with 'perfect' observations
   by integrating the model and using the information in the 'observation sequence' file to create simulated
   observations. This entails operating on the model state at the time of the observation with an appropriate forward
   operator (a function that operates on the model state vector to produce the expected value of the particular
   observation) and then adding a random sample from the observation error distribution specified in the observation set
   definition. At the same time, diagnostic output about the 'true' state trajectory can be created.
#. Assimilate the synthetic observations
   by running the filter; diagnostic output is generated.

1. Integrate the L63 model for a 'long' time
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``perfect_model_obs`` integrates the model for all the times specified in the 'observation sequence definition' file. To
this end, begin by creating an 'observation sequence definition' file that spans a long time. Creating an 'observation
sequence definition' file is a two-step procedure involving ``create_obs_sequence`` followed by
``create_fixed_network_seq``. After they are both run, it is necessary to integrate the model with
``perfect_model_obs``.

1.1 Create an observation set definition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``create_obs_sequence`` creates an observation set definition, the time-independent part of an observation sequence. An
observation set definition file only contains the ``location, type,`` and ``observational error characteristics``
(normally just the diagonal observational error variance) for a related set of observations. There are no actual
observations, nor are there any times associated with the definition. For spin-up, we are only interested in integrating
the L63 model, not in generating any particular synthetic observations. Begin by creating a minimal observation set
definition.

In general, for the low-order models, only a single observation set need be defined. Next, the number of individual
scalar observations (like a single surface pressure observation) in the set is needed. To spin-up an initial condition
for the L63 model, only a single observation is needed. Next, the error variance for this observation must be entered.
Since we do not need (nor want) this observation to have any impact on an assimilation (it will only be used for
spinning up the model and the ensemble), enter a very large value for the error variance. An observation with a very
large error variance has essentially no impact on deterministic filter assimilations like the default variety
implemented in DART. Finally, the location and type of the observation need to be defined. For all types of models, the
most elementary form of synthetic observations are called 'identity' observations. These observations are generated
simply by adding a random sample from a specified observational error distribution directly to the value of one of the
state variables. This defines the observation as being an identity observation of the first state variable in the L63
model. The program will respond by terminating after generating a file (generally named ``set_def.out``) that defines
the single identity observation of the first state variable of the L63 model. The following is a screenshot (much of the
verbose logging has been left off for clarity), the user input looks *like this*.

.. container:: unix

   ::

      [unixprompt]$ ./create_obs_sequence
       Initializing the utilities module.
       Trying to read from unit           10
       Trying to open file dart_log.out
       
       Registering module :
       $source: /home/dart/CVS.REPOS/DART/utilities/utilities_mod.f90,v $
       $revision: 1.18 $
       $date: 2004/06/29 15:16:40 $
       Registration complete.
       
       &UTILITIES_NML
       TERMLEVEL= 2,LOGFILENAME=dart_log.out                                          
                                                                                  
       /
       
       Registering module :
       $source: /home/dart/CVS.REPOS/DART/obs_sequence/create_obs_sequence.f90,v $
       $revision: 1.18 $
       $date: 2004/05/24 15:41:46 $
       Registration complete.

       { ... }

       Input upper bound on number of observations in sequence
      10
       
       Input number of copies of data (0 for just a definition)
      0

       Input number of quality control values per field (0 or greater)
      0

       input a -1 if there are no more obs 
      0

   ::


      initialize_module obs_kind_nml values are

      -------------- ASSIMILATE_THESE_OBS_TYPES --------------
      RAW_STATE_VARIABLE
      -------------- EVALUATE_THESE_OBS_TYPES --------------
      ------------------------------------------------------

          Input -1 * state variable index for identity observations
          OR input the name of the observation kind from table below:
          OR input the integer index, BUT see documentation...
                      1 RAW_STATE_VARIABLE

   ::


      -1

       input time in days and seconds
      1 0

       Input error variance for this observation definition
      1000000

       input a -1 if there are no more obs 
      -1

       Input filename for sequence (  set_def.out   usually works well)
       set_def.out 
       write_obs_seq  opening formatted file set_def.out
       write_obs_seq  closed file set_def.out

1.2 Create an observation sequence definition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``create_fixed_network_seq`` creates an 'observation sequence definition' by extending the 'observation set
  definition' with the temporal attributes of the observations.
| The first input is the name of the file created in the previous step, i.e. the name of the observation set definition
  that you've just created. It is possible to create sequences in which the observation sets are observed at regular
  intervals or irregularly in time. Here, all we need is a sequence that takes observations over a long period of time -
  indicated by entering a 1. Although the L63 system normally is defined as having a non-dimensional time step, the DART
  system arbitrarily defines the model timestep as being 3600 seconds. If we declare that we have one observation per
  day for 1000 days, we create an observation sequence definition spanning 24000 'model' timesteps; sufficient to
  spin-up the model onto the attractor. Finally, enter a name for the 'observation sequence definition' file. Note
  again: there are no observation values present in this file. Just an observation type, location, time and the error
  characteristics. We are going to populate the observation sequence with the ``perfect_model_obs`` program.

.. container:: unix

   ::

      [unixprompt]$ ./create_fixed_network_seq

       ...

       Registering module :
       $source: /home/dart/CVS.REPOS/DART/obs_sequence/obs_sequence_mod.f90,v $
       $revision: 1.31 $
       $date: 2004/06/29 15:04:37 $
       Registration complete.
       
       Input filename for network definition sequence (usually  set_def.out  )
      set_def.out

       ...

       To input a regularly repeating time sequence enter 1
       To enter an irregular list of times enter 2
      1
       Input number of observations in sequence
      1000
       Input time of initial ob in sequence in days and seconds
      1, 0
       Input period of obs in days and seconds
      1, 0
                 1
                 2
                 3
      ...
               997
               998
               999
              1000
      What is output file name for sequence (  obs_seq.in   is recommended )
      obs_seq.in
       write_obs_seq  opening formatted file obs_seq.in
       write_obs_seq closed file [blah blah blah]/work/obs_seq.in

1.3 Initialize the model onto the attractor
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``perfect_model_obs`` can now advance the arbitrary initial state for 24,000 timesteps to move it onto the attractor.

``perfect_model_obs`` uses the Fortran90 namelist input mechanism instead of (admittedly gory, but temporary)
interactive input. All of the DART software expects the namelists to found in a file called ``input.nml``. When you
built the executable, an example namelist was created ``input.nml.perfect_model_obs_default`` that contains all of the
namelist input for the executable. If you followed the example, each namelist was saved to a unique name. We must now
rename and edit the namelist file for ``perfect_model_obs``. Copy ``input.nml.perfect_model_obs_default`` to
``input.nml`` and edit it to look like the following:

::


      &perfect_model_obs_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      obs_seq_in_file_name = "obs_seq.in",
      obs_seq_out_file_name = "obs_seq.out",
      start_from_restart = .false.,
      output_restart = .true.,
      restart_in_file_name = "perfect_ics",
      restart_out_file_name = "perfect_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_interval = 1 /

   &ensemble_manager_nml
      in_core = .true.,
      single_restart_file_in = .true.,
      single_restart_file_out = .true. /

   &assim_tools_nml
      filter_kind = 1,
      cutoff = 0.2,
      sort_obs_inc = .false.,
      cov_inflate = -1.0,
      cov_inflate_sd = 0.05,
      sd_lower_bound = 0.05,
      deterministic_cov_inflate = .true.,
      start_from_assim_restart = .false.,
      assim_restart_in_file_name =
   'assim_tools_ics',
      assim_restart_out_file_name =
   'assim_tools_restart',
      do_parallel = 0,
      num_domains = 1
      parallel_command = "./assim_filter.csh",
      spread_restoration = .false.,
      cov_inflate_upper_bound = 10000000.0,
      internal_outlier_threshold = -1.0 /

   &cov_cutoff_nml
      select_localization = 1 /

   &reg_factor_nml
      select_regression = 1,
      input_reg_file = "time_mean_reg"
      save_reg_diagnostics = .false.,
      reg_diagnostics_file = 'reg_diagnostics' /

   &obs_sequence_nml
      write_binary_obs_sequence = .false. /

   &obs_kind_nml
      assimilate_these_obs_types = 'RAW_STATE_VARIABLE' /

   &assim_model_nml
      write_binary_restart_files = .true. /

   &model_nml
      sigma = 10.0,
      r = 28.0,
      b = 2.6666666666667,
      deltat = 0.01,
      time_step_days = 0,
      time_step_seconds = 3600 /

   &utilities_nml
      TERMLEVEL = 1
      logfilename = 'dart_log.out' /

For the moment, only two namelists warrant explanation. Each namelists is covered in detail in the html files
accompanying the source code for the module.

perfect_model_obs_nml
~~~~~~~~~~~~~~~~~~~~~

+---------------------------+-----------------------------------------------------------------------------------------+
| namelist variable         | description                                                                             |
+===========================+=========================================================================================+
| ``async``                 | For the lorenz_63, simply ignore this. Leave it set to '0'                              |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``advance_ens_command``   | specifies the shell commands or script to execute when async /= 0                       |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``obs_seq_in_file_name``  | specifies the file name that results from running ``create_fixed_network_seq``, i.e.    |
|                           | the 'observation sequence definition' file.                                             |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``obs_seq_out_file_name`` | specifies the output file name containing the 'observation sequence', finally populated |
|                           | with (perfect?) 'observations'.                                                         |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``start_from_restart``    | When set to 'false', ``perfect_model_obs`` generates an arbitrary initial condition     |
|                           | (which cannot be guaranteed to be on the L63 attractor).                                |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``output_restart``        | When set to 'true', ``perfect_model_obs`` will record the model state at the end of     |
|                           | this integration in the file named by ``restart_out_file_name``.                        |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``restart_in_file_name``  | is ignored when 'start_from_restart' is 'false'.                                        |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``restart_out_file_name`` | if ``output_restart`` is 'true', this specifies the name of the file containing the     |
|                           | model state at the end of the integration.                                              |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``init_time_``\ *xxxx*    | the start time of the integration.                                                      |
+---------------------------+-----------------------------------------------------------------------------------------+
| ``output_interval``       | interval at which to save the model state.                                              |
+---------------------------+-----------------------------------------------------------------------------------------+

utilities_nml
~~~~~~~~~~~~~

+-------------------+-------------------------------------------------------------------------------------------------+
| namelist variable | description                                                                                     |
+===================+=================================================================================================+
| ``TERMLEVEL``     | When set to '1' the programs terminate when a 'warning' is generated. When set to '2' the       |
|                   | programs terminate only with 'fatal' errors.                                                    |
+-------------------+-------------------------------------------------------------------------------------------------+
| ``logfilename``   | Run-time diagnostics are saved to this file. This namelist is used by all programs, so the file |
|                   | is opened in APPEND mode. Subsequent executions cause this file to grow.                        |
+-------------------+-------------------------------------------------------------------------------------------------+

Executing ``perfect_model_obs`` will integrate the model 24,000 steps and output the resulting state in the file
``perfect_restart``. Interested parties can check the spinup in the ``True_State.nc`` file.

::


   $ perfect_model_obs

2. Generate a set of ensemble initial conditions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| The set of initial conditions for a 'perfect model' experiment is created in several steps. 1) Starting from the
  spun-up state of the model (available in ``perfect_restart``), run ``perfect_model_obs`` to generate the 'true state'
  of the experiment and a corresponding set of observations. 2) Feed the same initial spun-up state and resulting
  observations into ``filter``.
| The first step is achieved by changing a perfect_model_obs namelist parameter, copying ``perfect_restart`` to
  ``perfect_ics``, and rerunning ``perfect_model_obs``. This execution of ``perfect_model_obs`` will advance the model
  state from the end of the first 24,000 steps to the end of an additional 24,000 steps and place the final state in
  ``perfect_restart``. The rest of the namelists in ``input.nml`` should remain unchanged.

::


   &perfect_model_obs_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      obs_seq_in_file_name = "obs_seq.in",
      obs_seq_out_file_name = "obs_seq.out",
      start_from_restart = .true.,
      output_restart = .true.,
      restart_in_file_name = "perfect_ics",
      restart_out_file_name = "perfect_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_interval = 1 /

::


   $ cp perfect_restart perfect_ics
   $ perfect_model_obs

A ``True_State.nc`` file is also created. It contains the 'true' state of the integration.

Generating the ensemble
^^^^^^^^^^^^^^^^^^^^^^^

This step (#2 from above) is done with the program ``filter``, which also uses the Fortran90 namelist mechanism for
input. It is now necessary to copy the ``input.nml.filter_default`` namelist to ``input.nml`` or you may simply insert
the ``filter_nml`` namelist block into the existing ``input.nml``. Having the ``perfect_model_obs`` namelist in the
input.nml does not hurt anything. In fact, I generally create a single ``input.nml`` that has all the namelist blocks in
it. I simply copied the filter namelist block from ``input.nml.filter_default`` and inserted it into our ``input.nml``
for the following example.

::


   &perfect_model_obs_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      obs_seq_in_file_name = "obs_seq.in",
      obs_seq_out_file_name = "obs_seq.out",
      start_from_restart = .true.,
      output_restart = .true.,
      restart_in_file_name = "perfect_ics",
      restart_out_file_name = "perfect_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_interval = 1 /

   &filter_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      ens_size = 100,
      cov_inflate = 1.0,
      start_from_restart = .false.,
      output_restart = .true.,
      obs_sequence_in_name = "obs_seq.out",
      obs_sequence_out_name = "obs_seq.final",
      restart_in_file_name = "perfect_ics",
      restart_out_file_name = "filter_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_state_ens_mean = .true.,
      output_state_ens_spread = .true.,
      output_obs_ens_mean = .true.,
      output_obs_ens_spread = .true.,
      num_output_state_members = 20,
      num_output_obs_members = 20,
      output_interval = 1,
      num_groups = 1,
      outlier_threshold = -1.0 /

   &ensemble_manager_nml
      in_core = .true.,
      single_restart_file_in = .true.,
      single_restart_file_out = .true. /

   &assim_tools_nml
      filter_kind = 1,
      cutoff = 0.2,
      sort_obs_inc = .false.,
      cov_inflate = -1.0,
      cov_inflate_sd = 0.05,
      sd_lower_bound = 0.05,
      deterministic_cov_inflate = .true.,
      start_from_assim_restart = .false.,
      assim_restart_in_file_name =
   'assim_tools_ics',
      assim_restart_out_file_name =
   'assim_tools_restart',
      do_parallel = 0,
      num_domains = 1
      parallel_command = "./assim_filter.csh",
      spread_restoration = .false.,
      cov_inflate_upper_bound = 10000000.0,
      internal_outlier_threshold = -1.0 /

   &cov_cutoff_nml
      select_localization = 1 /

   &reg_factor_nml
      select_regression = 1,
      input_reg_file = "time_mean_reg"
      save_reg_diagnostics = .false.,
      reg_diagnostics_file = 'reg_diagnostics' /

   &obs_sequence_nml
      write_binary_obs_sequence = .false. /

   &obs_kind_nml
      assimilate_these_obs_types = 'RAW_STATE_VARIABLE'
   /

   &assim_model_nml
      write_binary_restart_files = .true. /

   &model_nml
      sigma = 10.0,
      r = 28.0,
      b = 2.6666666666667,
      deltat = 0.01,
      time_step_days = 0,
      time_step_seconds = 3600 /

   &utilities_nml
      TERMLEVEL = 1
      logfilename = 'dart_log.out' /

Only the non-obvious(?) entries for ``filter_nml`` will be discussed.

+------------------------------+--------------------------------------------------------------------------------------+
| namelist variable            | description                                                                          |
+==============================+======================================================================================+
| ``ens_size``                 | Number of ensemble members. 100 is sufficient for most of the L63 exercises.         |
+------------------------------+--------------------------------------------------------------------------------------+
| ``cutoff``                   | to limit the impact of an observation, set to 0.0 (i.e. spin-up)                     |
+------------------------------+--------------------------------------------------------------------------------------+
| ``cov_inflate``              | A value of 1.0 results in no inflation.(spin-up)                                     |
+------------------------------+--------------------------------------------------------------------------------------+
| ``start_from_restart``       | when '.false.', ``filter`` will generate its own ensemble of initial conditions. It  |
|                              | is important to note that the filter still makes use of ``perfect_ics`` by randomly  |
|                              | perturbing these state variables.                                                    |
+------------------------------+--------------------------------------------------------------------------------------+
| ``output_state_ens_mean``    | when '.true.' the mean of all ensemble members is output.                            |
+------------------------------+--------------------------------------------------------------------------------------+
| ``output_state_ens_spread``  | when '.true.' the spread of all ensemble members is output.                          |
+------------------------------+--------------------------------------------------------------------------------------+
| ``num_output_state_members`` | may be a value from 0 to ``ens_size``                                                |
+------------------------------+--------------------------------------------------------------------------------------+
| ``output_obs_ens_mean``      | when '.true.' Output ensemble mean in observation output file.                       |
+------------------------------+--------------------------------------------------------------------------------------+
| ``output_obs_ens_spread``    | when '.true.' Output ensemble spread in observation output file.                     |
+------------------------------+--------------------------------------------------------------------------------------+
| ``num_output_obs_members``   | may be a value from 0 to ``ens_size``                                                |
+------------------------------+--------------------------------------------------------------------------------------+
| ``output_interval``          | The frequency with which output state diagnostics are written. Units are in          |
|                              | assimilation times. Default value is 1 meaning output is written at every            |
|                              | observation time                                                                     |
+------------------------------+--------------------------------------------------------------------------------------+

The filter is told to generate its own ensemble initial conditions since ``start_from_restart`` is '.false.'. However,
it is important to note that the filter still makes use of ``perfect_ics`` which is set to be the
``restart_in_file_name``. This is the model state generated from the first 24,000 step model integration by
``perfect_model_obs``. ``Filter`` generates its ensemble initial conditions by randomly perturbing the state variables
of this state.

The arguments ``output_state_ens_mean`` and ``output_state_ens_spread`` are '.true.' so that these quantities are output
at every time for which there are observations (once a day here) and ``num_output_ens_members`` means that the same
diagnostic files, ``Posterior_Diag.nc`` and ``Prior_Diag.nc`` also contain values for 20 ensemble members once a day.
Once the namelist is set, execute ``filter`` to integrate the ensemble forward for 24,000 steps with the final ensemble
state written to the ``filter_restart``. Copy the ``perfect_model_obs`` restart file ``perfect_restart`` (the \`true
state') to ``perfect_ics``, and the ``filter`` restart file ``filter_restart`` to ``filter_ics`` so that future
assimilation experiments can be initialized from these spun-up states.

::


   $ filter
   $ cp perfect_restart perfect_ics
   $ cp filter_restart filter_ics

The spin-up of the ensemble can be viewed by examining the output in the netCDF files ``True_State.nc`` generated by
``perfect_model_obs`` and ``Posterior_Diag.nc`` and ``Prior_Diag.nc`` generated by ``filter``. To do this, see the
detailed discussion of matlab diagnostics in Appendix I.

3. Simulate a particular observing system
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Begin by using ``create_obs_sequence`` to generate an observation set in which each of the 3 state variables of L63 is
observed with an observational error variance of 1.0 for each observation. To do this, use the following input sequence
(the text including and after # is a comment and does not need to be entered):

============= ===========================================================
*4*           # upper bound on num of observations in sequence
*0*           # number of copies of data (0 for just a definition)
*0*           # number of quality control values per field (0 or greater)
*0*           # -1 to exit/end observation definitions
*-1*          # observe state variable 1
*0 0*         # time -- days, seconds
*1.0*         # observational variance
*0*           # -1 to exit/end observation definitions
*-2*          # observe state variable 2
*0 0*         # time -- days, seconds
*1.0*         # observational variance
*0*           # -1 to exit/end observation definitions
*-3*          # observe state variable 3
*0 0*         # time -- days, seconds
*1.0*         # observational variance
*-1*          # -1 to exit/end observation definitions
*set_def.out* # Output file name
============= ===========================================================

Now, generate an observation sequence definition by running ``create_fixed_network_seq`` with the following input
sequence:

============= ===============================================================
*set_def.out* # Input observation set definition file
*1*           # Regular spaced observation interval in time
*1000*        # 1000 observation times
*0, 43200*    # First observation after 12 hours (0 days, 12 \* 3600 seconds)
*0, 43200*    # Observations every 12 hours
*obs_seq.in*  # Output file for observation sequence definition
============= ===============================================================

4. Generate a particular observing system and true state
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An observation sequence file is now generated by running ``perfect_model_obs`` with the namelist values (unchanged from
step 2):

::


   &perfect_model_obs_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      obs_seq_in_file_name = "obs_seq.in",
      obs_seq_out_file_name = "obs_seq.out",
      start_from_restart = .true.,
      output_restart = .true.,
      restart_in_file_name = "perfect_ics",
      restart_out_file_name = "perfect_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_interval = 1 /

This integrates the model starting from the state in ``perfect_ics`` for 1000 12-hour intervals outputting synthetic
observations of the three state variables every 12 hours and producing a netCDF diagnostic file, ``True_State.nc``.

5. Filtering
~~~~~~~~~~~~

Finally, ``filter`` can be run with its namelist set to:

::


   &filter_nml
      async = 0,
      adv_ens_command = "./advance_ens.csh",
      ens_size = 100,
      cov_inflate = 1.0,
      start_from_restart = .true.,
      output_restart = .true.,
      obs_sequence_in_name = "obs_seq.out",
      obs_sequence_out_name = "obs_seq.final",
      restart_in_file_name = "filter_ics",
      restart_out_file_name = "filter_restart",
      init_time_days = 0,
      init_time_seconds = 0,
      output_state_ens_mean = .true.,
      output_state_ens_spread = .true.,
      output_obs_ens_mean = .true.,
      output_obs_ens_spread = .true.,
      num_output_state_members = 20,
      num_output_obs_members = 20,
      output_interval = 1,
      num_groups = 1,
      outlier_threshold = -1.0 /

The large value for the cutoff allows each observation to impact all other state variables (see Appendix V for
localization). ``filter`` produces two output diagnostic files, ``Prior_Diag.nc`` which contains values of the ensemble
mean, ensemble spread, and ensemble members for 12- hour lead forecasts before assimilation is applied and
``Posterior_Diag.nc`` which contains similar data for after the assimilation is applied (sometimes referred to as
analysis values).

Now try applying all of the matlab diagnostic functions described in the Matlab Diagnostics section.

Matlab® diagnostics
-------------------

The output files are netCDF files, and may be examined with many different software packages. We happen to use Matlab®,
and provide our diagnostic scripts in the hopes that they are useful.

The diagnostic scripts and underlying functions reside in two places: ``DART/diagnostics/matlab`` and ``DART/matlab``.
They are reliant on the public-domain `netcdf
toolbox <http://woodshole.er.usgs.gov/staffpages/cdenham/public_html/MexCDF/nc4ml5.html>`__ from
``http://woodshole.er.usgs.gov/staffpages/cdenham/public_html/MexCDF/nc4ml5.html`` as well as the public-domain `CSIRO
matlab/netCDF interface <http://www.marine.csiro.au/sw/matlab-netcdf.html>`__ from
``http://www.marine.csiro.au/sw/matlab-netcdf.html``. If you do not have them installed on your system and want to use
Matlab to peruse netCDF, you must follow their installation instructions. The 'interested reader' may want to look at
the ``DART/matlab/startup.m`` file I use on my system. If you put it in your ``$HOME/matlab`` directory, it is invoked
every time you start up Matlab.

| Once you can access the ``getnc`` function from within Matlab, you can use our diagnostic scripts. It is necessary to
  prepend the location of the ``DART/matlab`` scripts to the ``matlabpath``. Keep in mind the location of the netcdf
  operators on your system WILL be different from ours ... and that's OK.

.. container:: unix

   ::

      0[269]0 ghotiol:/<5>models/lorenz_63/work]$ matlab -nojvm

                                                   < M A T L A B >
                                       Copyright 1984-2002 The MathWorks, Inc.
                                           Version 6.5.0.180913a Release 13
                                                     Jun 18 2002

        Using Toolbox Path Cache.  Type "help toolbox_path_cache" for more info.
       
        To get started, type one of these: helpwin, helpdesk, or demo.
        For product information, visit www.mathworks.com.

      >> which getnc
      /contrib/matlab/matlab_netcdf_5_0/getnc.m
      >>ls *.nc

      ans =

      Posterior_Diag.nc  Prior_Diag.nc  True_State.nc


      >>path('../../../matlab',path)
      >>path('../../../diagnostics/matlab',path)
      >>which plot_ens_err_spread
      ../../../matlab/plot_ens_err_spread.m
      >>help plot_ens_err_spread

        DART : Plots summary plots of the ensemble error and ensemble spread.
                               Interactively queries for the needed information.
                               Since different models potentially need different 
                               pieces of information ... the model types are 
                               determined and additional user input may be queried.
       
        Ultimately, plot_ens_err_spread will be replaced by a GUI.
        All the heavy lifting is done by PlotEnsErrSpread.
       
        Example 1 (for low-order models)
       
        truth_file = 'True_State.nc';
        diagn_file = 'Prior_Diag.nc';
        plot_ens_err_spread

      >>plot_ens_err_spread

And the matlab graphics window will display the spread of the ensemble error for each state variable. The scripts are
designed to do the "obvious" thing for the low-order models and will prompt for additional information if needed. The
philosophy of these is that anything that starts with a lower-case *plot\_\ some_specific_task* is intended to be
user-callable and should handle any of the models. All the other routines in ``DART/matlab`` are called BY the
high-level routines.

+-------------------------------+-------------------------------------------------------------------------------------+
| Matlab script                 | description                                                                         |
+===============================+=====================================================================================+
| ``plot_bins``                 | plots ensemble rank histograms                                                      |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_correl``               | Plots space-time series of correlation between a given variable at a given time and |
|                               | other variables at all times in a n ensemble time sequence.                         |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_ens_err_spread``       | Plots summary plots of the ensemble error and ensemble spread. Interactively        |
|                               | queries for the needed information. Since different models potentially need         |
|                               | different pieces of information ... the model types are determined and additional   |
|                               | user input may be queried.                                                          |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_ens_mean_time_series`` | Queries for the state variables to plot.                                            |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_ens_time_series``      | Queries for the state variables to plot.                                            |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_phase_space``          | Plots a 3D trajectory of (3 state variables of) a single ensemble member.           |
|                               | Additional trajectories may be superimposed.                                        |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_total_err``            | Summary plots of global error and spread.                                           |
+-------------------------------+-------------------------------------------------------------------------------------+
| ``plot_var_var_correl``       | Plots time series of correlation between a given variable at a given time and       |
|                               | another variable at all times in an ensemble time sequence.                         |
+-------------------------------+-------------------------------------------------------------------------------------+

Bias, filter divergence and covariance inflation (with the l63 model)
---------------------------------------------------------------------

One of the common problems with ensemble filters is filter divergence, which can also be an issue with a variety of
other flavors of filters including the classical Kalman filter. In filter divergence, the prior estimate of the model
state becomes too confident, either by chance or because of errors in the forecast model, the observational error
characteristics, or approximations in the filter itself. If the filter is inappropriately confident that its prior
estimate is correct, it will then tend to give less weight to observations than they should be given. The result can be
enhanced overconfidence in the model's state estimate. In severe cases, this can spiral out of control and the ensemble
can wander entirely away from the truth, confident that it is correct in its estimate. In less severe cases, the
ensemble estimates may not diverge entirely from the truth but may still be too confident in their estimate. The result
is that the truth ends up being farther away from the filter estimates than the spread of the filter ensemble would
estimate. This type of behavior is commonly detected using rank histograms (also known as Talagrand diagrams). You can
see the rank histograms for the L63 initial assimilation by using the matlab script ``plot_bins``.

A simple, but surprisingly effective way of dealing with filter divergence is known as covariance inflation. In this
method, the prior ensemble estimate of the state is expanded around its mean by a constant factor, effectively
increasing the prior estimate of uncertainty while leaving the prior mean estimate unchanged. The program ``filter`` has
a namelist parameter that controls the application of covariance inflation, ``cov_inflate``. Up to this point,
``cov_inflate`` has been set to 1.0 indicating that the prior ensemble is left unchanged. Increasing ``cov_inflate`` to
values greater than 1.0 inflates the ensemble before assimilating observations at each time they are available. Values
smaller than 1.0 contract (reduce the spread) of prior ensembles before assimilating.

You can do this by modifying the value of ``cov_inflate`` in the namelist, (try 1.05 and 1.10 and other values at your
discretion) and run the filter as above. In each case, use the diagnostic matlab tools to examine the resulting changes
to the error, the ensemble spread (via rank histogram bins, too), etc. What kind of relation between spread and error is
seen in this model?

Synthetic observations
----------------------

Synthetic observations are generated from a \`perfect' model integration, which is often referred to as the \`truth' or
a \`nature run'. A model is integrated forward from some set of initial conditions and observations are generated as *y
= H(x) + e* where *H* is an operator on the model state vector, *x*, that gives the expected value of a set of
observations, *y*, and *e* is a random variable with a distribution describing the error characteristics of the
observing instrument(s) being simulated. Using synthetic observations in this way allows students to learn about
assimilation algorithms while being isolated from the additional (extreme) complexity associated with model error and
unknown observational error characteristics. In other words, for the real-world assimilation problem, the model has
(often substantial) differences from what happens in the real system and the observational error distribution may be
very complicated and is certainly not well known. Be careful to keep these issues in mind while exploring the
capabilities of the ensemble filters with synthetic observations.
