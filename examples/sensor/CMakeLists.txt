# Anbox - The Android in a Box runtime environment
# Copyright 2019 Canonical Ltd.  All rights reserved.

project(ANBOX_PLATFORM_PLUGIN_SENSOR)
cmake_minimum_required(VERSION 3.1.3)

include(CTest)
enable_testing()

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default to release")
    set(CMAKE_BUILD_TYPE "release")
endif()

# Load the anbox-sdk cmake package if we are running stand-alone
# Otherwise if we are running inside the SDK build itself, we can't include
# the SDK obviously because it hasn't been built yet
if("${CMAKE_PROJECT_NAME}" STREQUAL "ANBOX_PLATFORM_PLUGIN_SENSOR")
  find_package(anbox-platform-sdk REQUIRED)
endif()

set(PLATFORM_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/anbox/platforms/custom)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DPLATFORM_INSTALL_DIR=\\\"${CMAKE_INSTALL_PREFIX}/${PLATFORM_INSTALL_DIR}\\\"")

# Add the plugin target
set(SENSOR_PLATFORM_SOURCES sensor_platform.cpp)
add_library(AnboxSensorPlatform
    SHARED ${SENSOR_PLATFORM_SOURCES})

# Link against the anbox-platform-sdk-internal library
target_link_libraries(AnboxSensorPlatform PUBLIC
  anbox-platform-sdk-internal)

# Need to include the build directory for the configured file "arch.h"
target_include_directories(AnboxSensorPlatform
  PUBLIC ${CMAKE_BINARY_DIR})

# All platforms need to follow the naming scheme platform_<name>.so
set_target_properties(
  AnboxSensorPlatform PROPERTIES
  OUTPUT_NAME platform_sensor
  PREFIX ""
  SUFFIX ".so"
  COMPILE_FLAGS "-fPIC -std=c++14"
  INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${PLATFORM_INSTALL_DIR}")

# Run the validation test suites against the platform implementation
add_test(NAME SensorPlatformValidation
    COMMAND ${ANBOX_PLATFORM_TESTER} --gtest_filter=-PlatformInputProcessorTest.*:PlatformAudioProcessorTest.*:PlatformGraphicsProcessorTest.*:PlatformGpsProcessorTest.*:PlatformCameraProcessorTest.*:PlatformProxyTest.SendMessageWhenSet ${CMAKE_CURRENT_BINARY_DIR}/platform_sensor.so)
