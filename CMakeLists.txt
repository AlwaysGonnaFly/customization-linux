project(applet_window_appmenu)

cmake_minimum_required(VERSION 3.16 FATAL_ERROR)

set(CMAKE_CXX_STANDARD 17)
set(VERSION 0.8.0)
set(AUTHOR "Michail Vourlakos")
set(EMAIL "mvourlakos@gmail.com")

set(QT_MIN_VERSION "6.2.0")
set(KF6_MIN_VERSION "6.0.0")

find_package(ECM ${KF6_MIN_VERSION} REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR})

find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED Quick Widgets DBus)

find_package(KF6 ${KF6_MIN_VERSION} REQUIRED COMPONENTS Plasma WindowSystem ConfigWidgets)
find_package(KF6Wayland QUIET) # Опциональная проверка для Wayland

if (KF6Wayland_FOUND)
    message(STATUS "Found KF6Wayland: ${KF6Wayland_VERSION}")
else()
    message(WARNING "KF6Wayland not found! Wayland-related features will be disabled.")
endif()

find_package(KDecoration2 REQUIRED)
find_package(LibTaskManager)

include(ECMQtDeclareLoggingCategory)
include(KDECMakeSettings)
include(KDECompilerSettings)

# Load X11 libraries
find_package(X11)
set_package_properties(X11 PROPERTIES DESCRIPTION "X11 libraries"
                        URL "http://www.x.org"
                        TYPE REQUIRED
                        PURPOSE "Required for building the X11 based workspace")

if(X11_FOUND)
  find_package(XCB MODULE REQUIRED COMPONENTS XCB RANDR)
  set_package_properties(XCB PROPERTIES TYPE REQUIRED)
  if(NOT X11_SM_FOUND)
    message(FATAL_ERROR "\nThe X11 Session Management (SM) development package could not be found.\nPlease install libSM.\n")
  endif()

  find_package(Qt6 ${QT_MIN_VERSION} CONFIG REQUIRED COMPONENTS X11Extras)
endif()

if(X11_FOUND AND XCB_XCB_FOUND)
  set(HAVE_X11 ON)
endif()

configure_file(config-appmenu.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/plugin/config-appmenu.h)
configure_file(config-appmenu.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/lib/config-appmenu.h)

include(KDEInstallDirs)

add_subdirectory(lib)
add_subdirectory(plugin)
plasma_install_package(package org.kde.windowappmenu)
