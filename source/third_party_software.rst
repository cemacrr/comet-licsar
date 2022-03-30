.. include:: links.rst

Third Party Software
====================

On the JASMIN systems third party software used for the LiCSAR processing is stored within::

  /gws/smf/j04/nceo_geohazards

This is SSD storage (`JASMIN Storage Types`_), which is more suitable for storing software than the slower larger volume storage areas, though the space is much more limited.

At present this area is 100G in size::

  Size  Used Avail Use% Mounted on
  100G   55G   46G  55% /gws/smf/j04/nceo_geohazards

Software includes `GAMMA`_, `LiCSBAS`_, `SNAPHU`_, an `Anaconda`_ environment and various other tools.

There is some use of `Environment Modules`_ for loading and unloading of software, and various versions of some packages are available.

CEMAC ARC Software
------------------

CEMAC manage various software installations on the ARC systems in Leeds, and could be a useful example to show some ideas which might help with managing the LiCSAR software.

The files are all within::

  /nobackup/cemac/software

The software is built via build scripts within::

  /nobackup/cemac/software/build

This is also where the required installation sources are stored.

The software is installed within the directories::

  /nobackup/cemac/software/apps
  /nobackup/cemac/software/compilers
  /nobackup/cemac/software/libraries

This directory structure was set up to match the structure used by the system software on the ARC systems, and may not be required for the LiCSAR software, where a single installation directory may be more suitable.

The Environment Modules files are stored within::

  /nobackup/cemac/software/modulefiles

The build scripts and module files are all stored in a GitHub repository at:

  https://github.com/cemac/arc/

An individual can add to their login files (``~/.bashrc``)::

  if [ -r /nobackup/cemac/cemac.sh ] ; then
    . /nobackup/cemac/cemac.sh
  fi

The software is then available for use via the ``module`` command.

JASMIN Considerations
---------------------

Unlike the ARC systems at Leeds, where all machines related to the system (e.g. ``arc4``) are all more or less the same and run the same version of the operating system, the systems available to users at JASMIN differ.

In the past, the machines have been mixture of different versions of the operating system and this could happen again in the future. It may therefore be necessary to include the operating system version in installation directory / module loading paths, for example, if compiling GAMMA from source on a CentOS / RedHat 7 machine, it may be sensible to install to a path similar to::

  /gws/smf/j04/nceo_geohazards/software/apps/el7/gamma/20220101

If JASMIN machines were upgraded machines to a newer version of the operating system, or a different operating system altogether, the software could also be built for that operating system, and the process which sets up the ``MODULEPATH`` for a user could automatically adjust settings based on the machine in use.

Another thing to be aware of at JASMIN is that some machines don't have access to the network storage areas (such as the login nodes) and may have to be treated differently. Again, this could be handled by the process which sets up the Environment Modules variables, by checking if things are readable or checking the system host name.

Build Scripts
-------------

Keeping track of the installation process for each version of each piece of software can be incredibly valuable, not only for keeping a useful record of how the software was built and saving time when installing an updated version of a piece of software, it may also make it easier to deploy the software on a new system in the future if required.

Storing the source files with the build scripts helps to keep all required files in the same place, and can also be very useful if rebuilding software is required in the future, as older installation / source files can sometimes become unavailable.

The build process for some software may be as simple as extracting some files and moving them in to place, or it may be a more complex process of building from source and building various dependencies, but it is usually possible to script the process in some way.

Installation Directories
------------------------

Ideally, once a particular version of a piece of software has been installed, it should be made read only, never changed, and kept indefinitely.

If a particular version of a piece of software needed to be rebuilt or changed for any reason, a new *build version* could be created.

For example, if GAMMA/20220101 was already installed, but a rebuild of this version was required, for example to apply a patch, the updated version could be built and installed in separate directories, so you could end up with something like::

  /gws/smf/j04/nceo_geohazards/software/apps/el7/gamma/20220101/1
  /gws/smf/j04/nceo_geohazards/software/apps/el7/gamma/20220101/2

Where ``1`` and ``2`` are the build versions.

There are different ways in which loading software via Environment Modules could be handled, and it would be possible for a user to ``module load gamma`` and always get the latest version, while also being able to load previous versions by specifying the version of the software.

As the space in the software area is limited, older versions of different pieces of software could be moved to the slower / larger volume storage and replaced with symlinks, so the older versions are still accessible if and when required.

Environment Modules
-------------------

Keeping Environment Module files tidy and easy to understand and use isn't always simple, taking a look at the modules provided by default on the JASMIN systems will show how things can become messy and difficult to use.

For any software which has multiple versions, it is possible to set the default version by placing a ``.version`` file in the directory.

For example, a directory containing module files for GAMMA may contain::

  20210101
  20220101
  .version

Where the ``.version`` file contains::

  #%Module1.0
  set ModulesVersion "20210101"

So when ``module load gamma`` is run, the ``20210101`` version is loaded by default.

It is also possible to create Environment Modules named ``old``, ``testing``, or similar, and when these modules are loaded, the ``MODULEPATH`` variable is expanded to include older or testing versions of software. This can help keep the default list of software tidy, while allowing access to additional versions of software.

Anaconda Considerations
-----------------------

Creating a list of installed packages in an Anaconda environment is quite straight forward, and is a useful way of keeping a record of installed packages.

In theory, it should be possible to use this list to create an identical environment at a later date, but in practice this is rarely simple and creation of a new environment fails.

When creating an environment, it is often best to specify names of required packages, create the environment and then test everything works as required.

The `Mamba`_ tool is a "faster" version of the ``conda`` command, and can greatly reduce the time required to set up a new environment.

Creation of a new Anaconda installation may be similar to::

  wget -O ${MINICONDA_FILE} \
    "https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh"
  chmod 755 ${MINICONDA_FILE}
  ./${MINICONDA_FILE} -b -p ${INSTALL_DIR}
  \rm ${MINICONDA_FILE}
  . ${INSTALL_DIR}/etc/profile.d/conda.sh
  conda update -y -n base -c defaults conda
  conda install -y -n base -c conda-forge mamba
  mamba install matplotlib notebook h5py gdal==2.1.0

In this example, ``gdal`` version ``2.1.0`` is explicitly required, whereas other requested packages should be the most recent available that don't conflict with any other requested packages.

Linking Software Versions With Data Versions
--------------------------------------------

At present, I don't believe the LiCSAR outputs have any kind of version information.

One step towards this may be creating versions of the LiCSAR software tools.

For example, the LiCSAR tools version 1.0.0 may include::

  condalics/20220101
  gamma/20220101
  licsbas/1.5.11

These packages could be loaded by creating a ``LiCSAR`` (or similarly named) Environment Modules file.

It could also be possible that in the future the version of the LiCSAR tools is directly linked to the version of the output data, if desired. This could mean that version ``x`` of the LiCSAR outputs were created with version ``x`` of the LiCSAR tools.

A good, and hopefully simple, first step would be to record the versions of the software used to create data when generating new LiCSAR output files, for example in the output directory for a processed pair of interferograms, some kind of meta data file in plain text, JSON, both or something else, could be created to include software details, along with any other required metai data.
