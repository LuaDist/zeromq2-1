# Copyright (C) 2007-2011 LuaDist.
# Created by David Manura
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

include_directories ( ${CMAKE_CURRENT_SOURCE_DIR}/../include )
include_directories ( ${CMAKE_CURRENT_BINARY_DIR}/../src )

add_executable ( test_pair_inproc test_pair_inproc.cpp testutil.hpp )
add_executable ( test_pair_tcp test_pair_tcp.cpp testutil.hpp )
add_executable ( test_reqrep_inproc test_reqrep_inproc.cpp testutil.hpp )
add_executable ( test_reqrep_tcp test_reqrep_tcp.cpp testutil.hpp )
add_executable ( test_hwm test_hwm.cpp )

target_link_libraries ( test_pair_inproc libzmq ${LIBS} )
target_link_libraries ( test_pair_tcp libzmq ${LIBS} )
target_link_libraries ( test_reqrep_inproc libzmq ${LIBS} )
target_link_libraries ( test_reqrep_tcp libzmq ${LIBS} )
target_link_libraries ( test_hwm libzmq ${LIBS} )

add_test ( test_pair_inproc  test_pair_inproc )
add_test ( test_pair_tcp test_pair_tcp )
add_test ( test_reqrep_inproc test_reqrep_inproc )
add_test ( test_reqrep_tcp test_reqrep_tcp )
add_test ( test_hwm test_hwm )

if ( NOT WINDOWS )
  add_executable ( test_shutdown_stress test_shutdown_stress.cpp )
  add_executable ( test_pair_ipc test_pair_ipc.cpp testutil.hpp )
  add_executable ( test_reqrep_ipc test_reqrep_ipc.cpp testutil.hpp )
  
  target_link_libraries ( test_shutdown_stress libzmq ${LIBS} )
  target_link_libraries ( test_pair_ipc libzmq ${LIBS} )
  target_link_libraries ( test_reqrep_ipc libzmq ${LIBS} )
  
  add_test ( test_shutdown_stress test_shutdown_stress )
  add_test ( test_pair_ipc test_pair_ipc )
  add_test ( test_reqrep_ipc test_reqrep_ipc )
endif ()

