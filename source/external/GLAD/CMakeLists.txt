cmake_minimum_required(VERSION 3.10.0)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

add_library(GLAD "src/glad.c")

target_include_directories(GLAD PUBLIC
                          "${PROJECT_SOURCE_DIR}/source/external/GLAD/include"
                          )

project(UnknownEngine)
