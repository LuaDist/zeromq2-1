include_directories ( ${CMAKE_CURRENT_SOURCE_DIR}/../include )
include_directories ( ${CMAKE_CURRENT_BINARY_DIR} )

add_executable ( local_lat local_lat.cpp )
target_link_libraries ( local_lat libzmq ${LIBS} )

add_executable ( remote_lat remote_lat.cpp )
target_link_libraries ( remote_lat libzmq ${LIBS} )

add_executable ( local_thr local_thr.cpp )
target_link_libraries ( local_thr libzmq ${LIBS} )

add_executable ( remote_thr remote_thr.cpp )
target_link_libraries ( remote_thr libzmq ${LIBS} )

add_executable ( inproc_lat inproc_lat.cpp )
target_link_libraries ( inproc_lat libzmq ${LIBS} )

add_executable ( inproc_thr inproc_thr.cpp )
target_link_libraries ( inproc_thr libzmq ${LIBS} )
