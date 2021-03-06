#-------------------------------------------------------------------------------
# Copyright (C) 2019-2020 Zilliz. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software distributed under the License
# is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
# or implied. See the License for the specific language governing permissions and limitations under the License.
#-------------------------------------------------------------------------------

# ----------------------------------------------------------------------
# Thirdpart cxx and c flags
add_compile_options( -O3 -fPIC -Wno-error -fopenmp )

# ----------------------------------------------------------------------
# verbose build info write into log file
if ( NOT KNOWHERE_VERBOSE_THIRDPARTY_BUILD )
    set( EP_LOG_OPTIONS LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1 LOG_DOWNLOAD 1 )
else ()
    set( EP_LOG_OPTIONS )
endif ()

set( MAKE_BUILD_ARGS "-j6" )

# ----------------------------------------------------------------------
# FetchContent config
include( FetchContent )
set( FETCHCONTENT_BASE_DIR  ${MILVUS_BINARY_DIR}/3rdparty_download )
set( FETCHCONTENT_QUIET     OFF )

if( CUSTOM_THIRDPARTY_DOWNLOAD_PATH )
    set( THIRDPARTY_DOWNLOAD_PATH ${CUSTOM_THIRDPARTY_DOWNLOAD_PATH} )
else()
    set( THIRDPARTY_DOWNLOAD_PATH ${CMAKE_BINARY_DIR}/3rdparty_download/download )
endif()
message( STATUS "Thirdparty downloaded file path: ${THIRDPARTY_DOWNLOAD_PATH}" )

# ----------------------------------------------------------------------
# using ccache
if( CCACHE_FOUND )
    set( EP_C_COMPILER      "${CCACHE_FOUND} ${CMAKE_C_COMPILER}" )
    set( EP_CXX_COMPILER    "${CCACHE_FOUND} ${CMAKE_CXX_COMPILER}" )
else()
    set( EP_C_COMPILER      ${CMAKE_C_COMPILER} )
    set( EP_CXX_COMPILER    ${CMAKE_CXX_COMPILER} )
endif()


# ----------------------------------------------------------------------
# Versions and URLs for toolchain builds, which also can be used to configure
# offline builds

file(STRINGS "versions.txt" TOOLCHAIN_VERSIONS_TXT)
foreach (_VERSION_ENTRY ${TOOLCHAIN_VERSIONS_TXT})
    # Exclude comments
    if (NOT _VERSION_ENTRY MATCHES "^[^#][A-Za-z0-9-_]+_VERSION=")
        continue()
    endif ()

    string(REGEX MATCH "^[^=]*" _LIB_NAME ${_VERSION_ENTRY})
    string(REPLACE "${_LIB_NAME}=" "" _LIB_VERSION ${_VERSION_ENTRY})

    # Skip blank or malformed lines
    if (${_LIB_VERSION} STREQUAL "")
        continue()
    endif ()

    # For debugging
    #message(STATUS "${_LIB_NAME}: ${_LIB_VERSION}")

    set(${_LIB_NAME} "${_LIB_VERSION}")
endforeach ()

# ----------------------------------------------------------------------
# Ensure that a default make is set
if ("${MAKE}" STREQUAL "")
    find_program(MAKE make)
endif ()

if (NOT DEFINED MAKE_BUILD_ARGS)
    set(MAKE_BUILD_ARGS "-j8")
endif ()
message(STATUS "Third Party MAKE_BUILD_ARGS = ${MAKE_BUILD_ARGS}")

# ----------------------------------------------------------------------
# Find pthreads

set( THREADS_PREFER_PTHREAD_FLAG ON )
find_package( Threads REQUIRED )

# ****************************** Thirdparty googletest ***************************************
if ( MILVUS_BUILD_TESTS )
    add_subdirectory( gtest )
endif()

# ****************************** Thirdparty grpc ***************************************
if ( MILVUS_WITH_GRPC )
    add_subdirectory( grpc )
endif ()

# ****************************** Thirdparty yaml ***************************************
if ( MILVUS_WITH_YAMLCPP )
    add_subdirectory( yaml-cpp )
endif()

# ****************************** Thirdparty crc32c ***************************************
add_subdirectory( crc32c )

# ****************************** Thirdparty opentracing ***************************************
if ( MILVUS_WITH_OPENTRACING )
    add_subdirectory( opentracing )
endif()

# ****************************** Thirdparty perftools ***************************************
if ( ENABLE_CPU_PROFILING )
    add_subdirectory( profilers )
endif ()

# ****************************** Thirdparty mysqlpp ***************************************
if ( MILVUS_WITH_MYSQLPP )
    add_subdirectory( mysqlpp )
endif ()

# ****************************** Thirdparty sqlite ***************************************
if ( MILVUS_WITH_SQLITE )
    add_subdirectory( sqlite )
endif()

# ****************************** Thirdparty prometheus ***************************************
if (MILVUS_WITH_PROMETHEUS)
    add_subdirectory( prometheus )
endif ()

# ****************************** Thirdparty fiu ***************************************
if ( MILVUS_WITH_FIU )
    add_subdirectory( fiu )
endif()

# ****************************** Thirdparty oatpp ***************************************
if ( MILVUS_WITH_OATPP )
    add_subdirectory( oatpp )
endif()

# ****************************** Thirdparty aws ***************************************
if (MILVUS_WITH_AWS)
    add_subdirectory( aws )
endif ()
