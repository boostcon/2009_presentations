# Hands-on C++0x Session Information

Start/HandsOnCXX0x - Last Change: 2009-05-01 15:36:40 by Doug Gregor 

This page provides some information for attendees of the "Hands-on Boost++0x" sessions, where we'll be using new C++0x features to upgrade and update Boost libraries for the new world of C++0x. These "hands-on" sessions will truly be hands-on, so we ask that attendees please follow the steps below before BoostCon begins so that we can skip the set-up and get hacking!

  1. Install GCC - GCC 4.3 provides a number of C++0x features as part of its [experimental C++0x mode](http://gcc.gnu.org/projects/cxx0x.html). Please follow the [GCC installation instructions](http://gcc.gnu.org/install/) to configure, build, and install GCC 4.3 or newer on your laptop and bring it in to the Hands-on C++0x sessions. We strongly recommend using the configure flag `--enable-languages=c++`, to only build the C and C++ compilers. Trust us: you don't want to wait for the Java compiler to build!
For Unix-like systems, those instructions should suffice. On Windows, you have two options:
    + For Cygwin users, the easiest approach is to install the gcc4-g++ package under the Devel section in the Cygwin installer.  That will install a late beta of g++-4.3.2.  If you want a true release version, the Cygwin wiki has a page about [how to build and install GCC 4.3.0](http://cygwin.wikia.com/wiki/How_to_install_GCC_4.3.0) (or newer) that you can follow (choose the latest 4.3.x you can find).
    + For Mingw users, there are binaries for GCC 4.3.3 available [here](http://www.tdragon.net/recentgcc/) that are relatively easy to install.

  2. Get Boost++0x - We'll all be working on a special branch of Boost where we can freely make C++0x-specific changes and share ideas and code. If you don't already have access to the Boost Subversion repository, please read about the [Boost Subversion repository](https://svn.boost.org/trac/boost/wiki/BoostSubversion) and send a note to doug.gregor@gmail.com to let me know you'd like an account. The Boost++0x branch is located in Subversion at https://svn.boost.org/svn/boost/sandbox/boost0x

  3. Read up about the major C++0x features we'll be using in Boost++0x. These features are:

 * [Rvalue references](http://cpp-next.com/archive/2009/08/want-speed-pass-by-value/) 
 * [Variadic templates](https://github.com/boostcon/2009_presentations/raw/master/tue/variadic.markdown) 
 * [Decltype](https://github.com/boostcon/2009_presentations/raw/master/tue/decltype.markdown)
 * [ExtendedSFINAE](https://github.com/boostcon/2009_presentations/raw/master/tue/SFINAE.markdown)