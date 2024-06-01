cmake_minimum_required(VERSION 3.3)

# Define the project and its version
project(PackageManager VERSION 1.0.0)

# Add the necessary module paths
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

# Find the required packages
find_package(WPEFramework REQUIRED)

set(MODULE_NAME ${NAMESPACE}${PROJECT_NAME})
message("Setup ${MODULE_NAME} v${PROJECT_VERSION}")

# Set configuration options
set(PLUGIN_PACKAGEMANAGER_AUTOSTART "true" CACHE STRING "Automatically start PackageManager plugin")
set(PLUGIN_PACKAGEMANAGER_MODE "Off" CACHE STRING "Controls if the plugin should run in its own process, in process or remote.")
set(PLUGIN_PACKAGEMANAGER_IMPLEMENTATION "ExternalService" CACHE STRING "Controls which implementation should be used for the PackageManager.")
option(PLUGIN_PACKAGEMANAGER_EXAMPLES "Controls if the example code should be build." OFF)

# Find necessary plugins and definitions
find_package(${NAMESPACE}Plugins REQUIRED)
find_package(${NAMESPACE}Definitions REQUIRED)
find_package(CompileSettingsDebug CONFIG REQUIRED)

# Define the library target
add_library(${MODULE_NAME} SHARED
    PackageManager.cpp
    Module.cpp)

# Set target properties
set_target_properties(${MODULE_NAME} PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED YES)

# Link the target with necessary libraries
target_link_libraries(${MODULE_NAME}
    PRIVATE
        CompileSettingsDebug::CompileSettingsDebug
        ${NAMESPACE}Plugins::${NAMESPACE}Plugins
        ${NAMESPACE}Definitions::${NAMESPACE}Definitions)

# Handle different implementations
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

# Write the configuration
write_config()

