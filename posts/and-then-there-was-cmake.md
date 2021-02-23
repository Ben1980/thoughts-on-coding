---
title: "My God, It’s Full Of Stars: And then there was CMake"
description: 
date: 2019-02-14
tags:
  - c++
  - n-body
  - cmake
layout: layouts/post.njk
image: /img/particleBackground.jpg
---

Welcome back at the 2nd part of the n-body-problem series. With the first post we had a short view on the equations of motion we need to calculate the position in space of a point mass. This time we need to conclude our equations and create a minimum project setup. I would like to use [CMake][1] for project setup for two reasons.

![Hero Image: Particles](/img/particleBackground.jpg)

1. I would like to keep the project environment and IDE independent.
2. I'm new to CMake and want to learn it since a couple of years

Let's start and get back into our project. First of all we are still missing the equation to calculate the change of velocity a mass point is experiencing under acceleration.

$$v=v_{0}+at$$

Now we can start with our project setup. As a source of information how to use CMake I use the CMake documentation and a GitBook. First of all we define a CMake typical folder structure:

``` bash
C:.
|   .gitignore
|   CMakeLists.txt             (1)
|   LICENSE
|   README.md
|   
+---build                      (2)
+---cmake                      (3)
|       FindCatch.cmake
|       
\---solver                     (4)
    |   CMakeLists.txt
    |   
    +---include
    |       solver.h
    |       
    +---src
    |       solver.cpp
    |       
    \---test                   (5)
            solverTest.cpp
            test-main.cpp
```

Both folders, root and solver must contain a `CMakeLists.txt` (1)(4). The files `solver.h/.cpp` are containing only a dummy HelloWorld class to test and demonstrate the project structure and test (5) setup. The cmake folder provides all necessary Find*.cmake files we need to derive for our project all necessary external libraries.

Lets go through the different files in our basic project setup starting with root `CMakeLists.txt`.

```cmake
cmake_minimum_required(VERSION 3.1...3.13)

if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

project(gravity VERSION 0.1.0
    DESCRIPTION "N-Body-Problem project of https://thoughts-on-cpp.com"
    LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/")
include(GNUInstallDirs)
find_package(Catch 2.6.0 REQUIRED)
if(CATCH_FOUND)
    message(STATUS "Building interpreter tests using Catch v${CATCH_VERSION}")
else()
    message(STATUS "Catch not detected. Interpreter tests will be skipped. Install Catch headers"
        " manually or use `cmake -DDOWNLOAD_CATCH=1` to fetch them automatically.")
    return()
endif()

add_library(Catch INTERFACE)
target_include_directories(Catch 
    INTERFACE
    ${CATCH_INCLUDE_DIR})

include(CTest)
add_subdirectory(solver solver)
```

Line 1-5 is defining, according to [modern-cmake][2], the range a CMake installation must fulfill for building the project. If the version of CMake is below version 3.12, [`cmake_policy`][3] is preserving backwards compatibility. Line 10-20 is adding the cmake folder to the [`module path`][4] needed to resolve external dependencies via [find_package][5] and `Find*.cmake` files. In case [Catch2][6] can't be found, it's possible to download the testing library via `-DDOWNLOAD_CATCH=1` parameter defined in `FindCatch.cmake`. In Line 22-25 we add the external Catch2 testing library with [`add_library`][7] and the parameter [`INTERFACE`][8] which ist telling CMake that the library is a pure header interface without any .lib or .a file. With [`target_include_directory`][9] we set where to find the Catch2 header. At the last two lines we are including [`CTest`][10], which we can invoke after building the project to run our tests, and adding the subdirectory of our solver to the project. With [`add_subdirectory`][11] CMake knows where to search for additional `CMakeLists.txt` files.

Now let's have a look at the solver's `CMakeLists.txt`

```cmake
add_library(solver
    SHARED
    src/solver.cpp)

target_include_directories(solver 
    PUBLIC 
    include)

add_executable(solverTest 
    test/test-main.cpp
	test/solverTest.cpp)
target_link_libraries(solverTest Catch
    solver)

enable_testing()
add_test(NAME solverTest COMMAND solverTest)
```

First of all we have to tell CMake how our library is called, which type we want to have it and where it's necessary implementation files are. In line 1-3 we see how this is done via add_library. The library needs also its headers which we include through `target_include_directories` and additional export them and its necessary header via the `PUBLIC` parameter. `PUBLIC` we need to use because we are also building a .lib or .a file which holds the implementation. Here we might need to extend the CMakeLists.txt later when we really exporting the library.

Now we have our minimal project setup which we can extend if it gets necessary. Additional we have a dummy library and a test project which is loading and executing the library. You can get the project at this state via GitHub. At line 8-12 we add the executable generated by Catch2 for running our solver specific tests. With `add_executable` we add the file which contains the main function, and all `.cpp` files which contain our tests. With `target_link_libraries` we link the solver library onto the test executable. The last two lines are enabling CTest and registering the test executable to tell `CTest` what to execute. You can build the project and run its test via the following commands:

```bash
mkdir build //(1) We need a build directory
cd build
cmake ..    //(2) Generating the the artifacts to build, if necesarry download Catch2 via -DDOWNLOAD_CATCH=1 parameter
make        //(3) Build the binaries on linux, for windows you might need different commands, e.g. msbuild gravity.sln for a visual studio based build
ctest       //(4) Running tests
```

## Summary

So that's it for today's post. We successfully setup a very small and basic project from where we can start to implement our n-body-problem project. If necessary I think it will be rather easy to extend it. Maybe we add later a Qt based UI to show us our calculation results. The next post we start to implement the solver and a couple of tests which we need to validate our solver is working and evolving the right way.

[1]: https://cmake.org
[2]: https://cliutils.gitlab.io/modern-cmake/chapters/basics.html
[3]: https://cmake.org/cmake/help/v3.13/manual/cmake-policies.7.html
[4]: https://cmake.org/cmake/help/v3.13/variable/CMAKE_MODULE_PATH.html?highlight=cmake_module_path
[5]: https://cmake.org/cmake/help/v3.13/command/find_package.html#command:find_package
[6]: https://github.com/catchorg/Catch2
[7]: https://cmake.org/cmake/help/v3.13/command/add_library.html
[8]: https://cmake.org/cmake/help/v3.13/command/add_library.html#interface-libraries
[9]: https://cmake.org/cmake/help/v3.13/command/target_include_directories.html?highlight=target_include_directory
[10]: https://cmake.org/cmake/help/v3.13/manual/ctest.1.html
[11]: https://cmake.org/cmake/help/v3.13/command/add_subdirectory.html?highlight=add_subdirectory
