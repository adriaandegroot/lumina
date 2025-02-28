# lumina-core build, CMake edition. EXPERIMENTAL.
#
# SPDX-License-Identifier: BSD-2-Clause
# SPDX-FileCopyrightText: Copyright 2022 Adriaan de Groot <adridg@FreeBSD.org>
cmake_minimum_required(VERSION 3.22 FATAL_ERROR)
project(lumina-core VERSION 1.6.2 LANGUAGES C CXX) # TODO: inherit this from top-level

# CMake Settings (policies to do AUTOMOC as much as possible)
cmake_policy(SET CMP0057 NEW)
cmake_policy(SET CMP0071 NEW)
include(GNUInstallDirs)

# C++ Settings (pretend everything is modern)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ECM setup
include(FeatureSummary)
find_package(ECM 5.88.0 NO_MODULE)
set_package_properties(
    ECM
    PROPERTIES
    TYPE REQUIRED
    DESCRIPTION "Extra CMake Modules."
    URL "https://commits.kde.org/extra-cmake-modules"
)
feature_summary(
    WHAT REQUIRED_PACKAGES_NOT_FOUND
    FATAL_ON_MISSING_REQUIRED_PACKAGES
)

set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

# Dependencies
set(QT_VERSION 5.15.0)
find_package(
    Qt5
    ${QT_VERSION}
    CONFIG
    REQUIRED
        Concurrent
        Core
        Gui
        Multimedia
        MultimediaWidgets
        Network
        Quick
        Qml
        Svg
        Widgets
        X11Extras
)
find_package(X11)
find_package(
    XCB
    MODULE
    COMPONENTS
        XCB
        KEYSYMS
        XKB
        XFIXES
        AUX
        COMPOSITE
        DAMAGE
        DPMS
        EWMH
        ICCCM
        IMAGE
        SCREENSAVER
    OPTIONAL_COMPONENTS XTEST
)
if(NOT X11_FOUND OR NOT XCB_XCB_FOUND)
    message(FATAL_ERROR "Required XCB component for X11 not found.")
endif()

# OS-detection
#
# This ignores the actual OS and just sets FreeBSD-useful values
add_definitions(
    -DL_ETCDIR=QStringLiteral\("${CMAKE_INSTALL_PREFIX}/etc"\)
    -DL_SHAREDIR=QStringLiteral\("${CMAKE_INSTALL_PREFIX}/share"\)
)

# Build options
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

# Lumina Components
add_subdirectory(libLumina)
add_subdirectory(lumina-desktop)
add_subdirectory(lumina-session)
add_subdirectory(lumina-open)
add_subdirectory(lumina-info)
add_subdirectory(lumina-pingcursor)
add_subdirectory(lumina-theme-engine)

# Additional files

install(
    FILES menu-scripts/README.md menu-scripts/ls.json.sh
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lumina-desktop/menu-scripts
)
install(
    FILES
        themes/DarkGlass.qss.template
        themes/Glass.qss.template
        themes/Lumina-default.qss.template
        themes/None.qss.template
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lumina-desktop/themes
)

# This is a typo, also in the original core.pro, should be xtrafiles/theme.cfg
#
#install(
#    FILES extrafiles/theme.cfg
#    DESTINATION ${CMAKE_INSTALL_DATADIR}/fluxbox/style/lumina-dark/theme.cfg
#)
install(
    FILES xtrafiles/globs2
    DESTINATION ${CMAKE_INSTALL_DATADIR}/lumina-desktop
)

# The icon theme lives outside of core, but should be
# installed along with it. The icon-themes here is the
# binary-directory to use during the build; there isn't
# much to do there except write Makefiles, since the
# icon themes do nothing but copy files out of the source-tree.
add_subdirectory(../../icon-theme icon-themes)
