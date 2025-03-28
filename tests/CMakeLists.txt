# Copyright (c) Microsoft Corporation.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

set(STL_BUILD_ROOT "${PROJECT_BINARY_DIR}/out")
set(STL_SOURCE_DIR "${PROJECT_SOURCE_DIR}")
set(STL_TEST_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}")
set(STL_TEST_UTILS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/utils")
set(STL_TESTED_HEADERS_DIR "${STL_BUILD_ROOT}/inc")
set(STL_TESTED_MODULES_DIR "${STL_BUILD_ROOT}/modules")

set(LLVM_PROJECT_SOURCE_DIR "${STL_SOURCE_DIR}/llvm-project" CACHE PATH
    "Location of the llvm-project source tree")
set(LIBCXX_SOURCE_DIR "${LLVM_PROJECT_SOURCE_DIR}/libcxx" CACHE PATH
    "Location of the libcxx source tree")

option(TESTS_BUILD_ONLY "Only run the build steps of tests" OFF)

add_subdirectory(libcxx)
add_subdirectory(std)
add_subdirectory(tr1)

# Add the stl-lit subdirectory last so all the test directories have had a
# chance to add to the config map and test directory global properties.
add_subdirectory(utils/stl-lit)

find_package(Python "3.13" REQUIRED COMPONENTS Interpreter)

if(NOT DEFINED LIT_FLAGS)
    set(LIT_FLAGS "-o" "${CMAKE_CURRENT_BINARY_DIR}/test_results.json")
endif()

set(STL_LIT_COMMAND ${Python_EXECUTABLE} ${STL_LIT_OUTPUT} ${LIT_FLAGS})
get_property(STL_LIT_TEST_DIRS GLOBAL PROPERTY STL_LIT_TEST_DIRS)

add_custom_target(STL-CI COMMAND ${STL_LIT_COMMAND} -Dnotags=ASAN ${STL_LIT_TEST_DIRS} USES_TERMINAL)
add_custom_target(STL-ASan-CI COMMAND ${STL_LIT_COMMAND} -Dtags=ASAN ${STL_LIT_TEST_DIRS} USES_TERMINAL)
add_custom_target(test-only-edg COMMAND ${STL_LIT_COMMAND} -Dnotags=ASAN -Dtest-only-edg=True --xunit-xml-output test-results.xml ${STL_LIT_TEST_DIRS} USES_TERMINAL)
