# cmake specification for FREEDM Broker
cmake_minimum_required(VERSION 2.6)
project(Broker)

set(ALT_BUILD_VERSION "Unversioned")
# Build Versioning
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMakeModules/")
include(GetGitRevisionDescription)
get_git_head_revision(GIT_REFSPEC GIT_SHA1)

# configurable project options
# TUTORIAL on how to add a new option:
#   add the option below with a help message defaulted to OFF
#   add the option to Broker/include/config.hpp.cmake as #cmakedefine OPTION
#   activate / deactivate the option as described in the message below
option(BUILD_TESTS "Enable the building and running of unit tests." OFF)
option(CUSTOMNETWORK "for network.xml support" OFF)
option(DATAGRAM "for UDP Datagram service w/o sequencing" OFF)
option(SHOW_MESSAGES "Enable help messages during cmake execution" ON)
option(SHOW_WARNINGS "warnings displayed during project compile" ON)
option(USE_CLANG "Build FREEDM using Clang instead of GCC" OFF)
option(USE_DEVICE_PSCAD "Enable the PSCAD simulation interface" OFF)
option(USE_DEVICE_RTDS "Enable the RTDS simulation interface" OFF)

if(SHOW_MESSAGES)
    message("This project uses custom CMake settings:"
        "\n\tView the available settings with cmake -LH"
        "\n\tChange a setting with -DSETTING=ON/OFF")
endif(SHOW_MESSAGES)

# find the required boost libraries
find_package(Boost REQUIRED COMPONENTS system thread program_options serialization date_time)

if(Boost_FOUND)
    # add the boost libraries to the project
    include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
    link_directories(${Boost_LIBRARY_DIRS})


    # set C++ compiler flags
    if(SHOW_WARNINGS)
        set(WARNING_FLAGS "-Wall -Wextra -pedantic")
    endif(SHOW_WARNINGS)

    set(CMAKE_CXX_FLAGS "-std=c++98 -g ${WARNING_FLAGS}")

    if(USE_CLANG)
        set(CMAKE_CXX_COMPILER "/usr/bin/clang++")
        set(CMAKE_CXX_FLAGS "-g ${WARNING_FLAGS}")
    endif(USE_CLANG)

    set(CMAKE_EXE_LINKER_FLAGS "-lpthread")

    # generate the configuration file
    CONFIGURE_FILE(
        ${CMAKE_BINARY_DIR}/include/config.hpp.cmake
        ${CMAKE_BINARY_DIR}/include/config.hpp )
    
    # include project source directories
    include_directories("${CMAKE_BINARY_DIR}/include")
    include_directories("${CMAKE_BINARY_DIR}/src")

    # goto src/CMakeLists.txt
    add_subdirectory( src )

    add_executable(PosixBroker src/PosixMain.cpp)

    # link the executable to its dependencies
    target_link_libraries(
        PosixBroker
        broker
        ${Boost_THREAD_LIBRARY}
        ${Boost_SYSTEM_LIBRARY}
        ${Boost_SERIALIZATION_LIBRARY}
        ${Boost_PROGRAM_OPTIONS_LIBRARY}
        ${Boost_DATE_TIME_LIBRARY}
    )

    if(BUILD_TESTS)
        enable_testing()

        # goto test/CMakeLists.txt
        add_subdirectory(test)
    endif(BUILD_TESTS)
endif(Boost_FOUND)
