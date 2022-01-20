# Copyright (c) 2021-2022, Yongjie Zhu.  All rights reserved.

# This a template for cmake

cmake_minimum_required(VERSION 3.19)

set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

project(
    XXX
    VERSION 1.0
    DESCRIPTION "XXX"
    LANGUAGES C CXX CUDA)

set(XXX_VERSION "${CMAKE_PROJECT_VERSION}")

option(XXX_BUILD_WITH_GUI "Build with GUI support (requires GLFW and GLEW)?" ON)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# ##################################################################################################
# Build type and C++ compiler setup
# ##################################################################################################

# set a default configuration if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "No release type specified. Setting to 'Release'.")
    set(CMAKE_BUILD_TYPE
        Release
        CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo")
endif()

if(APPLE)
    set(CMAKE_MACOSX_RPATH ON)
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D_CRT_SECURE_NO_WARNINGS")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP24")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS OFF)

# ##################################################################################################
# CUDA compiler setup
# ##################################################################################################

set(CMAKE_CUDA_STANDARD 14)
set(CMAKE_CUDA_STANDARD_REQUIRED ON)
set(CMAKE_CUDA_EXTENSIONS OFF)
set(CUDA_LINK_LIBRARIES_KEYWORD PUBLIC)

if(MSVC)
    list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-bigobj")
else()
    list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-mf16c")
    list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-Wno-float-conversion")
    list(APPEND CUDA_NVCC_FLAGS "-Xcompiler=-fno-strict-aliasing")
    list(APPEND CUDA_NVCC_FLAGS "-fPIC")
    set(CUDA_TOOLKIT_ROOT_DIR /opt/cuda/targets/x86_64-linux)
endif()
list(APPEND CUDA_NVCC_FLAGS "--extended-lambda")
list(APPEND CUDA_NVCC_FLAGS "--expt-relaxed-constexpr")

# ##################################################################################################
# Dependencies (ordered alphabetically)
# ##################################################################################################
