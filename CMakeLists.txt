# Copyright (C) 2007-2011 LuaDist.
# Created by David Manura
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.
#
# Note: This is based in part on the configure.in and Makefile.am in zeromq2-1, which has
# its own license.

project ( zeromq2-1 C CXX )
cmake_minimum_required ( VERSION 2.8 )
include ( dist.cmake )
include ( CheckLibraryExists )
include ( CheckFunctionExists )
include ( CheckCSourceCompiles )

set ( LIBS )

macro ( my_check_library_exists library function location variable )
  check_library_exists ( "${library}" "${function}" "${location}" "${variable}" )
  if ( ${variable} )
    set ( LIBS ${LIBS} ${library} )
  endif ()
endmacro ()

my_check_library_exists ( pthread pthread_create "" HAVE_LIBPTHREAD )

# :TODO: not fully implemented
set ( libzmq_pedantic 1 )       # set pedantic
set ( libzmq_werror 1 )         # compiling with -Werror except OSX.  
set ( libzmq_dso_visibility 1 ) # default use DSO visibility
set ( libzmq_on_mingw32 0 )     # whether mingw

# Set some default features required by 0MQ code.
add_definitions ( -D_REENTRANT -D_THREAD_SAFE )  # CPPFLAGS

if ( APPLE )
  add_definitions ( -D_DARWIN_C_SOURCE )
  set ( libzmq_pedantic 0 )
  set ( libzmq_werror 0 )
  set ( ZMQ_HAVE_OSX  1 ) # [Have DarwinOSX OS]
  check_cxx_compiler_flag ( -Wno-uninitialized _OK )
  if ( _OK )
    add_definitions ( -Wno-uninitialized )
  endif ()
elseif ( UNIX ) # :TODO: actually only *linux*)
  if (0) # :TODO: GNU
    add_definitions ( -D_GNU_SOURCE )
  endif ()
  set ( ZMQ_HAVE_LINUX 1 )
  my_check_library_exists ( rt sem_init "" HAVE_LIBRT )
  my_check_library_exists ( uuid uuid_generate "" HAVE_LIBUUID )
  if ( NOT HAVE_LIBUUID )
    message ( FATAL_ERROR "cannot link with -luuid, install uuid-dev." )
  endif ()
  if (0) # :TODO: host_os in *android*)
    set ( ZMQ_HAVE_ANDROID 1 ) # [Have Android OS]
  endif ()
elseif ( 0 ) # :TODO: *solaris*
  add_definitions ( -D_PTHREADS )
  set ( ZMQ_HAVE_SOLARIS 1 )
  my_check_library_exists ( socket socket "" HAVE_LIBSOCKET )
  my_check_library_exists ( nsl gethostbyname "" HAVE_LIBNSL )
  my_check_library_exists ( rt sem_init "" HAVE_LIBRT )
  my_check_library_exists ( uuid uuid_generate "" HAVE_UUID )
  if ( NOT HAVE_UUID )
    message ( FATAL_ERROR "cannot link with -luuid, install uuid-dev" )
  endif ()
  check_c_source_compiles ( "
     #include <atomic.h>
     uint32_t value; atomic_cas_32 (&value, 0, 0); return 0;", _OK )
  # Solaris 8 does not have atomic operations exported to user space.
  if ( NOT _OK )
    set ( ZMQ_FORCE_MUTEXES 1 ) # [Force to use mutexes])
  endif ()
elseif ( 0 ) # :TODO: *freebsd*
  add_definitions ( -D__BSD_VISIBLE )
  set ( ZMQ_HAVE_FREEBSD 1 )
elseif ( 0 ) # :TODO: *netbsd*
  add_definitions ( -D_NETBSD_SOURCE )
  set ( ZMQ_HAVE_NETBSD 1 )
  check_cxx_source_compiles ( "
    #include <atomic.h>
    uint32_t value; atomic_cas_32 (&value, 0, 0); return 0; ", _OK)
  if ( NOT _OK )
    set ( ZMQ_FORCE_MUTEXES 1 ) # [Force to use mutexes])
  endif ()
elseif ( 0 ) # :TODO: *openbsd*
  add_definitions ( -D_BSD_SOURCE )
  set ( ZMQ_HAVE_OPENBSD 1 )
elseif ( 0 ) # :TODO: *nto-qnx*
  set ( libzmq_pedantic 0 )
  set ( ZMQ_HAVE_QNXNTO 1 )
  my_check_library_exists ( socket socket "" HAVE_LIBSOCKET )
  my_check_library_exists ( crypto RAND_bytes "" HAVE_LIBCRYPTO )
elseif ( 0 ) # :TODO: *aix*
  set ( ZMQ_HAVE_AIX 1 )
  my_check_library_exists ( crypto RAND_bytes "" HAVE_LIBCRYPTO )
elseif ( 0 ) # :TODO: *hpux*
  add_definitions ( -D_POSIX_C_SOURCE=200112L )
  set ( ZMQ_HAVE_HPUX 1 ) # [Have HPUX OS])
  my_check_library_exists ( rt sem_init HAVE_LIBRT )
  if (0) # :TODO:....with_dce
    my_check_library_exists ( dcekt, uuid_create )
  else ()
    my_check_library_exists ( crypto RAND_bytes )
  endif ()
elseif ( WINDOWS )  # :TODO: actually only *mingw32*
  set ( ZMQ_HAVE_WINDOWS 1 ) # [Have Windows OS])
  set ( ZMQ_HAVE_MINGW32 1 ) # [Have MinGW32])
  check_include_file ( windows.h HAVE_WINDOWS_H )
  my_check_library_exists ( ws2_32   main "" HAVE_LIBWS2_32   )
  my_check_iibrary_exists ( rpcrt4   main "" HAVE_LIBRPCRT4   )
  my_check_library_exists ( iphlpapi main "" HAVE_LIBIPHLPAPI )
  if ( NOT HAVE_LIBWS2_32 )
    message ( FATAL_ERROR "cannot link with ws2_32.dll" )
  endif ()
  if ( NOT HAVE_LIBRPCRT4 )
    message ( FATAL_ERROR "cannot link with rpcrt4.dll" )
  endif ()
  if ( NOT HAVE_LIBIPHLPAPI )
    message ( FATAL_ERROR "cannot link with iphlpapi.dll" )
  endif ()
  check_cxx_compiler_flag ( -Wno-long-long _OK )  # mingw32 defines __int64_t as long long
  if ( _OK )
    add_definitions ( -Wno-long-long )
  endif ()
  set ( libzmq_on_mingw32 1 )
  set ( libzmq_dso_visibility 0 )
#  if .... enable_static
#    message ( FATAL_ERROR "Building static libraries is not supported under MinGW32" )
#  endif ()
elseif ( CYGWIN )
  add_definitions ( -D_GNU_SOURCE ) # Define on Cygwin to enable all library features
  set ( ZMQ_HAVE_CYGWIN 1 ) # [Have Cygwin])
  # Cygwin provides libuuid as part of the e2fsprogs package, and somewhat
  # uselessly installs the library in /usr/lib/e2fsprogs
  my_check_library_exists ( uuid uuid_generate "-L/usr/lib/e2fsprogs" HAVE_LIBUUID )
  if ( NOT HAVE_LIBUUID )
    message ( FATAL_ERROR "cannot link with -luuid, install the e2fsprogs package" )
  endif ()
#  if .... enable_static
#    message ( FATAL_ERROR "Building static libraries is not supported under Cygwin" )
#  endif ()
else ()
  message ( FATAL_ERROR "unsupported system")
endif ()


check_function_exists ( clock_gettime HAVE_CLOCK_GETTIME )

# Note: maybe variables created by configure.in in platform.hpp
# are not actually used in the sources, so we omit those checks here.

check_c_source_compiles ( "
#include <sys/types.h>
#include <sys/socket.h>
int main (int argc, char *argv []) { int s = socket (PF_INET, SOCK_STREAM | SOCK_CLOEXEC, 0); return (s == -1); }"
HAVE_SOCK_CLOEXEC )


configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/platform.cmakein
               ${CMAKE_CURRENT_BINARY_DIR}/src/platform.hpp @ONLY)

if ( BUILD_PGM ) # :TODO:
  add_subdirectory ( foreign/openpgm )
endif ()

enable_testing()

add_subdirectory ( src )
#add_subdirectory ( doc )
add_subdirectory ( perf )
add_subdirectory ( tests )


install_header ( include/zmq.h include/zmq.hpp include/zmq_utils.h )
install_doc ( AUTHORS COPYING COPYING.LESSER INSTALL MAINTAINERS NEWS README )
# :TODO: install tests / perf?
