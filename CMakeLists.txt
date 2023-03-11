cmake_minimum_required(VERSION 3.0.0)
project(demo VERSION 0.1.0)
set(CMAKE_CXX_COMPILER "g++")

include_directories(${PROJECT_SOURCE_DIR}/include) #头文件
add_executable(demo e_src/helloworld_main.cpp src/helloworld_test.cpp) #可执行文件[main函数文件+源文件]


include(CTest)
enable_testing()

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
