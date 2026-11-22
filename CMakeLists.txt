# Example CMake command line to create project build files:
#
# cmake -B build .
#
# DelegateMQ is fetched from GitHub at the release tag DMQ_GIT_TAG when CMake
# configures. To build against a local DelegateMQ checkout instead, add
# -DFETCHCONTENT_SOURCE_DIR_DELEGATEMQ=<path>. To configure again without
# network access, add -DFETCHCONTENT_FULLY_DISCONNECTED=ON.

cmake_minimum_required(VERSION 3.16)

project(StateMachine VERSION 1.1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# DelegateMQ build options, set before DelegateMQ is added
set(DMQ_ALLOCATOR "OFF")
set(DMQ_UTIL "ON")
set(DMQ_THREAD "DMQ_THREAD_STDLIB")
set(DMQ_SERIALIZE "DMQ_SERIALIZE_NONE")
set(DMQ_TRANSPORT "DMQ_TRANSPORT_NONE")

# DelegateMQ release to build against
set(DMQ_GIT_TAG "v3.0.0" CACHE STRING "DelegateMQ release tag (or branch) to fetch")

include(FetchContent)
FetchContent_Declare(
    delegatemq
    GIT_REPOSITORY https://github.com/DelegateMQ/DelegateMQ.git
    GIT_TAG        ${DMQ_GIT_TAG}
    GIT_SHALLOW    TRUE
)
FetchContent_MakeAvailable(delegatemq)

# Collect project source files
file(GLOB SOURCES      "${CMAKE_SOURCE_DIR}/main.cpp")
file(GLOB SM_SOURCES   "${CMAKE_SOURCE_DIR}/state-machine/*.cpp"
                       "${CMAKE_SOURCE_DIR}/state-machine/*.h")
file(GLOB EXAMPLE_SOURCES "${CMAKE_SOURCE_DIR}/examples/*.cpp"
                          "${CMAKE_SOURCE_DIR}/examples/*.h")
file(GLOB TEST_SOURCES "${CMAKE_SOURCE_DIR}/unit-tests/*.cpp"
                       "${CMAKE_SOURCE_DIR}/unit-tests/*.h")

list(APPEND SOURCES ${SM_SOURCES} ${EXAMPLE_SOURCES} ${TEST_SOURCES})

# Organize files in IDE (Visual Studio Solution Explorer)
source_group("State Machine"   FILES ${SM_SOURCES})
source_group("Examples"        FILES ${EXAMPLE_SOURCES})
source_group("Unit Tests"      FILES ${TEST_SOURCES})

add_executable(StateMachineApp ${SOURCES})

target_include_directories(StateMachineApp PRIVATE
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_SOURCE_DIR}/state-machine
    ${CMAKE_SOURCE_DIR}/examples
)

# DelegateMQ's include directories, compile definitions and library
target_link_libraries(StateMachineApp PRIVATE DelegateMQ::DelegateMQ)

# -----------------------------------------------------------------------------
# PRECOMPILED HEADERS (PCH)
# This single command works for both MSVC (Windows) and GCC/Clang (Linux).
# It compiles DelegateMQ.h once and auto-includes it into every source file
# in the target, significantly speeding up compilation.
# -----------------------------------------------------------------------------
target_precompile_headers(StateMachineApp PRIVATE
    "${delegatemq_SOURCE_DIR}/src/delegate-mq/DelegateMQ.h"
)
