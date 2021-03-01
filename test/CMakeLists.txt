cmake_minimum_required(VERSION 3.15)
project(test_redis++)

enable_testing()

# Normally this would be REQUIRED, but CMAKE_DISABLE_FIND_PACKAGE_<name>
# does not apply to "REQUIRED" find_package calls and we need to disable
# this from the parent project.
find_package(redis++)

add_executable(test_redis++ src/sw/redis++/test_main.cpp)
target_link_libraries(
    test_redis++
    PRIVATE
    redis++::redis++
    $<$<PLATFORM_ID:Solaris,SunOS>:socket>
    $<$<PLATFORM_ID:Windows>:ws2_32>
)

add_test(
    NAME test_redis++
    COMMAND test_redis++ -h 127.0.0.1 -p 6379
)

