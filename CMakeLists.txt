# === This file is part of Calamares - <https://github.com/calamares> ===
#
#   SPDX-FileCopyrightText: 2019 Adriaan de Groot <groot@kde.org>
#   SPDX-License-Identifier: GPL-3.0-or-later
#
###
#
#   Calamares-Examples is Free Software: see the License-Identifier above.
#
#   Individual files may have different licenses (like the CMake
#   infrastructure, which is BSD-2-Clause licensed). Check the SPDX
#   identifiers in each file.
#
###
#
# This is an example of a repository containing Calamares branding-components
# and Calamares modules and Calamares configuration, showing how a single
# repository can be used to provide customisation for Calamares, so that a
# distro can use an unmodified (upstream) Calamares package and a local
# customisation package in tandem.
#
# Besides being an example repository, it is also a collection of modules
# and branding that is usable in its own right.
#
### CONFIGURING
#
# By default, all the branding examples and all the modules are built.
# This can be influenced through:
#   SKIP_MODULES    : a space or semicolon-separated list of directory names
#                     under src/modules that should not be built.
#   USE_*           : fills in SKIP_MODULES for modules called *-<something>
# In this repository, there is just one "group" to which USE_* applies:
#   USE_os          : operating-system-specific modules.
#
### NOTES
#
# Call this CMake file in script mode, e.g. `cmake -P CMakeLists.txt`
# to print out version information. Use `cmake -DVERSION_STYLE=short`
# to get just the short versioning.
#
cmake_minimum_required(VERSION 3.3 FATAL_ERROR)

set( CALAMARES_EXTENSIONS_VERSION 1.2.1 )

include( ${CMAKE_CURRENT_LIST_DIR}/CMakeModules/ExtendedVersion.cmake )
if ( CMAKE_SCRIPT_MODE_FILE )
    report_version( ${CALAMARES_EXTENSIONS_VERSION} ${CMAKE_CURRENT_LIST_DIR} )
    return()
endif()

project(calamares-extensions
    VERSION ${CALAMARES_EXTENSIONS_VERSION}
    LANGUAGES CXX
)

set( CMAKE_CXX_STANDARD 17 )
set( CMAKE_CXX_STANDARD_REQUIRED ON )

# On developer's machine, the user package registry breaks
# consumers by loading the developer's config from a build
# directory (which doesn't have the rest of the config
# installed inside it).
set( CALAMARES_VERSION_REQUIRED 3.2.46 )
find_package(Calamares ${CALAMARES_VERSION_REQUIRED} NO_CMAKE_PACKAGE_REGISTRY)
if (NOT TARGET Calamares::calamares OR NOT TARGET Calamares::calamaresui)
    find_package(Calamares ${CALAMARES_VERSION_REQUIRED} REQUIRED)
endif()

### CMAKE SETUP
#
# Enable IN_LIST
if( POLICY CMP0057 )
    cmake_policy( SET CMP0057 NEW )
endif()
# Let ``AUTOMOC`` and ``AUTOUIC`` process ``GENERATED`` files.
if( POLICY CMP0071 )
    cmake_policy( SET CMP0071 NEW )
endif()
# Recognize more macros to trigger automoc
if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.10.0")
    list(APPEND CMAKE_AUTOMOC_MACRO_NAMES
        "K_PLUGIN_FACTORY_WITH_JSON"
        "K_EXPORT_PLASMA_DATAENGINE_WITH_JSON"
        "K_EXPORT_PLASMA_RUNNER"
    )
endif()
include( CTest )


### BRANDING
#
# Typically you would use only one branding, since that's
# the (single) branding for your distro.
#
calamares_add_branding_subdirectory( branding/default NAME default )
calamares_add_branding_subdirectory( branding/default-mobile NAME default-mobile )
calamares_add_branding_subdirectory( branding/fancy NAME fancy )

# This one has files in subdirectories
calamares_add_branding_subdirectory( branding/samegame NAME samegame SUBDIRECTORIES img )

# KaOS branding, with translations, note we can *NAME* something
# different from the source directory it lives in; this will be installed
# to a directory called *NAME* though -- and the `branding.desc` must
# have a *componentName* that matches this *NAME*.
calamares_add_branding_subdirectory( branding/kaos_branding NAME kaos )

### MODULES
#
# Add one of more modules, either C++ or Python.
#
set(LIST_SKIPPED_MODULES "")

calamares_add_module_subdirectory( modules/freebsddisk LIST_SKIPPED_MODULES )  # C++ viewmodule
calamares_add_module_subdirectory( modules/mobile LIST_SKIPPED_MODULES )
calamares_add_module_subdirectory( modules/os-freebsd LIST_SKIPPED_MODULES )
calamares_add_module_subdirectory( modules/os-nixos LIST_SKIPPED_MODULES )
calamares_add_module_subdirectory( modules/refind LIST_SKIPPED_MODULES )
calamares_add_module_subdirectory( modules/slowpython LIST_SKIPPED_MODULES )  # Python job
calamares_add_module_subdirectory( modules/unpackfsc LIST_SKIPPED_MODULES )

# If modules cannot be built, they usually call a macro
# which builds a list of explanations; show that list.
calamares_explain_skipped_modules( ${LIST_SKIPPED_MODULES} )
