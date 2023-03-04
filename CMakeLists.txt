cmake_minimum_required(VERSION 3.10...3.22)

project(polyscope)

option(BUILD_TESTING "Build tests" off)
option(BUILD_EXAMPLES "Build examples" off)

option(POLYSCOPE_BACKEND_OPENGL3_GLFW "Build opengl glfw backend" on)
option(POLYSCOPE_BACKEND_OPENGL_MOCK "Build opengl mock backend" on)

### Policy settings
cmake_policy(SET CMP0054 NEW)   # don't implicitly dereference inside if()

### Add things for this project
add_subdirectory(src)

if(BUILD_TESTING)
    add_subdirectory(test)
endif()

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()