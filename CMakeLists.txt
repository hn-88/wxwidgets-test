# begin basic metadata
# minimum CMake version required for C++20 support, among other things
cmake_minimum_required(VERSION 3.15)

# detect if MyHovercraftIsFullOfEels is being used as a sub-project of another CMake project
if(NOT DEFINED PROJECT_NAME)
    set(WX_WIDGETS_TEST_SUBPROJECT OFF)
else()
    set(WX_WIDGETS_TEST_SUBPROJECT ON)
endif()

project(wx-widgets-test VERSION 0.4.1 LANGUAGES CXX)

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

# Set a default build type if none was specified
set(DEFAULT_BUILD_TYPE "Release")

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# set some handy custom variables to detect Release-type builds from Debug-type ones
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(WX_WIDGETS_TEST_BUILD_DEBUG ON)
    set(WX_WIDGETS_TEST_BUILD_RELEASE OFF)
else()
    set(WX_WIDGETS_TEST_BUILD_DEBUG OFF)
    set(WX_WIDGETS_TEST_BUILD_RELEASE ON)
endif()

message(STATUS "[wx-widgets-test] Build Mode: ${CMAKE_BUILD_TYPE}")

# set the C++ standard to use to C++20 always
set(WX_WIDGETS_TEST_CXX_STANDARD "20")
message(STATUS "[wx-widgets-test] C++ Standard set to C++${WX_WIDGETS_TEST_CXX_STANDARD}")
set(CMAKE_CXX_STANDARD ${WX_WIDGETS_TEST_CXX_STANDARD})
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# end basic metadata

# This is a special target which only exists to capture compilation options
# used for wx-widgets-test and its tests. This is to avoid setting global compiler
# options which would be inherited by dependencies as well, which is bad
# because wx-widgets-test uses strict compiler warning options which not all other
# projects can build successfully with.
# Any target linked with this one will inherit the compiler options used for
# wx-widgets-test.
add_library(wx-widgets-test-compiler-options INTERFACE)

# used for enabling additional compiler options if supported
include(CheckCXXCompilerFlag)
# removed the warning flags to see if this will build in 2025

# add custom dependencies directory
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules/")

# a better way to load dependencies
include(CPM)

# add wxWidgets as dependency
CPMFindPackage(
    NAME wxWidgets
    GIT_REPOSITORY https://github.com/wxWidgets/wxWidgets
    GIT_TAG v3.1.6
    EXCLUDE_FROM_ALL YES
)

add_executable(demo WIN32 main.cpp)
# target_link_libraries(demo PRIVATE wx-widgets-test-compiler-options)
target_link_libraries(demo PRIVATE wx::net wx::core wx::base)
