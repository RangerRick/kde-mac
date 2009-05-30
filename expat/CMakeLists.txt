project (expat)

set(EXPAT_VERSION_MAJOR 2 CACHE INT "Major Expat version number" FORCE)
set(EXPAT_VERSION_MINOR 0 CACHE INT "Minor Expat version number" FORCE)
set(EXPAT_VERSION_PATCH 1 CACHE INT "Release Expat version number" FORCE)
set(EXPAT_VERSION_STRING "${EXPAT_VERSION_MAJOR}.${EXPAT_VERSION_MINOR}.${EXPAT_VERSION_PATCH}" CACHE STRING "Expat version string" FORCE)

# based on the libtool compatibility version info from autoconf
set(DYLIB_COMPATIBILITY_VERSION 7.0.0)
set(DYLIB_CURRENT_VERSION 7.2.0)

cmake_minimum_required(VERSION 2.6)

# compile in debug mode
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING
      "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif(NOT CMAKE_BUILD_TYPE)

set (LIB_SUFFIX "" CACHE STRING "Define suffix of directory name (32/64)" )
set (LIB_DESTINATION "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}" CACHE STRING "Library directory name" FORCE)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH  TRUE)
set(CMAKE_INSTALL_RPATH "${LIB_DESTINATION}" )
if(APPLE)
   set(CMAKE_INSTALL_NAME_DIR ${LIB_DESTINATION})
endif(APPLE)

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckTypeSize)
include(TestBigEndian)

set(CPACK_PACKAGE_VENDOR "Expat")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Expat XML parsing library")
set(CPACK_PACKAGE_VERSION_MAJOR ${EXPAT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${EXPAT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${EXPAT_VERSION_PATCH})
if (APPLE)
	set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.txt")
	set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt")
	set(CPACK_SET_DESTDIR ON)
	set(CPACK_PACKAGE_RELOCATABLE OFF)
endif (APPLE)

include(CPack)

check_function_exists(bcopy HAVE_BCOPY)
check_function_exists(memmove HAVE_MEMMOVE)
check_include_files(unistd.h HAVE_UNISTD_H)

configure_file(expat_config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/expat_config.h)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib)
add_definitions(-DHAVE_EXPAT_CONFIG_H)

# libexpat
set(expat_SRCS
	lib/xmlparse.c
	lib/xmlrole.c
	lib/xmltok.c
	lib/xmltok_impl.c
	lib/xmltok_ns.c
)
add_library(expat SHARED ${expat_SRCS})
set_target_properties(expat PROPERTIES
	VERSION 1.5.2
	SOVERSION 1
)

# xmlwf
set(xmlwf_SRCS
	xmlwf/xmlwf.c
	xmlwf/xmlfile.c
	xmlwf/codepage.c
	xmlwf/unixfilemap.c
)
add_executable(xmlwf ${xmlwf_SRCS})
target_link_libraries(xmlwf expat)

# examples
add_executable(elements examples/elements.c)
target_link_libraries(elements expat)

add_executable(outline examples/outline.c)
target_link_libraries(outline expat)

# install
install(TARGETS expat xmlwf
	LIBRARY DESTINATION ${LIB_DESTINATION}
	RUNTIME DESTINATION bin
	ARCHIVE DESTINATION ${LIB_DESTINATION}
)

install(FILES
	lib/expat.h
	lib/expat_external.h
	DESTINATION include
)
install(FILES
	doc/xmlwf.1
	DESTINATION share/man/man1
)
