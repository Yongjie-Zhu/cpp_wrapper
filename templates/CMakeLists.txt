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

include_directories("3rdparty")

# Eigen #

include_directories("3rdparty/eigen")
include_directories("3rdparty/filesystem")

if(XXX_BUILD_WITH_GUI)

    # GLFW #

    set(GLFW_BUILD_EXAMPLES
        OFF
        CACHE BOOL " " FORCE)
    set(GLFW_BUILD_TESTS
        OFF
        CACHE BOOL " " FORCE)
    set(GLFW_BUILD_DOCS
        OFF
        CACHE BOOL " " FORCE)
    set(GLFW_BUILD_INSTALL
        OFF
        CACHE BOOL " " FORCE)
    set(GLFW_INSTALL
        OFF
        CACHE BOOL " " FORCE)
    set(GLFW_USE_CHDIR
        OFF
        CACHE BOOL " " FORCE)
    set(BUILD_SHARED_LIBS
        ON
        CACHE BOOL " " FORCE)

    add_subdirectory(3rdparty/glfw)

    set_target_properties(glfw PROPERTIES EXCLUDE_FROM_ALL 1 EXCLUDE_FROM_DEFAULT_BUILD 1)

    mark_as_advanced(
        GLFW_BUILD_DOCS
        GLFW_BUILD_EXAMPLES
        GLFW_BUILD_INSTALL
        GLFW_BUILD_TESTS
        GLFW_DOCUMENT_INTERNALS
        GLFW_INSTALL
        GLFW_USE_CHDIR
        GLFW_USE_MENUBAR
        GLFW_USE_OSMESA
        GLFW_VULKAN_STATIC
        GLFW_USE_RETINA
        GLFW_USE_MIR
        BUILD_SHARED_LIBS
        USE_MSVC_RUNTIME_LIBRARY_DLL)

    include_directories("3rdparty/glfw/include")

    # GL3W/GLEW #

    if(MSVC)
        include_directories("3rdparty/imgui/gl3w")
        link_directories("3rdparty/imgui")
        set(GL_SOURCES "3rdparty/imgui/gl3w/GL/gl3w.c")
        set(GL_LIBRARIES $<TARGET_OBJECTS:glfw_objects>)
    else()
        find_package(GLEW REQUIRED)
        include_directories(${GLEW_INCLUDE_DIRS})
        set(GL_LIBRARIES GL ${GLEW_LIBRARIES} $<TARGET_OBJECTS:glfw_objects>)
    endif()

    set(GL_SOURCES
        ${GL_SOURCES}
        3rdparty/imgui/ImGuizmo.cpp
        3rdparty/imgui/imgui.cpp
        3rdparty/imgui/imgui_impl_glfw.cpp
        3rdparty/imgui/imgui_impl_opengl3.cpp
        3rdparty/imgui/imgui_draw.cpp
        3rdparty/imgui/imgui_tables.cpp
        3rdparty/imgui/imgui_widgets.cpp)

    add_definitions(-DXXX_GUI)
endif(XXX_BUILD_WITH_GUI)

# OpenMP #

find_package(OpenMP)
if(OPENMP_FOUND)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# Python #

find_package(Python 3.7 COMPONENTS Interpreter Development)
if(Python_FOUND)
    add_subdirectory("dependencies/pybind11")
endif()

###############################################################################
# Program
###############################################################################

add_definitions(-DXXX_VERSION="${XXX_VERSION}")

include_directories("include")

set(SOURCES
	${GL_SOURCES}
	src/XXX.cpp
)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR})

# get_filename_component(CUDA_COMPILER_BIN "${CMAKE_CUDA_COMPILER}" DIRECTORY)

add_library(XXX STATIC ${SOURCES})
set_target_properties(XXX PROPERTIES CUDA_RESOLVE_DEVICE_SYMBOLS ON)
set_target_properties(XXX PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
target_compile_options(XXX PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:${CUDA_NVCC_FLAGS}>)

add_executable(demo-XXX src/main.cpp)
target_link_libraries(demo-XXX PUBLIC XXX)
target_compile_options(demo-XXX PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:${CUDA_NVCC_FLAGS}>)

if (Python_FOUND)
	add_library(pyXXX SHARED src/python_api.cpp)
	set_target_properties(pyXXX PROPERTIES CXX_VISIBILITY_PRESET "hidden")
	set_target_properties(pyXXX PROPERTIES CUDA_VISIBILITY_PRESET "hidden")
	target_link_libraries(pyXXX PUBLIC XXX ${PYTHON_LIBRARIES} pybind11::module)
	target_compile_options(pyXXX PRIVATE $<$<COMPILE_LANGUAGE:CUDA>:${CUDA_NVCC_FLAGS}>)
	target_compile_definitions(pyXXX PUBLIC -DXXX_PYTHON)
	pybind11_extension(pyXXX)

	if (MSVC)
		# Copy dlls needed at runtime
		file(GLOB CUBLAS_DLLS "${CUDA_COMPILER_BIN}/cublas*.dll")
		if (CUBLAS_DLLS)
			add_custom_command(TARGET pyXXX POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy ${CUBLAS_DLLS} $<TARGET_FILE_DIR:pyXXX>
				COMMAND_EXPAND_LISTS
			)
		endif()
	endif()
endif()


