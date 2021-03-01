cmake_minimum_required(VERSION 3.15)
project(redis++ VERSION 1.2.1)

include(CTest)

##
## Project options
##

option(redis++_USE_TLS "Build with TLS support" OFF)
option(redis++_BUILD_TESTING "Build tests for redis++" "${BUILD_TESTING}")

##
## CMake dependencies
##

set(THREADS_PREFER_PTHREAD_FLAG YES)

find_package(Threads REQUIRED)
find_package(hiredis REQUIRED)

##
## Main library build
##

if (NOT DEFINED CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 11)
endif ()

set(CMAKE_CXX_EXTENSIONS NO)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS YES)

add_library(
    redis++
    src/sw/redis++/command.cpp
    src/sw/redis++/command_options.cpp
    src/sw/redis++/connection.cpp
    src/sw/redis++/connection_pool.cpp
    src/sw/redis++/crc16.cpp
    src/sw/redis++/errors.cpp
    src/sw/redis++/pipeline.cpp
    src/sw/redis++/redis.cpp
    src/sw/redis++/redis_cluster.cpp
    src/sw/redis++/reply.cpp
    src/sw/redis++/sentinel.cpp
    src/sw/redis++/shards.cpp
    src/sw/redis++/shards_pool.cpp
    src/sw/redis++/subscriber.cpp
    src/sw/redis++/transaction.cpp
)
add_library(redis++::redis++ ALIAS redis++)

if (redis++_USE_TLS)
    message(STATUS "Building WITH TLS support")
    set(tls_headers "${CMAKE_CURRENT_SOURCE_DIR}/src/sw/redis++/tls")

    target_link_libraries(redis++ PUBLIC hiredis::hiredis_ssl)
    target_sources(redit++ PRIVATE src/sw/redis++/tls/tls.cpp)
else ()
    message(STATUS "Building WITHOUT TLS support")
    set(tls_headers "${CMAKE_CURRENT_SOURCE_DIR}/src/sw/redis++/no_tls")

    target_link_libraries(redis++ PUBLIC hiredis::hiredis)
endif ()

target_compile_features(redis++ PUBLIC cxx_std_${CMAKE_CXX_STANDARD})
target_compile_definitions(redis++ PRIVATE $<$<PLATFORM_ID:Windows>:NOMINMAX>)
target_link_libraries(redis++ PUBLIC Threads::Threads)
target_include_directories(
    redis++
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${tls_headers}>
)

set_target_properties(
    redis++
    PROPERTIES
    VERSION "${redis++_VERSION}"
    SOVERSION "${redis++_VERSION_MAJOR}"
)

##
## Testing
##

if (redis++_BUILD_TESTING)
    set(CMAKE_DISABLE_FIND_PACKAGE_redis++ YES)
    add_subdirectory(test)
endif ()

##
## Installation
##

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

## Custom locations

set(redis++_INSTALL_CMAKEDIR "${CMAKE_INSTALL_LIBDIR}/cmake/redis++"
    CACHE STRING "Path to redis++ CMake files")

## Type-specific targets

if (BUILD_SHARED_LIBS)
    set(type shared)
else ()
    set(type static)
endif ()

install(
    TARGETS redis++ EXPORT redis++-libs
    RUNTIME COMPONENT redis++-runtime
    LIBRARY COMPONENT redis++-runtime
    NAMELINK_COMPONENT redis++-development
    ARCHIVE COMPONENT redis++-development
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
    EXPORT redis++-libs
    DESTINATION "${redis++_INSTALL_CMAKEDIR}"
    NAMESPACE redis++::
    FILE redis++-${type}-targets.cmake
    COMPONENT redis++-development
)

write_basic_package_version_file(
    redis++-config-version.cmake
    COMPATIBILITY SameMajorVersion
)

install(
    FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/redis++-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/redis++-config-version.cmake"
    DESTINATION "${redis++_INSTALL_CMAKEDIR}"
    COMPONENT redis++-development
)

## Headers

install(
    DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/src/sw"
    TYPE INCLUDE
    COMPONENT redis++-development
    FILES_MATCHING
    PATTERN "*/tls" EXCLUDE
    PATTERN "*/no_tls" EXCLUDE
    PATTERN "*.h"
    PATTERN "*.hpp"
)

install(
    DIRECTORY "${tls_headers}/"
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/sw/redis++"
    COMPONENT redis++-development
    FILES_MATCHING PATTERN "*.h"
)
