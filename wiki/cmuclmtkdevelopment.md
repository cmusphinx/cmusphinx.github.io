---
layout: page 
---
# CMUCLMTK Development

### Check out the code:

	
	https://cmusphinx.svn.sourceforge.net/svnroot/cmusphinx/trunk/cmuclmtk

The repository also contains binaries for Windows and Linux. Binaries are stored by OS, in bin/x86-nt/ and bin/x86-linux/.
### Configure and compile in linux:

	
	./autogen.sh && make && make install


Optionally you can use options to pass to autogen, for example ''--prefix=/somewhere''

### Configure and compile for Windows:

You should do your development under[Cygwin](http://cygwin.com) since the makefiles are autobuild-oriented. Be sure to download the developer and mingw components. You want to compile with mingw since this will give you Windows-native binaries and will therefore be portable from computer to computer.

	
	aclocal && autoheader && automake --add-missing --copy && autoconf
	./configure --enable-mingw CFLAGS="-g -Wall -O0" CXXFLAGS="-g -Wall -O0"
	make && make install


----
### For Development on Linux

Insure that binaries are created on both Linux and Windows platforms.

*   build with autoconf
*   commit
*   do an update from a windows box with cygwin
*   compile under cygwin
*   commit again

### For Development on Windows

*  build with cygwin
*  commit
*  do an update on a linux box
*  build with autoconf
*  commit again

If you're working on a filesystem shared by linux and windows then you can skip the middle commit and update steps.

### Future Release Plans

There has not been an official release of the toolkit since the one put out by Cambridge.  We intend to make one very soon.  Since the toolkit itself has not been fundamentally updated aside from the 32-bit word ID change, this can be considered version 2.1 of the toolkit.  Nonetheless, there will be many significant updates:


*  Perl scripts for easy language model construction from various source texts (DONE)

*  Dictionary building support for English using Festival (DONE)

*  Chinese segmentation

*  Chinese pronunciation generation

Successive releases will contain more fundamental changes to the toolkit.  Mainly, we intend to add support for modified Kneser-Ney smoothing.
