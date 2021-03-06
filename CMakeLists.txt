PROJECT(SeisComP)


# Global options (see below):
# - SC_GLOBAL_PYTHON_WRAPPER
# - SC_GLOBAL_PYTHON_WRAPPER_NUMPY


CMAKE_MINIMUM_REQUIRED(VERSION 2.8.12 FATAL_ERROR)
CMAKE_POLICY(SET CMP0003 NEW)
CMAKE_POLICY(SET CMP0005 NEW)
#SET(CMAKE_DEBUG_POSTFIX _d)
SET(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
SET(CMAKEDEFINE "#cmakedefine")

#OPTION(SHARED_LIBRARIES "Build shared libraries" ON)
SET(SHARED_LIBRARIES 1)
ADD_DEFINITIONS(-DSC_ALL_SHARED)
IF (WIN32)
	ADD_DEFINITIONS(-D_USE_MATH_DEFINES)
	ADD_DEFINITIONS(-DNOMINMAX)
ENDIF (WIN32)

# Find needed packages
FIND_PACKAGE(LibXml2 REQUIRED)

OPTION(SC_GLOBAL_UNITTESTS "Enable unit tests" ON)
OPTION(SC_GLOBAL_PYTHON_WRAPPER "Create Python wrappers" ON)
OPTION(SC_GLOBAL_PYTHON_WRAPPER_NUMPY "Add Numpy support for Python wrapper" ON)
OPTION(SC_GLOBAL_PYTHON_WRAPPER_PYTHON3 "Require Python3 interpreter" OFF)
OPTION(SC_ENABLE_CONTRIB "Enables descending into the contrib directory" ON)


# Modified by Adrien Oth <adrien.oth@ecgs.lu>
# Find python interpreter, macOS support
IF (APPLE)
    # To avoid picking up the system elements pre-maturely
    SET(CMAKE_FIND_FRAMEWORK LAST)
    IF (Python_VERSION_REQUIRED)
       FIND_PACKAGE(Python ${Python_VERSION_REQUIRED} EXACT REQUIRED COMPONENTS Interpreter)
    ELSE (Python_VERSION_REQUIRED)
       FIND_PACKAGE(Python REQUIRED COMPONENTS Interpreter)
    ENDIF (Python_VERSION_REQUIRED)

    SET(PYTHON_EXECUTABLE ${Python_EXECUTABLE})
ELSE (APPLE)
	IF(SC_GLOBAL_PYTHON_WRAPPER_PYTHON3)
		SET(PythonInterp_FIND_VERSION 3)
		SET(PythonInterp_FIND_VERSION_MAJOR 3)
		SET(PythonInterp_FIND_VERSION_EXACT 0)
	ENDIF()
    FIND_PACKAGE(PythonInterp)
ENDIF (APPLE)  

# Modified by Gilles Celli <gilles.celli@ecgs.lu>
# Alternative to above IF (APPLE) block using fixed Homebrew version of python 3.7
#IF (APPLE)
#    # Set Homebrew's Python3 variables
#    #SET(PYTHON_CONFIG_EXECUTABLE /usr/local/bin/python3-config)
#    FIND_PACKAGE (Python COMPONENTS Interpreter Development)
#    #find_package(PythonInterp)
#    #SET(PYTHON_EXECUTABLE /usr/local/bin/python3)
#    #SET(PYTHON_INCLUDE_DIR /usr/local/Frameworks/Python.framework/Versions/Current/include/python3.7m/)
#    #SET(PYTHON_LIBRARY     /usr/local/Frameworks/Python.framework/Versions/Current/lib/python3.7/)
#    #FIND_PACKAGE (Python3 COMPONENTS Interpreter Development)
#ELSE (APPLE)
#	IF(SC_GLOBAL_PYTHON_WRAPPER_PYTHON3)
#		SET(PythonInterp_FIND_VERSION 3)
#		SET(PythonInterp_FIND_VERSION_MAJOR 3)
#		SET(PythonInterp_FIND_VERSION_EXACT 0)
#	ENDIF()
#    FIND_PACKAGE(PythonInterp)#ENDIF (APPLE)    

EXEC_PROGRAM(
	${PYTHON_EXECUTABLE}
	ARGS "-c \"from __future__ import print_function;import sys;print(sys.version[:3])\""
	OUTPUT_VARIABLE PYTHON_VERSION
)

IF(SC_GLOBAL_PYTHON_WRAPPER_NUMPY)
	FIND_PACKAGE(Numpy)
ENDIF()

#IF (APPLE)
#	FIND_PACKAGE(PythonLibsApple REQUIRED)
#ELSE (APPLE)
#	FIND_PACKAGE(PythonLibs REQUIRED)
#ENDIF (APPLE)

# Modified by Adrien Oth <adrien.oth@ecgs.lu>
# Support for finding python on macOS, possibility to pass fixed version desired to be used
IF (SC_GLOBAL_PYTHON_WRAPPER)
    IF (APPLE)
       # To avoid picking up the system elements pre-maturely
       SET(CMAKE_FIND_FRAMEWORK LAST)
       IF (Python_VERSION_REQUIRED)
          FIND_PACKAGE(Python ${Python_VERSION_REQUIRED} EXACT REQUIRED COMPONENTS Interpreter Development)
       ELSE (Python_VERSION_REQUIRED)
          FIND_PACKAGE(Python REQUIRED COMPONENTS Interpreter Development)
       ENDIF (Python_VERSION_REQUIRED)
       
       SET(PYTHON_LIBRARIES ${Python_LIBRARIES})
       SET(PYTHON_INCLUDE_PATH ${Python_INCLUDE_DIRS})
       SET(PYTHON_INCLUDE_DIR ${Python_INCLUDE_DIRS})
       SET(PYTHON_EXECUTABLE ${Python_EXECUTABLE})
    ELSE (APPLE)
       FIND_PACKAGE(PythonLibs REQUIRED)
	   FIND_PACKAGE(PythonInterp REQUIRED)
	ENDIF (APPLE)
ENDIF (SC_GLOBAL_PYTHON_WRAPPER)

#FIND_PACKAGE(PythonInterp REQUIRED)
SET(PYTHON_LIBRARY_SUFFIX /python)
SET(PYTHON_LIBRARY_PATH lib${PYTHON_LIBRARY_SUFFIX})

OPTION(SC_GLOBAL_GUI "Build graphical user interfaces (requires Qt4 or Qt5)" ON)
OPTION(SC_GLOBAL_GUI_QT5 "Build graphical user interfaces for Qt5" ON)

IF(SC_GLOBAL_GUI)
	FIND_PACKAGE(Qt5 QUIET COMPONENTS Core Widgets)
	IF(NOT Qt5_FOUND)
		SET(SC_GLOBAL_GUI_QT5 OFF)
	ENDIF()

	IF(NOT SC_GLOBAL_GUI_QT5)
		FIND_PACKAGE(Qt4 QUIET)
	ENDIF()
ENDIF()

INCLUDE(AddFileDependencies)
INCLUDE(SeisComP)

SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "")
SET(CPACK_PACKAGE_VENDOR "seiscomp-devel@gfz-potsdam.de")
SET(CPACK_PACKAGE_VERSION_NAME  "Potsdam")
SET(CPACK_PACKAGE_VERSION_MAJOR "4")
SET(CPACK_PACKAGE_VERSION_MINOR "0")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
#SET(CPACK_STRIP_FILES "bin/*")
SET(CPACK_SOURCE_STRIP_FILES "")
#SET(CPACK_PACKAGE_EXECUTABLES "bin/*")
SET(CPACK_PACKAGE_FILE_NAME "seiscomp-${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
INCLUDE(CPack)

SET(PROJECT_CONFIG_DIR etc)
SET(PROJECT_SHARE_DIR share)
SET(PROJECT_API_H api.h)

SET(SC3_PACKAGE_ROOT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

# Default directories that can be overridden in the
# subprojects
SET(SC3_PACKAGE_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
SET(SC3_PACKAGE_BIN_DIR bin)
SET(SC3_PACKAGE_LIB_DIR lib)
SET(SC3_PACKAGE_INCLUDE_DIR include)
SET(SC3_PACKAGE_SHARE_DIR share)
SET(SC3_PACKAGE_CONFIG_DIR etc)
SET(SC3_PACKAGE_TEMPLATES_DIR templates)

# Platform check
#
IF (APPLE AND ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	SET (MACOSX 1)
	MESSAGE(STATUS "MACOSX detected")
ELSEIF (UNIX AND ${CMAKE_SYSTEM_NAME} MATCHES "Linux")
 	SET (LINUX 1)
ENDIF (APPLE AND ${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

# If gcc is installed add -wall and -pedantic to the commandline
#
IF (CMAKE_COMPILER_IS_GNUCC)
	IF ( ${CMAKE_BUILD_TYPE} MATCHES "Debug" )
		ADD_DEFINITIONS(-Wall -pedantic -Wno-long-long -Wno-variadic-macros)
		#ADD_DEFINITIONS(-Wall -Wno-long-long -Wno-variadic-macros)
	ELSE ( ${CMAKE_BUILD_TYPE} MATCHES "Debug" )
		ADD_DEFINITIONS(-Wall)
	ENDIF ( ${CMAKE_BUILD_TYPE} MATCHES "Debug" )

	# Query gcc version. We can't use CMAKE_CXX_COMPILER_VERSION which is
	# only supported in cmake >= 2.8.10. SLES 11 still comes with cmake 2.6.
	EXECUTE_PROCESS(
		COMMAND ${CMAKE_C_COMPILER} -dumpversion
		OUTPUT_VARIABLE GCC_VERSION
	)

	STRING(REGEX MATCHALL "[0-9]+" GCC_VERSION_COMPONENTS ${GCC_VERSION})
	LIST(LENGTH GCC_VERSION_COMPONENTS GCC_VERSION_COMPONENTS_SIZE)
	LIST(GET GCC_VERSION_COMPONENTS 0 GCC_VERSION_MAJOR)
	# E.g. Fedora 26 does by default report only the major version number (7)
	# The new option "-dumpfullversion" is not available on older compilers
	IF(GCC_VERSION_COMPONENTS_SIZE GREATER 1)
		LIST(GET GCC_VERSION_COMPONENTS 1 GCC_VERSION_MINOR)
	ELSE(GCC_VERSION_COMPONENTS_SIZE GREATER 1)
		SET(GCC_VERSION_MINOR 0)
	ENDIF(GCC_VERSION_COMPONENTS_SIZE GREATER 1)

	MESSAGE(STATUS "Found gcc version ${GCC_VERSION_MAJOR}.${GCC_VERSION_MINOR}")

	# Enable C++11
	IF(NOT GCC_VERSION VERSION_LESS "4.7")
		SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
	ENDIF(NOT GCC_VERSION VERSION_LESS "4.7")
ENDIF (CMAKE_COMPILER_IS_GNUCC)


# Set some specific settings for Mac OS X - macOS - Darwin
#
IF (APPLE)
    
    set(CMAKE_OSX_ARCHITECTURES "x86_64")
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13.0" CACHE STRING "")
    
    # OpenSSL with Homebrew
	FIND_PACKAGE(OpenSSL REQUIRED)
		        
    # Set RPATH for Mac OS X - Darwin
    SET(CMAKE_MACOSX_RPATH ON)
    
    # use, i.e. don't skip the full RPATH for the build tree
    SET(CMAKE_SKIP_BUILD_RPATH FALSE)
    
    # when building, don't use the install RPATH already
    # (but later on when installing)
    SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE) 

    # add the automatically determined parts of the RPATH
    # which point to directories outside the build tree to the install RPATH
    SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

    # the RPATH to be used when installing, but only if it's not a system directory
    LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
    IF("${isSystemDir}" STREQUAL "-1")
        SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
    ENDIF("${isSystemDir}" STREQUAL "-1")
    
	SET(CMAKE_INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib")
		
	# Use clang compiler
	IF (NOT DEFINED CMAKE_CXX_COMPILER)
		SET(CMAKE_CXX_COMPILER "/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang++")
		IF (NOT EXISTS "${CMAKE_CXX_COMPILER}")
			SET(CMAKE_CXX_COMPILER "clang++")
		ENDIF(NOT EXISTS "${CMAKE_CXX_COMPILER}")
	ENDIF(NOT DEFINED CMAKE_CXX_COMPILER)
	
	IF (NOT DEFINED CMAKE_C_COMPILER)
		SET(CMAKE_C_COMPILER "/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang")
		IF (NOT EXISTS "${CMAKE_C_COMPILER}")
			SET(CMAKE_C_COMPILER "clang")
		ENDIF(NOT EXISTS "${CMAKE_C_COMPILER}")
	ENDIF(NOT DEFINED CMAKE_C_COMPILER)
	
	# clang defaults to C++14 which might break compilation. Setting to C++11 works.
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wno-deprecated-declarations -Wno-inconsistent-missing-override")
	
ENDIF (APPLE)

# Set the platform specific suffix for dynamic libraries
ADD_DEFINITIONS(
	-DSHARED_LIBRARY_SUFFIX="${CMAKE_SHARED_LIBRARY_SUFFIX}"
	-DSHARED_MODULE_SUFFIX="${CMAKE_SHARED_MODULE_SUFFIX}"
 )

# Helper for Eclipse CDT
#
IF(CMAKE_COMPILER_IS_GNUCC)
	SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fmessage-length=0")
ENDIF(CMAKE_COMPILER_IS_GNUCC)
IF(CMAKE_COMPILER_IS_GNUCXX)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fmessage-length=0")
ENDIF(CMAKE_COMPILER_IS_GNUCXX)

ADD_DEFINITIONS(-D_REENTRANT)

IF(SC_GLOBAL_UNITTESTS)
	MESSAGE(STATUS "Enable unit tests")
	FIND_PACKAGE(Boost REQUIRED COMPONENTS unit_test_framework)
	ENABLE_TESTING()
ELSE(SC_GLOBAL_UNITTESTS)
	MESSAGE(STATUS "Disable unit tests")
ENDIF(SC_GLOBAL_UNITTESTS)

SC_ADD_CHANGELOG(seiscomp)
INSTALL(FILES doc/base/license.rst RENAME LICENSE DESTINATION ${SC3_PACKAGE_SHARE_DIR}/doc/seiscomp)
INSTALL(FILES LICENSE.html DESTINATION ${SC3_PACKAGE_SHARE_DIR}/doc/seiscomp)

SUBDIRS(src doc)
