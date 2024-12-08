cmake_minimum_required(VERSION 2.6)

project(CudaSift)
set(cudaSift_VERSION_MAJOR 2)
set(cudaSift_VERSION_MINOR 0)
set(cudaSift_VERSION_PATCH 0)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall -g -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -g  -O3")

set(CPACK_PACKAGE_VERSION_MAJOR "${cudaSift_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${cudaSift_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${cudaSift_VERSION_PATCH}")
set(CPACK_GENERATOR "ZIP")
include(CPack)

find_package(OpenCV REQUIRED)
find_package(CUDA)
if (NOT CUDA_FOUND)
  message(STATUS "CUDA not found. Project will not be built.")
endif(NOT CUDA_FOUND)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 ")

include(FindCUDA)
cuda_select_nvcc_arch_flags(ARCH_FLAGS)
message(STATUS "Setting CUDA arch to ${ARCH_FLAGS}.")

set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} ${ARCH_FLAGS}")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-O3")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-std=c++17")
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-lineinfo") # Enable line info for debugging
set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS} "-O2")

set(cuda_sources
  cudaImage.cu  
  cudaImage.h  
  cudaSiftH.cu 
  cudaSiftH.h  
  matching.cu  
  cudaSiftD.h  
  cudaSift.h  
  cudautils.h
)  

set(sources
  geomFuncs.cpp  
  mainSift.cpp  
)

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

include_directories(
  ${OpenCV_INCLUDE_DIRS}
  ${CMAKE_CURRENT_SOURCE_DIR} 
)

cuda_add_library(cudasift SHARED ${cuda_sources})
set_target_properties(cudasift PROPERTIES
  COMPILE_FLAGS "${EXTRA_CXX_FLAGS}"         
)
target_link_libraries(cudasift ${CUDA_cudadevrt_LIBRARY})
