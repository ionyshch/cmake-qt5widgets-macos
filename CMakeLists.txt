# using recommented version cmake version, see http://doc.qt.io/qt-5/cmake-manual.html
cmake_minimum_required(VERSION 3.1.0)

project(cmake-qt5widgets-macos LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

find_package(Qt5Widgets)

add_executable(${PROJECT_NAME} MACOSX_BUNDLE "main.cpp" "mainwindow.cpp")

target_link_libraries(${PROJECT_NAME} Qt5::Widgets)
