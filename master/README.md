VIPS WIN32
==========

We use jhbuilder to git, mingw to compile, and good old zip to package the
vips source code for WIN32.

[JHBuild] (http://live.gnome.org/Jhbuild)
-----------------------------------------

JHBuild is a tool designed to ease building collections of source packages,
called “modules”. JHBuild uses “module set” files to describe the
modules available to build. The “module set” files include dependency
information that allows JHBuild to discover what modules need to be built
and in what order.

The jhbuild in Ubuntu is fine.

[MinGW](http://www.mingw.org/)
------------------------------

MinGW, a contraction of "Minimalist GNU for Windows", is a minimalist
development environment for native Microsoft Windows applications.

MinGW provides a complete Open Source programming tool set which is suitable
for the development of native MS-Windows applications, and which do not depend
on any 3rd-party C-Runtime DLLs. (It does depend on a number of DLLs provided
by Microsoft themselves, as components of the operating system; most notable
among these is MSVCRT.DLL, the Microsoft C runtime library. Additionally,
threaded applications must ship with a freely distributable thread support
DLL, provided as part of MinGW itself).

Debian has two main mingw packages. The more modern one, gcc-mingw-w64, is
regular gcc, built as a cross-compiler, with the normal gcc tools all able to
build win32 binaries. We use this. 

PREREQUISITES
=============
[Ubuntu Desktop] (http://www.ubuntu.com/desktop/get-ubuntu/download)
- This doesn't mean you can't get the process to work on anything else. This
is simply what we are using and know to work. Tested on 11.10, 12.04LTS, and 13.10. 
Older versions of build-win32 work on older versions of Ubuntu.

OPTIONAL
========
[VMware Player] (http://www.vmware.com/products/player/) - Bubba runs his
Ubuntu Desktop in a VMWare Player on a Windows 7 Ultimate x64 Desktop host.

[DEPENDENCIES] (http://xkcd.com/754/)
==============
It is possible that you already have some of these installed on your Ubuntu
Desktop; however, it is not likely that you have all of them. Better safe
than sorry, install them all. You might even want to update the whole kit,
just for the heck of it.

Build/Tool Related Dependencies
-------------------------------
    sudo apt-get install build-essential \
	wine \
	gcc-mingw-w64-x86-64 \
	jhbuild \
	autoconf \
	automake1.4 \
	automake1.9 \
	autotools-dev \
	docbook-utils \
	docbook2x \
	gtk-doc-tools \
	nasm \
	bison \
	flex

Library Dependencies
--------------------
    sudo apt-get install libatk1.0-0 \
	libatk1.0-dev \
	libglib2.0-0 \
	libglib2.0-dev \
	libgtk2.0-0 \
	libgtk2.0-dev \
	libglade2-0 \
	libglade2-dev \
	libgsf-1-114 \
	libgsf-gnome-1-dev \
	libpango1.0-0 \
	libpango1.0-dev \
	libcairo2 \
	libcairo2-dev \
	libexpat1 \
	libexpat1-dev \
	libfontconfig1 \
	libfontconfig1-dev \
	libfreetype6 \
	libfreetype6-dev \
	gettext \
	libpng12-0 \
	libpng12-dev \
	libxml2 \
	libxml2-dev \
	tango-icon-theme \
	libcxxtools6 \
	libcxxtools-dev \
	zlib1g \
	zlib1g-dev \
	zlibc 

These are Ubuntu binaries and of course we will be building a Windows
binary. However, some of the packages we build are not very good at
cross-compiling and builds can fail unless they can find a native library as
well.

CONFIGURATION
=============
You will need to first check out this repository, if you haven't already. 

git This
--------
	mkdir ~/dev
	cd ~/dev
	git clone git://github.com/jcupitt/build-win32.git
	cd build-win32/7.38

Check versions
--------------
The variables.sh script defines some common variables (eg. vips version
number) used by the other scripts. Check they are all OK, and that the version
numbers in vips.modules are up to date too.

GNOME win32 Packages
--------------------
As we are building a win32 executable, we need some DLLs to link against,
and the GNOME project kindly provides us with a large number of these ready
to use! 

Run this script to create some directories for zips, tarballs and the
build tree and download all the zips you need from Gnome:

	./get-win32-packages.sh

If you desire to modify the packages used, just open up the
script and edit the list near the end. This is completely optional though,
as the ones checked out "should" work just fine for you needs.

After downloading, run the unpack script to unzip the files to your build
area:

	./unpack.sh

PATCHING
========

glib-dev_2.28.8-1_win32.zip creates a file called inst/bin/glib-mkenums. The
first line of this file needs changing from:

	#! /bin/perl

to 

	#! /usr/bin/perl

JHBUILD VERIFICATION
====================
We just want to make sure that jhbuild has everything it needs. If all steps
have been properly followed up to this point, this should be a no brainer.

	jhbuild --file=jhbuildrc sanitycheck

This will most likely complain that it couldn't find automake-1.8 which
is fine, we didn't install that. If it complains about anything else,
let me know.

BUILD
=====
Check that libvips is being built WITHOUT cfitsio, and run:

	jhbuild --file=jhbuildrc --moduleset=vips.modules build libvips

PACKAGE
=======
	./package-vipsdev.sh

BUILD NIP2
==========
Now turn on cfitsio and run:

	jhbuild --file=jhbuildrc --moduleset=vips.modules build nip2

	./package-nip2.sh

UPLOAD YOUR PACKAGE
===================
Assuming everything has worked perfectly up to this point, you will find
vips-dev-7.32.x.zip all packaged up and ready to go. You might upload it
to your favorite server via scp like this:

	scp vips-dev-7.32.x.zip <YOURID>@<YOURSERVER>:/your/favorite/directory

	scp nip2-7.32.x-setup.exe <YOURID>@<YOURSERVER>:/your/favorite/directory

Many servers will block direct downlods of .exe files. You might need to put
the .exe in a zip file.

CLEAN UP
========
It is always good to clean up after yourself. Be careful though, this command
will delete the package you just created! You did upload it to your favorite
server didn't you?

	./clean.sh

You'll need to run the unpack script again if you clean up. You won't need to
redownload the zips though.

OTHER NOTES
===========

Patching
--------
A primary reason one might desire to build their own executable, you simply
want to make a few changes to the code, or otherwise control how it was
compiled. 

First, build as described above:

	./unpack.sh

	jhbuild --file=jhbuildrc --moduleset=vips.modules build libvips

Now go to checkout/vips-7.32.0 and make any source changes you want. Build
again to compile your changes.

	jhbuild --file=jhbuildrc --moduleset=vips.modules build libvips

And package your new version.

	./package-vipsdev.sh

I suggest you rename your zip to avoid confusion. Call it something like
vips-dev-7.32.0-rob1.zip.

- remove strcasecmp() hacks from cfitsio, see "eval_l.c"
