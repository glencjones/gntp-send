cmake_minimum_required( VERSION 3.27 )

project( growl VERSION 1.0.3 DESCRIPTION "C++/C GNTP libs and cli " )

include(GNUInstallDirs)
find_package(PkgConfig)

option(WITH_STATIC    "Build static growl library"           OFF)

include_directories ( ./headers )

if( WIN32 )
  link_libraries ( ws2_32 )
endif( WIN32 )

set( GROWL_NTP_C_SRC
        source/growl.c
        source/tcp.c
        source/md5.c )

add_library( growl SHARED  ${GROWL_NTP_C_SRC} )
set_target_properties( growl PROPERTIES COMPILE_FLAGS "-DGROWL_DLL" )
set_target_properties( growl PROPERTIES PUBLIC_HEADER headers/growl.h)
set_target_properties( growl PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR} )

if( WITH_STATIC )
  add_library( growl-static  STATIC ${GROWL_NTP_C_SRC} )
  target_sources( growl-static PRIVATE ${GROWL_NTP_C_SRC})
  set_target_properties( growl-static PROPERTIES COMPILE_FLAGS "-DGROWL_DLL" )
  set_target_properties( growl-static PROPERTIES PUBLIC_HEADER headers/growl.h)
  set_target_properties( growl-static PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR} )
endif( WITH_STATIC )

add_library( growl++ SHARED source/growl++.cpp )
  target_sources( growl++ PRIVATE ${GROWL_NTP_C_SRC})
target_link_libraries ( growl++  growl)
set_target_properties( growl++ PROPERTIES COMPILE_FLAGS "-DGROWL_CPP_DLL" )
set_target_properties( growl++ PROPERTIES PUBLIC_HEADER headers/growl++.hpp)
set_target_properties( growl++ PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR} )

if( WITH_STATIC )
  add_library( growl++-static STATIC source/growl++.cpp )
  target_sources( growl++-static PRIVATE ${GROWL_NTP_C_SRC})
  target_link_libraries ( growl++-static growl)
  set_target_properties( growl++-static PROPERTIES COMPILE_FLAGS "-DGROWL_CPP_DLL" )
  set_target_properties( growl++-static PROPERTIES PUBLIC_HEADER headers/growl++.hpp)
  set_target_properties( growl++-static PROPERTIES VERSION ${PROJECT_VERSION} SOVERSION ${PROJECT_VERSION_MAJOR} )
endif( WITH_STATIC )

add_executable( gntp-send source/gntp-send.c )
target_sources( gntp-send PRIVATE ${GROWL_NTP_C_SRC})


option(BUILD_PKGCONFIG "Create and install pkgconfig" ON)
if( BUILD_PKGCONFIG )
  configure_file ("${CMAKE_CURRENT_SOURCE_DIR}/package.pc.in" "${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}++.pc" @ONLY)
  install (FILES "${CMAKE_CURRENT_BINARY_DIR}/lib${PROJECT_NAME}++.pc" DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
endif ()

if( WITH_STATIC)
  INSTALL(
    TARGETS gntp-send DESTINATION ${CMAKE_INSTALL_BINDIR}
    TARGETS growl
    TARGETS growl++
    TARGETS growl-static
    TARGETS growl++-static
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )
else()
  INSTALL(
    TARGETS gntp-send DESTINATION ${CMAKE_INSTALL_BINDIR}
    TARGETS growl
    TARGETS growl++
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )
endif( WITH_STATIC )
