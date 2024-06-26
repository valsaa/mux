# CMake version is 3.20 to be able to use C++ 23
# https://cmake.org/cmake/help/latest/prop_tgt/CXX_STANDARD.html
cmake_minimum_required(VERSION 3.20)

# Project name
project(mux)

# Enable google test
enable_testing()

# Project directories
get_filename_component(root_dir ${CMAKE_CURRENT_SOURCE_DIR} REALPATH)
message("-- root_dir is ${root_dir}")
cmake_path(APPEND cmake_dir ${root_dir} "cmake")
cmake_path(APPEND src_dir ${root_dir} "src")
cmake_path(APPEND tests_dir ${root_dir} "tests/unit")
cmake_path(APPEND bin_dir ${CMAKE_CURRENT_BINARY_DIR} "bin")

# Includes
include("${cmake_dir}/third_party.cmake")
include("${cmake_dir}/utils.cmake")
message("-- Custom CMake files included")

# Core lib target
cmake_path(APPEND core_dir ${src_dir} "core")
file(GLOB core_SOURCES "${core_dir}/*.cpp")
add_library(core ${core_SOURCES})
target_compile_features(core PRIVATE cxx_std_23)
target_include_directories(core PUBLIC ${core_dir})
target_link_libraries(core
    PUBLIC
    spdlog::spdlog
)
_set_compilers_options(core)

# Mux bin target
file(GLOB mux_SOURCES "${src_dir}/*.cpp")
add_executable(mux ${mux_SOURCES})
set_target_properties(mux PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${bin_dir})
target_compile_features(mux PRIVATE cxx_std_23)
target_link_libraries(mux
    PRIVATE
    cxxopts
    spdlog::spdlog
    core
)
_set_compilers_options(mux)

# Mux test target
file(GLOB tests_SOURCE "${tests_dir}/*.cpp")
add_executable(mux_test ${tests_SOURCE})
target_link_libraries(mux_test
    PRIVATE
    GTest::gtest_main
    core
)

include(GoogleTest)
gtest_discover_tests(mux_test)
