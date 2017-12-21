# using recommented version cmake version, see http://doc.qt.io/qt-5/cmake-manual.html
cmake_minimum_required(VERSION 3.1.0)

project(Qt5widgetsApp LANGUAGES CXX)

# Request C++11 standard, using new CMake variables.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS False)

# Set symbol visibility defaults for all targets.
cmake_policy(SET CMP0063 NEW)
set(CMAKE_CXX_VISIBILITY_PRESET "hidden")
set(CMAKE_VISIBILITY_INLINES_HIDDEN True)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt5Widgets)

add_executable(${PROJECT_NAME} MACOSX_BUNDLE "main.cpp" "mainwindow.cpp")

message(STATUS "CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS: ${CMAKE_INSTALL_SYSTEM_RUNTIME_LIBS}")

target_link_libraries(${PROJECT_NAME} Qt5::Widgets)
