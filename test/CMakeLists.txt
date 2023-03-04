# Set up testing
enable_testing()

# Configure the compiler

# NOTE: Polyscope itself uses C++11, but the tests use C++14 because googletest requires it.

if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")

    # using Clang (linux or apple) or GCC
    message("Using clang/gcc compiler flags")
    set(BASE_CXX_FLAGS "-std=c++14 -Wall -Wextra -g3") # use C++14 for tests only
    set(DISABLED_WARNINGS
        " -Wno-unused-parameter -Wno-unused-variable -Wno-unused-function -Wno-deprecated-declarations -Wno-missing-braces -Wno-unused-private-field"
    )
    set(TRACE_INCLUDES " -H -Wno-error=unused-command-line-argument")

    if("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
        message("Setting clang-specific options")
        set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} -ferror-limit=3 -fcolor-diagnostics")
        set(CMAKE_CXX_FLAGS_DEBUG "-fsanitize=address -fno-limit-debug-info")
    elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        message("Setting gcc-specific options")
        set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} -fmax-errors=5")
        set(DISABLED_WARNINGS
            "${DISABLED_WARNINGS} -Wno-maybe-uninitialized -Wno-format-zero-length -Wno-unused-but-set-parameter -Wno-unused-but-set-variable"
        )
    endif()

    set(CMAKE_CXX_FLAGS "${BASE_CXX_FLAGS} ${DISABLED_WARNINGS}")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g3")
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag(-march=native COMPILER_SUPPORTS_MARCH_NATIVE)
    if(COMPILER_SUPPORTS_MARCH_NATIVE)
        set(MARCH_NATIVE "-march=native")
    else()
        set(MARCH_NATIVE "")
    endif()
    set(CMAKE_CXX_FLAGS_RELEASE "${MARCH_NATIVE} -O3 -DNDEBUG")

elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    # using Visual Studio C++
    message("Using Visual Studio compiler flags")
    set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} /W4")
    set(BASE_CXX_FLAGS "${BASE_CXX_FLAGS} /MP") # parallel build
    set(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4267\""
    )# ignore conversion to smaller type (fires more aggressively than the gcc version, which is annoying)
    set(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4244\""
    )# ignore conversion to smaller type (fires more aggressively than the gcc version, which is annoying)
    set(DISABLED_WARNINGS "${DISABLED_WARNINGS} /wd\"4305\"") # ignore truncation on initialization
    set(CMAKE_CXX_FLAGS "${BASE_CXX_FLAGS} ${DISABLED_WARNINGS}")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MD")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MDd")

    add_definitions(/D "_CRT_SECURE_NO_WARNINGS")
    add_definitions(-DNOMINMAX)
else()
    # unrecognized
    message(FATAL_ERROR "Unrecognized compiler [${CMAKE_CXX_COMPILER_ID}]")
endif()

# Build the tests
set(TEST_SRCS include/polyscope_test.h src/main_test.cpp src/array_adaptors_test.cpp src/basics_test.cpp)

find_package(GTest CONFIG REQUIRED)
add_executable(polyscope-test "${TEST_SRCS}")
target_include_directories(polyscope-test PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_link_libraries(polyscope-test PRIVATE GTest::gtest_main polyscope)
