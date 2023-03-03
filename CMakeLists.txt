cmake_minimum_required(VERSION 3.10...3.22)

project(polyscope)

option(BUILD_TESTING "Build tests" off)

### Policy settings
cmake_policy(SET CMP0054 NEW)   # don't implicitly dereference inside if()

### Add things for this project
add_subdirectory(src)

if(BUILD_TESTING)
    add_subdirectory(test)
endif()

