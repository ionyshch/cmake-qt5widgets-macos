# Copyright (c) 2017 Ivan Onyshchenko

# using recommended cmake version, see http://doc.qt.io/qt-5/cmake-manual.html
cmake_minimum_required(VERSION 3.1.0)

set(CMAKE_OSX_DEPLOYMENT_TARGET "10.10" CACHE STRING "Minimum OS X deployment version")
project(Qt5widgetsApp LANGUAGES CXX)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Request C++11 standard, using new CMake variables.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS False)

# Set symbol visibility defaults for all targets.
cmake_policy(SET CMP0063 NEW)
set(CMAKE_CXX_VISIBILITY_PRESET "hidden")
set(CMAKE_VISIBILITY_INLINES_HIDDEN True)

# Automatically add the current source- and build directories to the include path
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

# includes section
include(InstallLocation)

# TODO set app version

if(APPLE)
    set(MACOSX_BUNDLE_NAME "${PROJECT_NAME}")
    set(prefix "${MACOSX_BUNDLE_NAME}.app/Contents")
    set(INSTALL_INCLUDE_DIR "${prefix}/${INSTALL_INCLUDE_DIR}")
    set(INSTALL_RUNTIME_DIR "${prefix}/MacOS")
    set(INSTALL_LIBRARY_DIR "${prefix}/${INSTALL_LIBRARY_DIR}")
    set(INSTALL_ARCHIVE_DIR "${prefix}/${INSTALL_ARCHIVE_DIR}")
    set(INSTALL_DATA_DIR    "${prefix}/${INSTALL_DATA_DIR}")
    set(INSTALL_DOC_DIR     "${prefix}/${INSTALL_DOC_DIR}")
    set(INSTALL_CMAKE_DIR   "${prefix}/Resources")
endif()

find_package(Qt5 COMPONENTS Widgets REQUIRED)

add_executable(${PROJECT_NAME} MACOSX_BUNDLE "main.cpp" "mainwindow.cpp")
qt5_use_modules(${PROJECT_NAME} Widgets)

target_link_libraries(${PROJECT_NAME} Qt5::Widgets)

add_custom_target(Docs SOURCES README.md .gitignore LICENSE)

# based on code from CMake's QtDialog/CMakeLists.txt
macro(install_qt5_plugin _qt_plugin_name _qt_plugins_var)
  get_target_property(_qt_plugin_path "${_qt_plugin_name}" LOCATION)
  if(EXISTS "${_qt_plugin_path}")
    get_filename_component(_qt_plugin_file "${_qt_plugin_path}" NAME)
    get_filename_component(_qt_plugin_type "${_qt_plugin_path}" PATH)
    get_filename_component(_qt_plugin_type "${_qt_plugin_type}" NAME)
    if(APPLE)
      set(_qt_plugin_dir "PlugIns")
    elseif(WIN32)
      set(_qt_plugin_dir "plugins")
    endif()
    set(_qt_plugin_dest "${_qt_plugin_dir}/${_qt_plugin_type}")
    install(FILES "${_qt_plugin_path}"
      DESTINATION "${prefix}/${_qt_plugin_dest}"
      ${COMPONENT})
    set(${_qt_plugins_var}
      "${${_qt_plugins_var}};\${CMAKE_INSTALL_PREFIX}/${prefix}/${_qt_plugin_dest}/${_qt_plugin_file}")
  else()
    message(FATAL_ERROR "QT plugin ${_qt_plugin_name} not found")
  endif()
endmacro()

if(APPLE)
  install_qt5_plugin("Qt5::QCocoaIntegrationPlugin" QT_PLUGINS)
  file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/qt.conf"
    "[Paths]\nPlugins = ${_qt_plugin_dir}\n")
  install(FILES "${CMAKE_CURRENT_BINARY_DIR}/qt.conf"
    DESTINATION "${INSTALL_CMAKE_DIR}"
    ${COMPONENT})
endif()


# Destination paths below are relative to ${CMAKE_INSTALL_PREFIX}
install(TARGETS ${PROJECT_NAME}
    BUNDLE DESTINATION . COMPONENT Runtime
    RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR} COMPONENT Runtime
    )

# Note Mac specific extension .app
set(APPS "\${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}.app")

# Directories to look for dependencies
set(DIRS "${CMAKE_BINARY_DIR}")

if(CMAKE_PREFIX_PATH)
    foreach(dir ${CMAKE_PREFIX_PATH})
        list(APPEND DIRS "${dir}/bin" "${dir}/lib")
    endforeach()
endif()

include(InstallRequiredSystemLibraries)

message(STATUS "APPS: ${APPS}")
message(STATUS "QT_PLUGINS: ${QT_PLUGINS}")
message(STATUS "DIRS: ${DIRS}")

install(CODE "include(BundleUtilities)
    fixup_bundle(\"${APPS}\" \"${QT_PLUGINS}\" \"${DIRS}\")")

set(CPACK_GENERATOR "DRAGNDROP")
include(CPack)
