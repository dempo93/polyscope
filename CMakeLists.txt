cmake_minimum_required(VERSION 3.14)

project(polyscope)

option(BUILD_TESTING "Build tests" off)
option(BUILD_EXAMPLES "Build examples" off)

option(POLYSCOPE_BACKEND_OPENGL3_GLFW "Build opengl glfw backend" on)
option(POLYSCOPE_BACKEND_OPENGL_MOCK "Build opengl mock backend" on)

add_subdirectory(src)

if(BUILD_TESTING)
    add_subdirectory(test)
endif()

if(BUILD_EXAMPLES)
    if(NOT POLYSCOPE_BACKEND_OPENGL3_GLFW)
        message(FATAL_ERROR "BUILD_EXAMPLES needs POLYSCOPE_BACKEND_OPENGL3_GLFW")
    endif()
    add_subdirectory(examples)
endif()