layout: post
title: RPM Build
date: 2013-5-13
categories: Linux
---

RPM build steps:

1. set up a directory hierarchy per the `rpmbuild` specifications
2. place sources and supplemental files in the proper locations in the hierarchy
3. create `spec` file
4. build the RPM

### step 1: build the hierarchy

Standard hierarchy contains five subdirectories:

- SOURCES(`%_sourcedir`): contains source files
- SPECS(`%_specdir`): contains spec files - one spece file per RPM you want to build
- BUILD(`%_builddir`): space to compile the sources
- BUILDROOT(`%_buildrootdir`): files are installed under here
- RPMS(`%_rpmdir`): contains binary PRM that the `rpmbuild` builds
- SRPMS(`%_srcrpmdir`): contains the source RPM built during the process

There at least need SOURCES and a spec file in SPECS.

### step 2: copy the sources and files to SOURCES

Copy the sources, ideally bundled as a tarball, into the SOURCES directory.
The name of sources tarball is `package-version.tar.gz` is better.

### step 3: create `spec` file

A `spec` file is nothing more than a text file with a special syntax.
But it is the key file for create rpm.
All configuration, commands and informations about create, install and uninstall rpms are defined in this file.

The simplest way to build rpm is:

~~~
> cd ~/rpmbuild/SPECS
> touch NAME.spec
~~~

`rpmbuild` will read `Name.spec` file and go throught in order the stages listed below:

- `%prep`(`%_sourcedir`->`%_builddir`)
- `%build`(`%_builddir`->`%_builddir`)
- `%check`(`%_builddir`->`%_builddir`)
- `%install`(`%_builddir`->`%_buildrootdir`)
- bin(`%_buildrootdir`->`%_rpmdir`)
- src(`%_sourcedir`->`%_srcrpmdir`)

##### `%prep`

This section describes how to unpack the compressed packages so that they can be built. Normally this section includes two macros:

- `%setup` macro is used to unpack the original sources.
- `%patch` macro is used to apply patches to the original source.

##### `%build`

This section describes how to configure and compile/build the files to be installed. Many programs use `configure` approach to config the sources, so:

- `%configure` macro is used to automatically invoke the correct options.

and use `make` to compile files.

##### `%install`

This section creates directories and copy relavant files from `%_builddir` to `%buildroot`. Normally, some variation of `make install` is performed here.


##### `%files`

This section declares which files and directories are owned by the package, and thus which files and directories will be placed into binary RPM.

- `%defattr` macro set the default file permissions, and is often found at the start of the`%files` section. Note that this is no longer necessary unless the permissions need to be altered.

##### Scriptlets

Scriptlets can be run:

- before(`%pre`) or after(`%post`) a package is installed
- before(`%preun`) or after(`%postun`) a package is uninstalled
- at the start(`%pretrans`) or end(`%posttrans`) of a transcation

The order of operations during the upgrade is:

1. Run the `%pre` of the RPM being installed
2. Install the files that RPM provides
3. Run the `%post` of RPM
4. Run the `%preun` of the old package
5. Delete any old files not overwritten by the newer version.
6. Run the `%postun` of the old package


### step4: build the RPM

```
rpmbuild -ba NAME.spec
```

### Reference

- http://www.ibm.com/developerworks/linux/library/l-rpm3/index.html
- https://fedoraproject.org/wiki/How_to_create_an_RPM_package
- http://www.redhat.com/mirrors/LDP/HOWTO/RPM-HOWTO/
- http://www.redhat.com/promo/summit/2008/downloads/pdf/Wednesday_130pm_Tom_Callaway_OSS.pdf
- http://rpm.org/max-rpm-snapshot/
