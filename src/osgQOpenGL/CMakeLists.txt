#===============================================================================
# osgQOpenGL Library - Qt6 OpenGL Widget Integration for OpenSceneGraph
#===============================================================================

if(NOT Qt6Widgets_FOUND)
    message(WARNING "Qt6Widgets not found - skipping osgQOpenGL library")
    return()
endif()

include(GenerateExportHeader)

#-------------------------------------------------------------------------------
# Library Configuration
#-------------------------------------------------------------------------------
set(LIB_NAME osgQOpenGL)

# Header and source paths
set(HEADER_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../include/osgQOpenGL")
set(SOURCE_PATH "${CMAKE_CURRENT_SOURCE_DIR}")

#-------------------------------------------------------------------------------
# Source Files
#-------------------------------------------------------------------------------
set(LIB_PUBLIC_HEADERS
    "${HEADER_PATH}/osgQOpenGLWidget.h"
    "${HEADER_PATH}/osgQOpenGLWindow.h"
    "${HEADER_PATH}/OSGRenderer.h"
)

set(LIB_SOURCES
    "${SOURCE_PATH}/CullVisitorEx.cpp"
    "${SOURCE_PATH}/GraphicsWindowEx.cpp"
    "${SOURCE_PATH}/osgQOpenGLWidget.cpp"
    "${SOURCE_PATH}/osgQOpenGLWindow.cpp"
    "${SOURCE_PATH}/OSGRenderer.cpp"
    "${SOURCE_PATH}/RenderStageEx.cpp"
    "${SOURCE_PATH}/StateEx.cpp"
)

#-------------------------------------------------------------------------------
# Qt6 MOC Processing
#-------------------------------------------------------------------------------
qt6_wrap_cpp(LIB_MOC_SOURCES ${LIB_PUBLIC_HEADERS})

#-------------------------------------------------------------------------------
# Library Target
#-------------------------------------------------------------------------------
add_library(${LIB_NAME}
    ${OPENSCENEGRAPH_USER_DEFINED_DYNAMIC_OR_STATIC}
    ${LIB_SOURCES}
    ${LIB_PUBLIC_HEADERS}
    ${LIB_MOC_SOURCES}
)

# Set library properties
set_target_properties(${LIB_NAME} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    FOLDER "osgQt Libraries"
    DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}"
    RELEASE_POSTFIX "${CMAKE_RELEASE_POSTFIX}"
    RELWITHDEBINFO_POSTFIX "${CMAKE_RELWITHDEBINFO_POSTFIX}"
    MINSIZEREL_POSTFIX "${CMAKE_MINSIZEREL_POSTFIX}"
)

#-------------------------------------------------------------------------------
# Include Directories
#-------------------------------------------------------------------------------
target_include_directories(${LIB_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${OPENSCENEGRAPH_INCLUDE_DIRS}
)

#-------------------------------------------------------------------------------
# Link Libraries
#-------------------------------------------------------------------------------
# Qt6 libraries
target_link_libraries(${LIB_NAME}
    PUBLIC
        Qt6::Widgets
        Qt6::OpenGL
        Qt6::OpenGLWidgets
)

# OpenSceneGraph libraries
target_link_libraries(${LIB_NAME}
    PUBLIC
        ${OPENSCENEGRAPH_LIBRARIES}
)

# OpenGL
target_link_libraries(${LIB_NAME}
    PRIVATE
        OpenGL::GL
)

# Threads
target_link_libraries(${LIB_NAME}
    PRIVATE
        Threads::Threads
)

#-------------------------------------------------------------------------------
# Export Header Generation
#-------------------------------------------------------------------------------
generate_export_header(${LIB_NAME}
    EXPORT_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/Export.h"
    EXPORT_MACRO_NAME OSGQOPENGL_EXPORT
)

#-------------------------------------------------------------------------------
# Installation Rules
#-------------------------------------------------------------------------------
include(GNUInstallDirs)

# Install library
install(TARGETS ${LIB_NAME}
    EXPORT osgQtTargets
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT development
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT runtime
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT runtime
)

# Install public headers
install(
    FILES ${LIB_PUBLIC_HEADERS}
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/osgQOpenGL"
    COMPONENT development
)

# Install generated export header
install(
    FILES "${CMAKE_CURRENT_BINARY_DIR}/Export.h"
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/osgQOpenGL"
    COMPONENT development
)

#-------------------------------------------------------------------------------
# CMake Package Configuration
#-------------------------------------------------------------------------------
install(EXPORT osgQtTargets
    FILE osgQtTargets.cmake
    NAMESPACE osgQt::
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/osgQt"
    COMPONENT development
)

# Generate and install package config files
include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/../../cmake/osgQtConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/osgQtConfig.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/osgQt"
    NO_SET_AND_CHECK_MACRO
    NO_CHECK_REQUIRED_COMPONENTS_MACRO
)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/osgQtConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/osgQtConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/osgQtConfigVersion.cmake"
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/osgQt"
    COMPONENT development
)
