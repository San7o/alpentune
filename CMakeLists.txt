cmake_minimum_required(VERSION 3.20)
project(
    alpentune 
    VERSION 1.0
    LANGUAGES C CXX
)

set(CMAKE_CXX_STANDARD 23 REQUIRED)
set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)

set(AT_SOURCES src/alpentune.c)
set(AT_HEADERS include)
set(AT_COMPILE_OPTIONS -Wall -Wextra -Wpedantic -Werror -Wconversion -Wshadow)
set(AT_TEST_SOURCES
        tests/alpentune_test.cpp
        fuzz/alpentune_fuzz.cpp
        benchmarks/alpentune_benchmark.cpp
)
set(AT_TEST_HEADERS)
set(AT_LINK_LIBRARIES)

option(AT_USE_CLANG "Use clang" OFF)
option(AT_BUILD_TESTS "Build tests" ON)
option(AT_BUILD_SHARED "Build shared library" ON)
option(AT_BUILD_STATIC "Build static library" OFF)
option(AT_BUILD_OPTIMIZED "Build optimized" OFF)
option(AT_BUILD_OPTIMIZED_AGGRESSIVE "Build with maximum optimization" OFF)

set(ENV{CPM_SOURCE_CACHE} ${CMAKE_CURRENT_SOURCE_DIR}/deps)
include(cmake/CPM.cmake) # <3
CPMUsePackageLock(package-lock.cmake)
CPMAddPackage(
  NAME Ccache.cmake
  GITHUB_REPOSITORY TheLartians/Ccache.cmake
  VERSION 1.2
)
CPMAddPackage(
    NAME valfuzz
    GITHUB_REPOSITORY San7o/valFuzz
    GIT_TAG v1.0.1
    DOWNLOAD_ONLY True
)
if (valfuzz_ADDED)
    list(APPEND AT_TEST_SOURCES ${valfuzz_SOURCE_DIR}/src/valfuzz.cpp)
    list(APPEND AT_TEST_HEADERS ${valfuzz_SOURCE_DIR}/include)
endif()

if(AT_USE_CLANG)
    set(CMAKE_CXX_COMPILER clang++)
    list(APPEND AT_COMPILE_OPTIONS -std=c++23 -fexperimental-library)
    list(APPEND AT_LINK_LIBRARIES -fexperimental-library)
endif()

if(AT_BUILD_SHARED)
    add_library(alpentune_shared SHARED ${AT_SOURCES})
    target_include_directories(alpentune_shared PRIVATE ${AT_HEADERS})
    target_compile_options(alpentune_shared PRIVATE ${AT_COMPILE_OPTIONS})
endif()

if(AT_BUILD_STATIC)
    add_library(alpentune_static STATIC ${AT_SOURCES})
    target_include_directories(alpentune_static PRIVATE ${AT_HEADERS})
    target_compile_options(alpentune_static PRIVATE ${AT_COMPILE_OPTIONS})
endif()

if(AT_BUILD_TESTS AND valfuzz_ADDED)
    add_executable(tests ${AT_SOURCES} ${AT_TEST_SOURCES})
    target_include_directories(tests PRIVATE ${AT_HEADERS} ${AT_TEST_HEADERS})
    target_compile_options(tests PRIVATE ${AT_COMPILE_OPTIONS})
    set_target_properties(tests PROPERTIES INTERFACE_INCLUDE_DIRECTORIES build/libalpentune_shared.so)
endif()
