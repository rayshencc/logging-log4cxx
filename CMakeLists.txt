#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
cmake_minimum_required(VERSION 3.13)
include(src/cmake/projectVersionDetails.cmake)
project(log4cxx VERSION ${log4cxx_VER} LANGUAGES CXX)
include(CTest)

# If you are including log4cxx from a higher-level CMake file(perhaps as a submodule?)
# CMAKE_SOURCE_DIR will refer to the first CMakeLists.txt.  Since we need some files
# in relation to the log4cxx source root, set a variable that contains the log4cxx root.
set(LOG4CXX_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

# The namespace in which all log4cxx entities will be placed, default value is "log4cxx"
set(LOG4CXX_NS "log4cxx" CACHE STRING "Root namespace name")

# define the SO Version of the library. Version of the log4cxx project is subtly different than the version of the library
# (at least in the past it was project vesion: 0.a.b.c and the library so vesion a.b.c.0
# See also: https://mail.kde.org/pipermail/kde-buildsystem/2008-April/004543.html
# Note that the lib version is different from the SOVERSION
# The lib version is the version of log4cxx, the SOVERSION is the ABI version
# See also: https://cmake.org/pipermail/cmake/2012-September/051904.html
option(LOG4CXX_BUILD_NEXT_ABI "Build code designated for the next ABI version" OFF)
if(LOG4CXX_BUILD_NEXT_ABI)
  set(log4cxx_ABI_VER 16)
else()
  set(log4cxx_ABI_VER 15)
endif()
set(LIBLOG4CXX_LIB_VERSION ${log4cxx_ABI_VER}.${log4cxx_VERSION_MINOR}.${log4cxx_VERSION_PATCH})
set(LIBLOG4CXX_LIB_SOVERSION ${log4cxx_ABI_VER})
# Set the 'release' version.  This is the human-readable version
set(LOG4CXX_RELEASE_VERSION ${log4cxx_VERSION_MAJOR}.${log4cxx_VERSION_MINOR}.${log4cxx_VERSION_PATCH})

# FindAPR and FindAPR-util are not provided by APR and APR-Util so source them locally
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/src/cmake")

# Add support for linking statically
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
if(NOT BUILD_SHARED_LIBS)
  set(LOG4CXX_COMPILE_DEFINITIONS LOG4CXX_STATIC)
endif()

option(ENABLE_COVERAGE "Enable code coverage" OFF)
if(ENABLE_COVERAGE)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
endif()

# Find Apache Runtime
option(APR_STATIC "Link to the APR static library" OFF)
find_package(APR REQUIRED)

# Find Apache Runtime Utilities
option(APU_STATIC "Link to the APR-Util static library" OFF)
find_package(APR-Util REQUIRED)

if(NOT MSVC)
  # The pthread library is used to name threads and mask signals
  find_package(Threads REQUIRED)
endif(NOT MSVC)

option(LOG4CXX_ENABLE_ODBC "Support logging via ODBC" OFF)
if(LOG4CXX_ENABLE_ODBC)
  find_package(ODBC)
  if(${ODBC_FOUND})
    set(HAS_ODBC 1)
  else()
    set(HAS_ODBC 0)
  endif(${ODBC_FOUND})

  if(NOT ${HAS_ODBC})
    message(SEND_ERROR "ODBC not found but requested")
  endif()
else()
  set(HAS_ODBC 0)
endif(LOG4CXX_ENABLE_ODBC)

option(LOG4CXX_ENABLE_ESMTP "Support logging via libesmtp" OFF)
option(LIBESMTP_STATIC "Link to the libesmtp static library" OFF)
if(LOG4CXX_ENABLE_ESMTP)
    find_package(ESMTP)
    if(ESMTP_FOUND)
       set(HAS_LIBESMTP 1)
    else()
       set(HAS_LIBESMTP 0)
       message(SEND_ERROR "libesmtp not found but SMTP support requested")
    endif(ESMTP_FOUND)
  else()
    set(ESMTP_LIBRARIES )
    set(HAS_LIBESMTP 0)
endif(LOG4CXX_ENABLE_ESMTP)

find_package(fmt 7.1 QUIET)
if(${fmt_FOUND})
    option(ENABLE_FMT_LAYOUT "Enable the FMT layout(if libfmt found)" ON)
else()
    set(ENABLE_FMT_LAYOUT "OFF")
endif()

# Request C++20, if available
# This *should* fallback to an older standard if it is not available
if( NOT "${CMAKE_CXX_STANDARD}")
    set(CMAKE_CXX_STANDARD 20)
endif()

# Don't allow for compiler-specific extensions
set(CMAKE_CXX_EXTENSIONS OFF)

# Use solution folders.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Building
add_subdirectory(src)

## Installing
if(WIN32)
  option(LOG4CXX_INSTALL_PDB "Install .pdb files (if generated)"  ON)
endif()
include(GNUInstallDirs)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/src/main/include/log4cxx
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h"
  PATTERN "Private" EXCLUDE
)
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/main/include/log4cxx
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h"
  PATTERN "Private" EXCLUDE
)

install(TARGETS log4cxx EXPORT ${LOG4CXX_LIB_NAME}Targets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

if(LOG4CXX_QT_SUPPORT)
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/main/include/log4cxx-qt
      DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
      FILES_MATCHING PATTERN "*.h"
    )
    install(TARGETS log4cxx-qt EXPORT ${LOG4CXX_LIB_NAME}-qtTargets
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
endif(LOG4CXX_QT_SUPPORT)

IF(MSVC AND BUILD_SHARED_LIBS AND LOG4CXX_INSTALL_PDB)
  INSTALL(FILES $<TARGET_PDB_FILE:log4cxx>
          DESTINATION ${CMAKE_INSTALL_BINDIR}
          CONFIGURATIONS RelWithDebInfo Debug
  )
ENDIF()

set(install_pkg_config ON)
if(MSVC)
    set(install_pkg_config OFF)
endif(MSVC)
option(LOG4CXX_WITH_PKG_CONFIG "Support pkg_config in consuming projects" ${install_pkg_config})
if(LOG4CXX_WITH_PKG_CONFIG)
    set(prefix "${CMAKE_INSTALL_PREFIX}")
    set(exec_prefix "${CMAKE_INSTALL_PREFIX}")
    set(libdir "\${prefix}/${CMAKE_INSTALL_LIBDIR}")
    set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
    set(VERSION "${log4cxx_VERSION_MAJOR}.${log4cxx_VERSION_MINOR}.${log4cxx_VERSION_PATCH}")
    configure_file("${CMAKE_CURRENT_SOURCE_DIR}/liblog4cxx.pc.in"
      "${CMAKE_CURRENT_BINARY_DIR}/lib${LOG4CXX_LIB_NAME}.pc"
    )

    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/lib${LOG4CXX_LIB_NAME}.pc"
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

    if(LOG4CXX_QT_SUPPORT)
        set(prefix "${CMAKE_INSTALL_PREFIX}")
        set(exec_prefix "${CMAKE_INSTALL_PREFIX}")
        set(libdir "\${prefix}/${CMAKE_INSTALL_LIBDIR}")
        set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
        set(VERSION "${log4cxx_VERSION_MAJOR}.${log4cxx_VERSION_MINOR}.${log4cxx_VERSION_PATCH}")
        configure_file("${CMAKE_CURRENT_SOURCE_DIR}/liblog4cxx-qt.pc.in"
          "${CMAKE_CURRENT_BINARY_DIR}/lib${LOG4CXX_LIB_NAME}-qt.pc"
        )

        install(FILES "${CMAKE_CURRENT_BINARY_DIR}/lib${LOG4CXX_LIB_NAME}-qt.pc"
          DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
        )
    endif(LOG4CXX_QT_SUPPORT)
endif(LOG4CXX_WITH_PKG_CONFIG)

# Support for find_package(log4cxx) in consuming CMake projects using
# target_include_directories(myApplication PRIVATE $<TARGET_PROPERTY:log4cxx,INTERFACE_INCLUDE_DIRECTORIES>)
# target_link_libraries( myApplication PRIVATE log4cxx)
install(EXPORT ${LOG4CXX_LIB_NAME}Targets
  FILE        ${LOG4CXX_LIB_NAME}Config.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LOG4CXX_LIB_NAME}
)
# Support for find_package(log4cxx 1.3) in consuming CMake projects
include(CMakePackageConfigHelpers)
write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/${LOG4CXX_LIB_NAME}ConfigVersion.cmake"
  VERSION       ${PROJECT_VERSION}
  COMPATIBILITY SameMinorVersion
)
install(FILES   "${CMAKE_CURRENT_BINARY_DIR}/${LOG4CXX_LIB_NAME}ConfigVersion.cmake"
  DESTINATION   ${CMAKE_INSTALL_LIBDIR}/cmake/${LOG4CXX_LIB_NAME}
)

if(LOG4CXX_QT_SUPPORT)
    install(EXPORT ${LOG4CXX_LIB_NAME}-qtTargets
      FILE        ${LOG4CXX_LIB_NAME}-qtConfig.cmake
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${LOG4CXX_LIB_NAME}-qt
    )
    # Support for find_package(log4cxx 0.11) in consuming CMake projects
    include(CMakePackageConfigHelpers)
    write_basic_package_version_file("${CMAKE_CURRENT_BINARY_DIR}/${LOG4CXX_LIB_NAME}-qtConfigVersion.cmake"
      VERSION       ${PROJECT_VERSION}
      COMPATIBILITY SameMinorVersion
    )
    install(FILES   "${CMAKE_CURRENT_BINARY_DIR}/${LOG4CXX_LIB_NAME}-qtConfigVersion.cmake"
      DESTINATION   ${CMAKE_INSTALL_LIBDIR}/cmake/${LOG4CXX_LIB_NAME}-qt
    )
endif(LOG4CXX_QT_SUPPORT)

#
# Get variables used in configuration information messages from subdirectories
#
get_directory_property( HAS_LIBESMTP DIRECTORY src/main/include DEFINITION HAS_LIBESMTP )
get_directory_property( HAS_SYSLOG DIRECTORY src/main/include DEFINITION HAS_SYSLOG )
get_directory_property( FILESYSTEM_IMPL DIRECTORY src DEFINITION FILESYSTEM_IMPL )
get_directory_property( STD_MAKE_UNIQUE_IMPL DIRECTORY src DEFINITION STD_MAKE_UNIQUE_IMPL )
get_directory_property( STD_LIB_HAS_UNICODE_STRING DIRECTORY src DEFINITION STD_LIB_HAS_UNICODE_STRING )
get_directory_property( HAS_THREAD_LOCAL DIRECTORY src DEFINITION HAS_THREAD_LOCAL )
get_directory_property( BUILD_FUZZERS DIRECTORY src DEFINITION BUILD_FUZZERS )

#
# Use ON/OFF in configuration information messages
#
foreach(varName HAS_ODBC  HAS_LIBESMTP  HAS_SYSLOG)
  if(${varName} EQUAL 0)
    set(${varName} "OFF" )
  elseif(${varName} EQUAL 1)
    set(${varName} "ON" )
  else()
    set(${varName} "UNKNOWN" )
  endif()
endforeach()

#
# Output configuration information
# Similar to APR CMake configuration
#
message(STATUS "")
message(STATUS "")
message(STATUS "Log4cxx configuration summary:")
message(STATUS "")

message(STATUS "  C++ compiler .................... : ${CMAKE_CXX_COMPILER}")
message(STATUS "  C++ features requested: ......... : ${CMAKE_CXX_STANDARD}")
message(STATUS "  Build shared library ............ : ${BUILD_SHARED_LIBS}")
message(STATUS "  Build tests ..................... : ${BUILD_TESTING}")
message(STATUS "  Build examples................... : ${BUILD_EXAMPLES}")
message(STATUS "  Build fuzzers.................... : ${BUILD_FUZZERS}")
message(STATUS "  Build site ...................... : ${BUILD_SITE}")
message(STATUS "  Install prefix .................. : ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  log4cxx library name ............ : ${LOG4CXX_LIB_NAME}")
message(STATUS "  log4cxx namespace ............... : ${LOG4CXX_NS}")
message(STATUS "  log4cxx char API ................ : ON")
message(STATUS "  log4cxx wchar API ............... : ${LOG4CXX_WCHAR_T}")
if(STD_LIB_HAS_UNICODE_STRING)
message(STATUS "  log4cxx unichar API ............. : ${LOG4CXX_UNICHAR}")
else()
message(STATUS "  std::basic_string<UniChar> ok? .. : FALSE")
endif()
if(APPLE)
message(STATUS "  log4cxx cfstring API ............ : ${LOG4CXX_CFSTRING}")
endif()
message(STATUS "  log4cxx char type ............... : ${LOG4CXX_CHAR}")
message(STATUS "  character encoding .............. : ${LOG4CXX_CHARSET}")
message(STATUS "  Networking support .............. : ${LOG4CXX_NETWORKING_SUPPORT}")
message(STATUS "  DOMConfigurator support ......... : ${LOG4CXX_DOMCONFIGURATOR_SUPPORT}")
message(STATUS "  Qt support ...................... : ${LOG4CXX_QT_SUPPORT}")
message(STATUS "C++ version and Boost settings:")
message(STATUS "  Prefer boost: ................... : ${PREFER_BOOST}")
message(STATUS "  make_unique implementation :..... : ${STD_MAKE_UNIQUE_IMPL}")
message(STATUS "  filesystem implementation ....... : ${FILESYSTEM_IMPL}")
message(STATUS "  format implementation ........... : ${LOG4CXX_FORMAT_NAMESPACE}::format")
message(STATUS "  thread_local support? ........... : ${HAS_THREAD_LOCAL}")

if(BUILD_TESTING)
message(STATUS "Applications required for tests:")
message(STATUS "  zip ............................. : ${ZIP_APP}")
message(STATUS "  sed ............................. : ${SED_APP}")
message(STATUS "  gzip ............................ : ${GZIP_APP}")
endif(BUILD_TESTING)

message(STATUS "Available appenders:")
message(STATUS "  Async Appender .................. : ON")
message(STATUS "  ODBC Appender ................... : ${HAS_ODBC}")
message(STATUS "  DB Appender ..................... : ON")
message(STATUS "  SMTP Appender ................... : ${HAS_LIBESMTP}")
message(STATUS "  XMLSocketAppender ............... : ${LOG4CXX_NETWORKING_SUPPORT}")
message(STATUS "  SyslogAppender .................. : ${LOG4CXX_NETWORKING_SUPPORT}")
if(LOG4CXX_NETWORKING_SUPPORT)
message(STATUS "  Using syslog.h .................. : ${HAS_SYSLOG}")
endif()
message(STATUS "  TelnetAppender .................. : ${LOG4CXX_NETWORKING_SUPPORT}")
if(WIN32)
message(STATUS "  NTEventLogAppender .............. : ON")
message(STATUS "  OutputDebugStringAppender ....... : ${LOG4CXX_NETWORKING_SUPPORT}")
endif()
message(STATUS "  ConsoleAppender ................. : ON")
message(STATUS "  FileAppender .................... : ON")
message(STATUS "  RollingFileAppender ............. : ON")
message(STATUS "  MultiprocessRollingFileAppender . : ${LOG4CXX_MULTIPROCESS_ROLLING_FILE_APPENDER}")

message(STATUS "Available layouts:")
message(STATUS "  HTMLLayout ...................... : ON")
message(STATUS "  JSONLayout ...................... : ON")
message(STATUS "  PatternLayout ................... : ON")
message(STATUS "  SimpleLayout .................... : ON")
message(STATUS "  XMLLayout ....................... : ON")
message(STATUS "  FMTLayout ....................... : ${ENABLE_FMT_LAYOUT}")
