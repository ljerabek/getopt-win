cmake_minimum_required(VERSION 3.12)

# Ensure this project only builds on Windows
if(NOT WIN32)
  message(FATAL_ERROR [=[
Expected the target system to be Windows.

This software is a modification of the Free Software Foundation, Inc. getopt
library for parsing command line arguments and its purpose is to provide a
Microsoft Visual C friendly derivative.]=]
  )
endif()

# Project configuration
project(getopt-win
  VERSION 1.1.0
  DESCRIPTION "Full getopt port for unicode and multibyte Windows applications"
  HOMEPAGE_URL "https://github.com/ludvikjerabek/getopt-win"
  LANGUAGES C
)

include(GNUInstallDirs)

# Build options
option(BUILD_SHARED_LIB "Build the shared library" ON)
option(BUILD_STATIC_LIB "Build the static library" ON)
option(BUILD_TESTING "Build the testing tree." ON)

# Shared library target
if(BUILD_SHARED_LIB)
  add_library(getopt_shared SHARED src/getopt.c)
  target_compile_definitions(getopt_shared PRIVATE EXPORTS_GETOPT)
  target_include_directories(getopt_shared PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
  )
  set_target_properties(getopt_shared PROPERTIES
    OUTPUT_NAME getopt
    PUBLIC_HEADER src/getopt.h
    VERSION ${PROJECT_VERSION}
  )
  add_library(${PROJECT_NAME}::getopt_shared ALIAS getopt_shared)
endif()

# Static library target
if(BUILD_STATIC_LIB)
  add_library(getopt_static STATIC src/getopt.c)
  target_compile_definitions(getopt_static PRIVATE STATIC_GETOPT)
  target_include_directories(getopt_static PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}>
  )
  set_target_properties(getopt_static PROPERTIES
    OUTPUT_NAME getopt_static
    PUBLIC_HEADER src/getopt.h
    VERSION ${PROJECT_VERSION}
  )
  add_library(${PROJECT_NAME}::getopt_static ALIAS getopt_static)
endif()

# Example programs for shared and static libraries
if(BUILD_SHARED_LIB)
  add_executable(example_shared EXCLUDE_FROM_ALL tests/main.c)
  target_link_libraries(example_shared PRIVATE getopt_shared)
endif()

if(BUILD_STATIC_LIB)
  add_executable(example_static EXCLUDE_FROM_ALL tests/main.c)
  target_link_libraries(example_static PRIVATE getopt_static)
  target_compile_definitions(example_static PRIVATE STATIC_GETOPT)
endif()

# Testing setup
if(BUILD_TESTING)
  enable_testing()
  
  # Test for shared library
  if(BUILD_SHARED_LIB)
    add_test(NAME example_shared
      COMMAND example_shared --add -b -c hello -d 42 --verbose nonopt1 nonopt2
    )
    set_tests_properties(example_shared PROPERTIES
      PASS_REGULAR_EXPRESSION [=[
option -a
option -b
option -c with value `hello'
option -d with value `42'
verbose flag is set
non-option ARGV-elements: nonopt1 nonopt2]=]
    )
  endif()
  
  # Test for static library
  if(BUILD_STATIC_LIB)
    add_test(NAME example_static
      COMMAND example_static --add -b -c hello -d 42 --verbose nonopt1 nonopt2
    )
    set_tests_properties(example_static PROPERTIES
      PASS_REGULAR_EXPRESSION [=[
option -a
option -b
option -c with value `hello'
option -d with value `42'
verbose flag is set
non-option ARGV-elements: nonopt1 nonopt2]=]
    )
  endif()
endif()

# Installation rules for the shared library
if(BUILD_SHARED_LIB)
  install(TARGETS getopt_shared
    EXPORT ${PROJECT_NAME}-targets
    ARCHIVE COMPONENT Library
    RUNTIME COMPONENT Library
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    COMPONENT Library
  )
endif()

# Installation rules for the static library
if(BUILD_STATIC_LIB)
  install(TARGETS getopt_static
    EXPORT ${PROJECT_NAME}-targets
    ARCHIVE COMPONENT Library
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}
    COMPONENT Library
  )
endif()

# CMake package export/install for external usage
install(EXPORT ${PROJECT_NAME}-targets
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
  NAMESPACE ${PROJECT_NAME}::
  COMPONENT Config
)

include(CMakePackageConfigHelpers)

# Configure package config file
configure_package_config_file(${PROJECT_NAME}-config.cmake.in
  ${PROJECT_NAME}-config.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  COMPATIBILITY SameMajorVersion
)

# Install the generated CMake config files
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
  COMPONENT Config
)
