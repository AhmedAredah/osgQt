#===============================================================================
# osgQt - OpenSceneGraph Qt6 Integration Library
#===============================================================================
#
# Modern CMake build configuration for osgQt with Qt6 support.
#
# Supported Build Types:
#   - Release       : Optimized build (no debug info)
#   - Debug         : Debug build with symbols (library suffix: d)
#   - RelWithDebInfo: Release with debug info (library suffix: rd)
#   - MinSizeRel    : Minimum size release (library suffix: s)
#
# Usage Example:
#   cmake -B build_release -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local/Release
#   cmake --build build_release --parallel
#   sudo cmake --install build_release
#
#===============================================================================

cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

#-------------------------------------------------------------------------------
# Project Definition
#-------------------------------------------------------------------------------
project(osgQt
    VERSION 3.6.5
    DESCRIPTION "OpenSceneGraph Qt6 Integration Library"
    LANGUAGES CXX
)

# Use folders in IDEs (Visual Studio, Xcode)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

#-------------------------------------------------------------------------------
# C++ Standard Configuration
#-------------------------------------------------------------------------------
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

#-------------------------------------------------------------------------------
# CMake Module Path
#-------------------------------------------------------------------------------
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules")
include(OsgMacroUtils)

#-------------------------------------------------------------------------------
# Build Type Configuration
#-------------------------------------------------------------------------------
# Set default build type if not specified (for single-config generators)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
        "Build type: Debug, Release, RelWithDebInfo, MinSizeRel" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
        "Debug" "Release" "RelWithDebInfo" "MinSizeRel")
endif()

#-------------------------------------------------------------------------------
# Build Options
#-------------------------------------------------------------------------------
option(BUILD_SHARED_LIBS "Build shared libraries instead of static" ON)
option(BUILD_OSG_EXAMPLES "Build osgQt example applications" OFF)
option(OSGQT_AGGRESSIVE_WARNINGS "Enable aggressive compiler warnings" ON)

#-------------------------------------------------------------------------------
# Library Naming - Build Type Postfixes
#-------------------------------------------------------------------------------
set(CMAKE_DEBUG_POSTFIX "d" CACHE STRING "Debug library name postfix")
set(CMAKE_RELEASE_POSTFIX "" CACHE STRING "Release library name postfix")
set(CMAKE_RELWITHDEBINFO_POSTFIX "rd" CACHE STRING "RelWithDebInfo library name postfix")
set(CMAKE_MINSIZEREL_POSTFIX "s" CACHE STRING "MinSizeRel library name postfix")

#-------------------------------------------------------------------------------
# Find Required Dependencies
#-------------------------------------------------------------------------------
# OpenSceneGraph (minimum 3.6.0)
#
# For builds with non-standard library names (e.g., MinSizeRel with 's' suffix),
# you can manually specify the libraries using OSG_MANUAL_LIBRARIES option:
#
#   cmake .. -DOSG_MANUAL_LIBRARIES=ON \
#            -DOSG_INCLUDE_DIR=/usr/local/MinSizeRel/include \
#            -DOSG_MANUAL_LIBS="/path/to/libosg.so;/path/to/libosgDB.so;..."
#
option(OSG_MANUAL_LIBRARIES "Manually specify OpenSceneGraph libraries" OFF)

if(OSG_MANUAL_LIBRARIES)
    # Manual library specification mode
    set(OSG_INCLUDE_DIR "" CACHE PATH "OpenSceneGraph include directory")
    set(OSG_MANUAL_LIBS "" CACHE STRING "Semicolon-separated list of OSG library paths")

    if(NOT OSG_INCLUDE_DIR OR NOT OSG_MANUAL_LIBS)
        message(FATAL_ERROR
            "OSG_MANUAL_LIBRARIES is ON but OSG_INCLUDE_DIR or OSG_MANUAL_LIBS not set.\n"
            "Please specify:\n"
            "  -DOSG_INCLUDE_DIR=/path/to/osg/include\n"
            "  -DOSG_MANUAL_LIBS=\"/path/to/libosg.so;/path/to/libosgDB.so;...\""
        )
    endif()

    set(OPENSCENEGRAPH_INCLUDE_DIRS "${OSG_INCLUDE_DIR}")
    set(OPENSCENEGRAPH_LIBRARIES "${OSG_MANUAL_LIBS}")
    set(OPENSCENEGRAPH_VERSION "3.7.0")  # Default version for manual mode

    message(STATUS "Using manually specified OpenSceneGraph libraries")
    message(STATUS "  Include: ${OPENSCENEGRAPH_INCLUDE_DIRS}")
    message(STATUS "  Libraries: ${OPENSCENEGRAPH_LIBRARIES}")
else()
    # Standard find_package mode
    find_package(OpenSceneGraph 3.6.0 REQUIRED
        COMPONENTS osgDB osgGA osgUtil osgText osgViewer osgWidget
    )
endif()

set(OPENSCENEGRAPH_SOVERSION 145)
set(OSG_PLUGINS "osgPlugins-${OPENSCENEGRAPH_VERSION}")

# Qt6 with required components
find_package(Qt6 COMPONENTS Widgets OpenGL OpenGLWidgets REQUIRED)

# OpenGL
find_package(OpenGL REQUIRED)

# Threads
find_package(Threads REQUIRED)

#-------------------------------------------------------------------------------
# Compiler Configuration
#-------------------------------------------------------------------------------
# Debug definition for Unix platforms
if(UNIX AND NOT WIN32)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_DEBUG")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -D_DEBUG")
endif()

# Warning flags based on compiler
if(OSGQT_AGGRESSIVE_WARNINGS)
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        add_compile_options(
            -Wall -Wextra -Wpedantic
            -Wparentheses -Wreturn-type -Wmissing-braces
            -Wunknown-pragmas -Wunused -Wshadow
            -Wno-long-long
        )
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        add_compile_options(
            -Wall -Wextra -Wpedantic
            -Wparentheses -Wreturn-type -Wmissing-braces
            -Wunknown-pragmas -Wunused
            -Wno-overloaded-virtual -Wno-long-long
        )
    elseif(MSVC)
        add_compile_options(/W4 /wd4706 /wd4127 /wd4100)
        add_definitions(-D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_DEPRECATE)
    endif()
endif()

#-------------------------------------------------------------------------------
# Output Directory Configuration
#-------------------------------------------------------------------------------
set(OUTPUT_BINDIR "${CMAKE_BINARY_DIR}/bin")
set(OUTPUT_LIBDIR "${CMAKE_BINARY_DIR}/lib")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${OUTPUT_LIBDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${OUTPUT_LIBDIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${OUTPUT_BINDIR}")

# Per-configuration output directories (for multi-config generators like VS/Xcode)
foreach(CONFIG_TYPE IN ITEMS DEBUG RELEASE RELWITHDEBINFO MINSIZEREL)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${CONFIG_TYPE} "${OUTPUT_LIBDIR}")
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${CONFIG_TYPE} "${OUTPUT_LIBDIR}")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${CONFIG_TYPE} "${OUTPUT_BINDIR}")
endforeach()

# Create output directories
file(MAKE_DIRECTORY "${OUTPUT_BINDIR}")
file(MAKE_DIRECTORY "${OUTPUT_LIBDIR}")

#-------------------------------------------------------------------------------
# Library Type Configuration (for OSG macros compatibility)
#-------------------------------------------------------------------------------
if(BUILD_SHARED_LIBS)
    set(OPENSCENEGRAPH_USER_DEFINED_DYNAMIC_OR_STATIC "SHARED")
    set(DYNAMIC_OPENSCENEGRAPH ON)
else()
    set(OPENSCENEGRAPH_USER_DEFINED_DYNAMIC_OR_STATIC "STATIC")
    set(DYNAMIC_OPENSCENEGRAPH OFF)
endif()

set(OPENSCENEGRAPH_SONAMES TRUE)
set(OpenSceneGraph_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")

# Plugin prefix for different platforms
set(OSG_PLUGIN_PREFIX "")
if(CYGWIN)
    set(OSG_PLUGIN_PREFIX "cygwin_")
elseif(MINGW)
    set(OSG_PLUGIN_PREFIX "mingw_")
endif()

# Library suffix for 64-bit Linux
set(LIB_POSTFIX "")
if(UNIX AND NOT APPLE AND CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(LIB_POSTFIX "64" CACHE STRING "Library directory suffix for 64-bit systems")
    mark_as_advanced(LIB_POSTFIX)
endif()

#-------------------------------------------------------------------------------
# Include Directories
#-------------------------------------------------------------------------------
include_directories(
    ${OPENSCENEGRAPH_INCLUDE_DIRS}
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
    "${CMAKE_CURRENT_BINARY_DIR}/include"
    "${CMAKE_CURRENT_BINARY_DIR}/src"
    ${OPENGL_INCLUDE_DIR}
)

#-------------------------------------------------------------------------------
# Build Subdirectories
#-------------------------------------------------------------------------------
add_subdirectory(src)

if(BUILD_OSG_EXAMPLES)
    add_subdirectory(examples)
endif()

#-------------------------------------------------------------------------------
# Package Configuration (pkg-config)
#-------------------------------------------------------------------------------
set(PKGCONFIG_MODULE_NAME osgQt)
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/packaging/pkgconfig/openscenegraph-osgQt.pc.in"
    "${CMAKE_CURRENT_BINARY_DIR}/packaging/pkgconfig/openscenegraph-${PKGCONFIG_MODULE_NAME}.pc"
    @ONLY
)

install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/packaging/pkgconfig/openscenegraph-${PKGCONFIG_MODULE_NAME}.pc"
    DESTINATION "lib${LIB_POSTFIX}/pkgconfig"
    COMPONENT development
)

#-------------------------------------------------------------------------------
# Uninstall Target
#-------------------------------------------------------------------------------
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules/cmake_uninstall.cmake.in")
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY
    )
    add_custom_target(uninstall
        COMMAND "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        COMMENT "Uninstalling osgQt..."
    )
endif()

#-------------------------------------------------------------------------------
# Configuration Summary
#-------------------------------------------------------------------------------
message(STATUS "")
message(STATUS "========================================")
message(STATUS "  osgQt Configuration Summary")
message(STATUS "========================================")
message(STATUS "  Version:            ${PROJECT_VERSION}")
message(STATUS "  Build Type:         ${CMAKE_BUILD_TYPE}")
message(STATUS "  Install Prefix:     ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  Shared Libraries:   ${BUILD_SHARED_LIBS}")
message(STATUS "  Build Examples:     ${BUILD_OSG_EXAMPLES}")
message(STATUS "")
message(STATUS "  Dependencies:")
message(STATUS "    OpenSceneGraph:   ${OPENSCENEGRAPH_VERSION}")
message(STATUS "    Qt6:              ${Qt6_VERSION}")
message(STATUS "    OpenGL:           Found")
message(STATUS "")
message(STATUS "  Compiler:")
message(STATUS "    C++ Compiler:     ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "    C++ Standard:     ${CMAKE_CXX_STANDARD}")
message(STATUS "")
message(STATUS "  Library Postfixes:")
message(STATUS "    Debug:            '${CMAKE_DEBUG_POSTFIX}'")
message(STATUS "    Release:          '${CMAKE_RELEASE_POSTFIX}'")
message(STATUS "    RelWithDebInfo:   '${CMAKE_RELWITHDEBINFO_POSTFIX}'")
message(STATUS "    MinSizeRel:       '${CMAKE_MINSIZEREL_POSTFIX}'")
message(STATUS "========================================")
message(STATUS "")
