cmake_minimum_required(VERSION 2.6)

project(CudaSift)
set(cudaSift_VERSION_MAJOR 2)
set(cudaSift_VERSION_MINOR 0)
set(cudaSift_VERSION_PATCH 0)

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

if (UNIX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -msse2 ")
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "aarch64")
    message(STATUS "Compiling on aarch64 architecture.")
    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_35;-lineinfo;-ccbin;/usr/bin/gcc;--compiler-options;-O2;-D_FORCE_INLINES;-DVERBOSE_NOT") 
  else()
    message(STATUS "Compiling on a non-aarch64 architecture.")
    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_86;-lineinfo;-ccbin;/usr/bin/gcc;--compiler-options;-O2;-D_FORCE_INLINES;-DVERBOSE_NOT") 
  endif()
endif()

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

#SET(CUDA_SEPARABLE_COMPILATION ON)

cuda_add_executable(cudasiftdemo ${cuda_sources} ${sources})
set_target_properties(cudasiftdemo PROPERTIES
  COMPILE_FLAGS "${EXTRA_CXX_FLAGS}"			   
)
target_link_libraries(cudasiftdemo ${CUDA_cudadevrt_LIBRARY} ${OpenCV_LIBS})

cuda_add_library(cudasift SHARED ${cuda_sources})
set_target_properties(cudasift PROPERTIES
  COMPILE_FLAGS "${EXTRA_CXX_FLAGS}"         
)
target_link_libraries(cudasift ${CUDA_cudadevrt_LIBRARY})
