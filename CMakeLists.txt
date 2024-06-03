cmake_minimum_required(VERSION 3.3)

# Set the CMAKE_PREFIX_PATH
if(NOT CMAKE_PREFIX_PATH)
    set(CMAKE_PREFIX_PATH "${CMAKE_SOURCE_DIR}/../wpeframework/4.4-r0/image/usr/lib/cmake/WPEFramework")
endif()

# Include the WPEFramework configuration
list(APPEND CMAKE_MODULE_PATH "${CMAKE_PREFIX_PATH}")

# Find the WPEFramework package


# Define the project
project(PackageManager VERSION 1.0.0)

# Set module name
set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})
message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

# Include additional paths if necessary
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

# Set plugin options
set(PLUGIN_PACKAGEMANAGER_AUTOSTART "true" CACHE STRING "Automatically start PackageManager plugin")
set(PLUGIN_PACKAGEMANAGER_MODE "Off" CACHE STRING "Controls if the plugin should run in its own process, in process or remote.")
set(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION "ExternalService" CACHE STRING "Controls which implementation should be used for the PackageManager.")
option(PLUGIN_PACKAGEMANAGER_EXAMPLES "Controls if the example code should be built." OFF)

# Find necessary WPEFramework components
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

# Add the library
add_library(${MODULE_NAME} SHARED
    PackageManager.cpp
    Module.cpp)

# Set target properties
set_target_properties(${MODULE_NAME} PROPERTIES
    CXX_STANDARD 11
    CXX_STANDARD_REQUIRED YES)

# Link the necessary libraries
target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions)

# Add source files based on the implementation choice
if(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION STREQUAL "Test")
    target_sources(${MODULE_NAME}
    PRIVATE
        TestImplementation/TestImplementation.cpp)
elseif(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION STREQUAL "ExternalService")
    target_sources(${MODULE_NAME}
    PRIVATE
        ExternalServiceImplementation/ExternalServiceImplementation.cpp)

    if(PLUGIN_PACKAGEMANAGER_EXAMPLES)
        add_subdirectory(ExternalServiceImplementation/ExampleClient)
    endif()
else ()
    message(FATAL_ERROR "There is no valid implementation provided for the PackageManager")
endif ()

# Install the target
install(TARGETS ${MODULE_NAME}
    DESTINATION lib/${STORAGE_DIRECTORY}/plugins)

# Function to write configuration (if exists)
if(COMMAND write_config)
    write_config()
endif()
