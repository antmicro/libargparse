cmake_minimum_required(VERSION 2.8.12)

project("libargparse")

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
    #Only set compiler settings if we are not a sub-project
    set(WARN_FLAGS "-Wall -Wextra -Wpedantic -Wcast-qual -Wcast-align -Wshadow -Wformat=2 -Wlogical-op -Wmissing-declarations -Wmissing-include-dirs -Wredundant-decls -Wswitch-default -Wundef -Wunused-variable -Wdisabled-optimization -Wnoexcept -Woverloaded-virtual -Wctor-dtor-privacy -Wnon-virtual-dtor")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++14 ${WARN_FLAGS}") 
    #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fsanitize=leak -fsanitize=undefined") 
    set(FLEX_BISON_WARN_SUPPRESS_FLAGS "-Wno-switch-default -Wno-unused-parameter -Wno-missing-declarations")
endif()

set(LIB_INCLUDE_DIRS src)
file(GLOB_RECURSE LIB_SOURCES src/*.cpp)
file(GLOB_RECURSE LIB_HEADERS src/*.hpp)

#Create the library
add_library(libargparse STATIC
             ${LIB_HEADERS}
             ${LIB_SOURCES})
set_target_properties(libargparse PROPERTIES PREFIX "") #Avoid extra 'lib' prefix
target_include_directories(libargparse PUBLIC ${LIB_INCLUDE_DIRS})

if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
    #Create the test executable
    add_executable(argparse_test argparse_test.cpp)
    target_link_libraries(argparse_test libargparse)

    #Create the example executable
    add_executable(argparse_example argparse_example.cpp)
    target_link_libraries(argparse_example libargparse)
endif()

#
# ----------------------- Code format --------------------------
#

set(code_format_files
    -name '*.cpp' -print -o -name '*.h' -print
    -o -name '*.tpp' -print -o -name '*.hpp' -print)

set(code_format_dirs ${PROJECT_SOURCE_DIR}/src)

#
# Use clang-format-5.0 for code format
#
add_custom_target(format-cpp
    COMMAND find ${code_format_dirs} ${code_format_files} |
        xargs -P `nproc` clang-format-5.0 -style=file -i)

#
# Use simple python script for fixing C like boxed comments
#
add_custom_target(format-cpp-fix-comments DEPENDS format-cpp
    COMMAND find ${code_format_dirs} ${code_format_files} |
        xargs -L 1 -P `nproc`
        python3 ${PROJECT_SOURCE_DIR}/scripts/format.py --inplace --fix-comments --input)

#
# Use simple python script for fixing template brackets e.g. <<>
#
add_custom_target(format-cpp-fix-template-operators DEPENDS format-cpp
    COMMAND find ${code_format_dirs} ${code_format_files} |
        xargs -L 1 -P `nproc`
        python3 ${PROJECT_SOURCE_DIR}/scripts/format.py --inplace --fix-template-operators --input)

add_custom_target(format DEPENDS format-cpp-fix-comments format-cpp-fix-template-operators)
