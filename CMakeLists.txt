# Vertex Package Manager (VPM) - CMake Build Configuration
# Licensed under GNU General Public License v2.0 or later

cmake_minimum_required(VERSION 3.13)
project(vpm)

# Generic header locations
include_directories(${PROJECT_BINARY_DIR}/include)

# Enable testing
enable_testing()

# Build options
option(REQUIRE_MERGED_USR "Require merged-usr." ON)
option(WITH_DOC "Build all documentation." ON)
option(WITH_TESTS "Build tests" ON)
option(USE_NLS "Localisation support." ON)

# CMake modules path
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMake")

# C++ Standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)

# Compiler flags
add_optional_compile_options(Wall)
add_optional_compile_options(Wextra)
add_optional_compile_options(Wcast-align)
add_optional_compile_options(Wlogical-op)
add_optional_compile_options(Werror=return-type)

# Find required packages
find_package(Threads REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(ZLIB REQUIRED)
find_package(BZip2 REQUIRED)
find_package(LZMA REQUIRED)
find_package(LZ4 REQUIRED)
find_package(Perl REQUIRED)

# Optional packages
find_package(ZSTD)
find_package(Udev)
find_package(Systemd)
find_package(SECCOMP)

# Project version
set(PACKAGE ${PROJECT_NAME})
set(PACKAGE_MAIL "VPM Development Team")
set(PACKAGE_VERSION "1.0.0")
string(REGEX MATCH "^[0-9.]+" PROJECT_VERSION ${PACKAGE_VERSION})

# Set directories
set(STATE_DIR "${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/lib/vpm" CACHE PATH "VPM state directory")
set(CACHE_DIR "${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/cache/vpm" CACHE PATH "VPM cache directory")
set(LOG_DIR "${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/log/vpm" CACHE PATH "VPM log directory")
set(CONF_DIR "${CMAKE_INSTALL_FULL_SYSCONFDIR}/vpm" CACHE PATH "VPM config directory")
set(BIN_DIR "${CMAKE_INSTALL_FULL_BINDIR}")

# Configure headers
configure_file(CMake/config.h.in ${PROJECT_BINARY_DIR}/include/config.h)
configure_file(CMake/vpmi18n.h.in ${PROJECT_BINARY_DIR}/include/vpmi18n.h)

# Add subdirectories
add_subdirectory(vendor)
if(NOT USE_SYSTEM_VPMPKG)
  add_subdirectory(vpm-pkg)
  add_subdirectory(vpm-private)
endif()
add_subdirectory(cmdline)
add_subdirectory(completions)
add_subdirectory(doc)
add_subdirectory(methods)
add_subdirectory(ftparchive)
add_subdirectory(test)

if(USE_NLS)
  add_subdirectory(po)
endif()

# Create directories
install_empty_directories(
  ${CONF_DIR}/vpm.conf.d
  ${CONF_DIR}/auth.conf.d
  ${CONF_DIR}/preferences.d
  ${CONF_DIR}/sources.list.d
  ${CONF_DIR}/trusted.gpg.d
  ${CACHE_DIR}/archives/partial
  ${STATE_DIR}/lists/partial
  ${STATE_DIR}/mirrors/partial
  ${STATE_DIR}/periodic
  ${LOG_DIR}
)
