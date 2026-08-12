cmake_minimum_required(VERSION 3.10)
project(hello_world CXX)

add_executable(hello_world main.cpp)

enable_testing()
add_test(NAME hello_world_test COMMAND hello_world)
