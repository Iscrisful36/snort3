cmake_minimum_required (VERSION 3.5)
project (snort CXX C)

set (VERSION_MAJOR 3)
set (VERSION_MINOR 9)
set (VERSION_PATCH 2)
set (VERSION_SUBLEVEL 0)
set (VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}.${VERSION_SUBLEVEL}")

if (DEFINED VERSION_BUILD)
    set (BUILD "${VERSION_BUILD}")
endif ()

set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CXX_STANDARD_REQUIRED ON)
set (CMAKE_CXX_EXTENSIONS OFF)

set (CMAKE_C_STANDARD 99)
set (CMAKE_C_STANDARD_REQUIRED ON)
set (CMAKE_C_EXTENSIONS ON)

set (CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
# Pull in definitions of various install directories
include (GNUInstallDirs)
include (${CMAKE_MODULE_PATH}/macros.cmake)
include (${CMAKE_MODULE_PATH}/create_options.cmake)
include (${CMAKE_MODULE_PATH}/include_libraries.cmake)
include (${CMAKE_MODULE_PATH}/platforms.cmake)
include (${CMAKE_MODULE_PATH}/configure_options.cmake)
#include (${CMAKE_MODULE_PATH}/compiler_features.cmake)
include (${CMAKE_MODULE_PATH}/sanity_checks.cmake)

set (INSTALL_SUFFIX "${CMAKE_PROJECT_NAME}")
set (INCLUDE_INSTALL_PATH "${CMAKE_INSTALL_INCLUDEDIR}/${INSTALL_SUFFIX}")
set (LIB_INSTALL_PATH "${CMAKE_INSTALL_LIBDIR}/${INSTALL_SUFFIX}")
set (PLUGIN_INSTALL_PATH "${LIB_INSTALL_PATH}/plugins")

configure_file (
    "${PROJECT_SOURCE_DIR}/config.cmake.h.in"
    "${PROJECT_BINARY_DIR}/config.h"
)

add_definitions( -DHAVE_CONFIG_H )

# Set these after all tests are done but *before* any subdirectories are included
#  or other targets declared.
string(APPEND CMAKE_C_FLAGS " ${EXTRA_C_FLAGS}")
string(APPEND CMAKE_CXX_FLAGS " ${EXTRA_CXX_FLAGS}")
string(APPEND CMAKE_EXE_LINKER_FLAGS " ${EXTRA_LINKER_FLAGS}")
string(APPEND CMAKE_MODULE_LINKER_FLAGS " ${EXTRA_LINKER_FLAGS}")

include_directories (${PROJECT_BINARY_DIR})
include_directories (${PROJECT_SOURCE_DIR})

if (ENABLE_UNIT_TESTS)
    include(CTest)
endif (ENABLE_UNIT_TESTS)

if (ENABLE_UNIT_TESTS OR ENABLE_BENCHMARK_TESTS)
    add_custom_target (check COMMAND ${CMAKE_CTEST_COMMAND})
endif (ENABLE_UNIT_TESTS OR ENABLE_BENCHMARK_TESTS)

add_subdirectory (src)
add_subdirectory (tools)
add_subdirectory (lua)
add_subdirectory (daqs)

if ( MAKE_DOC )
    add_subdirectory (doc)
endif ( MAKE_DOC )

# Miscellaneous stuff.  Does NOT directly affect Snort's build environment.
include (${CMAKE_MODULE_PATH}/packaging_data.cmake)
include (${CMAKE_MODULE_PATH}/create_pkg_config.cmake)

# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)

string(TOUPPER "${CMAKE_BUILD_TYPE}" CMAKE_BUILD_TYPE_UPPER)

message("
-------------------------------------------------------")
if (DEFINED BUILD)
    message("${CMAKE_PROJECT_NAME} version ${VERSION} build ${BUILD}")
else()
    message("${CMAKE_PROJECT_NAME} version ${VERSION}")
endif()
message("
Install options:
    prefix:     ${CMAKE_INSTALL_PREFIX}
    includes:   ${CMAKE_INSTALL_FULL_INCLUDEDIR}/${INSTALL_SUFFIX}
    plugins:    ${CMAKE_INSTALL_FULL_LIBDIR}/${INSTALL_SUFFIX}

Compiler options:
    CC:             ${CMAKE_C_COMPILER}
    CXX:            ${CMAKE_CXX_COMPILER}
    CFLAGS:         ${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_${CMAKE_BUILD_TYPE_UPPER}}
    CXXFLAGS:       ${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE_UPPER}}
    EXE_LDFLAGS:    ${CMAKE_EXE_LINKER_FLAGS}
    MODULE_LDFLAGS: ${CMAKE_MODULE_LINKER_FLAGS}

Feature options:\
")

if (ENABLE_STATIC_DAQ)
    message("\
    DAQ Modules:    Static (${DAQ_STATIC_MODULES})")
else ()
    message("\
    DAQ Modules:    Dynamic")
endif ()

if (HAVE_ATOMIC)
    message("\
    libatomic:      User-specified")
else ()
    message("\
    libatomic:      System-provided")
endif ()

if (HAVE_HYPERSCAN)
    message("\
    Hyperscan:      ON")
else ()
    message("\
    Hyperscan:      OFF")
endif ()

if (HAVE_ICONV)
    message("\
    ICONV:          ON")
else ()
    message("\
    ICONV:          OFF")
endif ()

if (HAVE_LIBUNWIND)
    message("\
    Libunwind:      ON")
else ()
    message("\
    Libunwind:      OFF")
endif ()

if (HAVE_LZMA)
    message("\
    LZMA:           ON")
else ()
    message("\
    LZMA:           OFF")
endif ()

if (USE_TIRPC)
    message("\
    RPC DB:         TIRPC")
else ()
    message("\
    RPC DB:         Built-in")
endif ()

if (HAVE_SAFEC)
    message("\
    SafeC:          ON")
else ()
    message("\
    SafeC:          OFF")
endif ()

if (HAVE_TCMALLOC)
    message("\
    TCMalloc:       ON")
else ()
    message("\
    TCMalloc:       OFF")
endif ()

if (HAVE_JEMALLOC)
    message("\
    JEMalloc:       ON")
else ()
    message("\
    JEMalloc:       OFF")
endif ()

if (HAVE_UUID)
    message("\
    UUID:           ON")
else ()
    message("\
    UUID:           OFF")
endif ()

if (HAVE_NUMA)
    message("\
    NUMA:           ON")
else ()
    message("\
    NUMA:           OFF")
endif ()

if (HAVE_LIBML)
    message("\
    LibML:          ON")
else ()
    message("\
    LibML:          OFF")
endif ()

message("-------------------------------------------------------\n")
