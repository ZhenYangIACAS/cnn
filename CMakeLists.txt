project(cnn)
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# you may need t do some explicit settings for cygwin and win32
if(WIN64 OR WIN32)
set(BOOST_ROOT C:/local/boost_1_59_0)
set(BOOST_LIBRARYDIR C:/local/boost_1_59_0/lib64-msvc-12.0)
else()
set(BOOST_ROOT /home/kaisheng/tools/boost_1_59_0)
set(BOOST_LIBRARYDIR /usr/local/lib)
set(BOOST_INCLUDEDIR /usr/local/include/boost)
endif()

# best possible performance, the following are recommended:
#   1. use very recent versions of gcc or Clang to build
#   2. use very recent versions of Eigen (ideally the dev version)
#   3. try compiler options like -march=native or other architecture
#      flags (the compiler does not always make the best configuration
#      decisions without help)
if (WIN32 OR WIN64)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -funroll-loops -Wall -std=c++11 -Ofast -g -DEIGEN_FAST_MATH -march=native /bigobj /openmp")
else()
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -funroll-loops -Wall -std=c++11 -Ofast -g -DEIGEN_FAST_MATH -Wno-unused-local-typedefs -march=native -Xlinker -zmuldefs -pthread")
endif()
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")

enable_testing()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}
                    ${PROJECT_SOURCE_DIR}/external/easyloggingpp/src)

# look for Boost
if(DEFINED ENV{BOOST_ROOT})
  set(Boost_NO_SYSTEM_PATHS ON)
endif()

if (CYGWIN OR WIN32 OR WIN64)
set(Boost_REALPATH OFF)
else()
set(Boost_REALPATH ON)
endif()

if (CYGWIN OR UNIX OR LINUX)
find_package(Boost COMPONENTS thread system locale program_options regex serialization REQUIRED)
include_directories(${Boost_INCLUDE_DIR})
set(LIBS ${LIBS} ${Boost_LIBRARIES})
endif()
include_directories(${BOOST_ROOT})
set(LIBS ${LIBS} ${BOOST_LIBRARYDIR})
if (WIN32 OR WIN64)
find_package(Boost COMPONENTS program_options serialization REQUIRED)
endif()

# trouble shooting:
# if boost library cannot be found, in addition to install boost library
# check if environment variables are set
# 
# to set boost root and its library root in environment variable, use
# for example
# echo "export BOOST_LIBRARYDIR=/usr/local/lib" >> ~/.bashrc
# echo "export BOOST_ROOT=/cygdrive/d/tools/boost_1_58_0/boost_1_58_0" >> ~/.bashrc
# then run source ~/.bashrc to have those environment variable effective immediately

if (USE_DOUBLE)
  message("-- use double precsion")
  add_definitions(-DUSE_DOUBLE)
endif()

if(LOOKUP_AT_CPU)
  message("-- LOOKUP_AT_CPU: ${LOOKUP_AT_CPU}")
  set(LOOKUP_AT_CPU "USE_CPU_FOR_LOOKUP_PARAM")
else()
  message("-- LOOKUP_AT_CPU not specified, defaulting to host lookup table at GPU if HAVE_CUDA is defined")
endif()

if(LOOKUP_AT_CPU MATCHES "^USE_CPU_FOR_LOOKUP_PARAM$")
  add_definitions(-DUSE_CPU_FOR_LOOKUP_PARAM)
endif()

if (USE_CURAND)
  add_definitions(-DUSE_CURAND)
endif()

if(SMALL_GPU)
  message("-- SMALL_GPU: ${SMALL_GPU}")
  add_definitions(-DSMALL_GPU)
endif()

if(BACKEND)
  message("-- BACKEND: ${BACKEND}")
else()
  message("-- BACKEND not specified, defaulting to eigen.")
  set(BACKEND "eigen")
endif()

if(BACKEND MATCHES "^eigen$")
  set(WITH_EIGEN_BACKEND 1)
elseif(BACKEND MATCHES "^cuda$")
  set(WITH_CUDA_BACKEND 1)
else()
  message(SEND_ERROR "BACKEND must be eigen or cuda")
endif()


if(WITH_CUDA_BACKEND)
  if(CUDA_ROOT)
  message("CUDA root : ${CUDA_ROOT}")
  else()
  message("Need to define CUDA root by runing -DCUDA_ROOT")
  endif()

  find_package(CUDA REQUIRED)

  set(CUDA_INCLUDE_DIRS ${CUDA_ROOT}/include)
  if (CYGWIN OR UNIX OR LINUX)
     set(CUDA_LIBRARIES ${CUDA_ROOT}/lib64)
  else()
    set(CUDA_LIBRARIES ${CUDA_ROOT}/lib/x64)
  endif()
  set(CUDA_TOOLKIT_ROOT_DIR ${CUDA_ROOT})

  include_directories(SYSTEM ${CUDA_INCLUDE_DIRS})
  add_definitions(-DHAVE_CUDA)
  set(LIBS ${LIBS} ${CUDA_LIBRARIES})
  link_directories(${CUDA_LIBRARIES})

  if (WIN32 OR WIN64)
  set(CUDNN_LIB ${CUDA_LIBRARIES}/cudnn.lib)
  else()
  set(CUDNN_LIB ${CUDA_LIBRARIES}/libcudnn_static.a)
  endif()
endif()

# look for Eigen
find_package(Eigen3 REQUIRED)
include_directories(${EIGEN3_INCLUDE_DIR})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

link_directories(${BOOST_LIBRARYDIR})


add_subdirectory(cnn)
add_subdirectory(examples)

