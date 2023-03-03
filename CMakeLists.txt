cmake_minimum_required(VERSION 3.5...3.22)

project(polyscope)

### Policy settings
cmake_policy(SET CMP0054 NEW)   # don't implicitly dereference inside if()

### Add things for this project
add_subdirectory(src)
