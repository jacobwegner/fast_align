project(fast_align)
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Add Apple Silicon and LLVM support
if(APPLE)
    # Set paths for LLVM from Homebrew
    set(CMAKE_C_COMPILER "/opt/homebrew/opt/llvm/bin/clang")
    set(CMAKE_CXX_COMPILER "/opt/homebrew/opt/llvm/bin/clang++")

    set(CMAKE_OSX_ARCHITECTURES "arm64")

    # OpenMP settings for Apple
    set(OpenMP_CXX_FLAGS "-Xpreprocessor -fopenmp")
    set(OpenMP_C_FLAGS "-Xpreprocessor -fopenmp")
    set(OpenMP_CXX_LIB_NAMES "omp")
    set(OpenMP_C_LIB_NAMES "omp")
    set(OpenMP_omp_LIBRARY "/opt/homebrew/opt/libomp/lib/libomp.dylib")

    # Include directories for OpenMP
    include_directories("/opt/homebrew/opt/libomp/include")
    link_directories("/opt/homebrew/opt/libomp/lib")

    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11 -O3 -g ${OpenMP_CXX_FLAGS}")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -g ${OpenMP_C_FLAGS}")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=c++11 -O3 -g")
    find_package(OpenMP QUIET)
    if(OPENMP_FOUND)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    endif(OPENMP_FOUND)
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

find_package(SparseHash)
if(SPARSEHASH_FOUND)
    add_definitions(-DHAVE_SPARSEHASH)
endif(SPARSEHASH_FOUND)

# Create a library for shared code
add_library(fastalign_lib STATIC
    src/ttables.cc
)

if(APPLE)
    target_link_libraries(fastalign_lib PRIVATE omp)
endif()

# Link executables with the library
add_executable(fast_align src/fast_align.cc)
target_link_libraries(fast_align fastalign_lib)
if(APPLE)
    target_link_libraries(fast_align PRIVATE omp)
endif()

add_executable(atools src/alignment_io.cc src/atools.cc)
if(APPLE)
    target_link_libraries(atools PRIVATE omp)
endif()

configure_file(src/force_align.py force_align.py COPYONLY)
