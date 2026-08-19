cmake_minimum_required(VERSION 3.10)
project(OptionsProject)

option(MY_OPTION "A custom option" OFF)

if(MY_OPTION)
  message(STATUS "MY_OPTION is ON")
else()
  message(STATUS "MY_OPTION is OFF")
endif()

add_executable(hello hello.c)
