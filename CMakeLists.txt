cmake_minimum_required(VERSION 3.8.0)

project(gst LANGUAGES CXX CUDA C)

find_package(CUDAToolkit REQUIRED)

# Set a default build type if none was specified
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build: Release/Debug/RelWithDebInfo" FORCE)

# # Global CXX flags/options
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(DEBUG_MATRIX_SIZES MATCHES ON)
message("DEBUG_MATRIX_SIZES ")
    set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -DDEBUG_MATRIX_SIZES")
endif()


include_directories( ${PROJECT_SOURCE_DIR}/util ${CUDAToolkit_INCLUDE_DIRS} )

add_executable(gst main.cu
    ${PROJECT_SOURCE_DIR}/util/test_args.cu
    ${PROJECT_SOURCE_DIR}/util/measure.cc
    ${PROJECT_SOURCE_DIR}/util/device_info.cu
    ${PROJECT_SOURCE_DIR}/util/test_util.cu
)

set_target_properties(gst PROPERTIES CUDA_ARCHITECTURES "70;75;80;90")

target_compile_options(gst
    PRIVATE
    $<$<OR:$<CXX_COMPILER_ID:Clang>,$<CXX_COMPILER_ID:AppleClang>,$<CXX_COMPILER_ID:GNU>>: -Wall -Wextra -fno-strict-aliasing -Wno-unused-parameter -m64>
    $<$<CXX_COMPILER_ID:Clang>:-Wl,--no-undefined -Wl,--no-allow-shlib-undefined>
    $<$<CXX_COMPILER_ID:MSVC>:/Wall -m64>
    )

target_link_directories(gst PRIVATE ${CUDAToolkit_LIBRARY_DIR} )
target_link_libraries(gst CUDA::cublas CUDA::cublasLt)