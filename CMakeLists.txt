cmake_minimum_required(VERSION 2.8.12)

project("clinfo" C CXX)

include_directories(${CMAKE_SOURCE_DIR}/src ${OPENCL_SDK_ROOT}/include)
link_directories(${OPENCL_SDK_ROOT}/lib)

file(GLOB SRC_C_FILES ./src/*.c)
list(APPEND SRC_FILES ${SRC_C_FILES})
file(GLOB SRC_CC_FILES ./src/*.cc)
list(APPEND SRC_FILES ${SRC_CC_FILES})
file(GLOB SRC_CPP_FILES ./src/*.cpp)
list(APPEND SRC_FILES ${SRC_CPP_FILES})

add_executable(clinfo ${SRC_FILES})
target_link_libraries(clinfo OpenCL)
