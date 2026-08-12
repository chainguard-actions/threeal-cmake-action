cmake_minimum_required(VERSION 3.10)
project(hello_c C)

add_executable(hello_c main.c)

enable_testing()
add_test(NAME hello_c_test COMMAND hello_c)
