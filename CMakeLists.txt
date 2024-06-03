# Copyright 2023 Comcast Cable Communications Management, LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0
#

cmake_minimum_required(VERSION 3.3)

project(PackageManager)

# Ensure CMAKE_MODULE_PATH is set correctly before find_package
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

find_package(WPEFramework REQUIRED)

# Define project version
set(VERSION_MAJOR 1)
set(VERSION_MINOR 0)
set(VERSION_PATCH 0)
set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})

# Set the module name
set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})
message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

# Plugin options
set(PLUGIN_PACKAGEMANAGER_AUTOSTART "true" CACHE STRING "Automatically start PackageManager plugin")
set(PLUGIN_PACKAGEMANAGER_MODE "Off" CACHE STRING "Controls if the plugin should run in its own process, in process or remote.")
set(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION "ExternalService" CACHE STRING "Controls which implementation should be used for the PackageManager.")
option(PLUGIN_PACKAGEMANAGER_EXAMPLES "Controls if the example code should be built." OFF)

# Find necessary packages
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

# Add the PackageManager library
add_library(${MODULE_NAME} SHARED
    PackageManager.cpp
    Module.cpp)

# Set target properties
set_target_properties(${MODULE_NAME} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED YES)

# Link libraries
target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions)

# Select the implementation
if(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION STREQUAL "Test")
    target_sources(${MODULE_NAME} PRIVATE TestImplementation/TestImplementation.cpp)
elseif(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION STREQUAL "ExternalService")
    target_sources(${MODULE_NAME} PRIVATE ExternalServiceImplementation/ExternalServiceImplementation.cpp)

    if(PLUGIN_PACKAGEMANAGER_EXAMPLES)
        add_subdirectory(ExternalServiceImplementation/ExampleClient)
    endif()
else()
    message(FATAL_ERROR "There is no valid implementation provided for the PackageManager")
endif()

# Install the target
install(TARGETS ${MODULE_NAME}
    DESTINATION lib/${STORAGE_DIRECTORY}/plugins)

# Generate configuration
write_config()
