project(BTree)
# set(CMAKE_VERBOSE_MAKEFILE 1)
if(CMAKE_OLD)
cmake_minimum_required(VERSION 2.8)
else(CMAKE_OLD)
cmake_minimum_required(VERSION 3.1)
endif(CMAKE_OLD)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) ## on g++ this ensures: -std=c++11 and not -std=gnu++11

if(CPP_STATIC)
set(CMAKE_CXX_FLAGS "-m64 -Wall -Wextra -static")
else(CPP_STATIC)
set(CMAKE_CXX_FLAGS "-m64 -Wall -Wextra")
endif(CPP_STATIC)


#### Build setting
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  #### Set a default build type if none was specified
  if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  endif()
  message("Building in ${CMAKE_BUILD_TYPE} mode")

  set(CMAKE_CXX_FLAGS_DEBUG "-O0 -ggdb -g -p -Wconversion")
  set(CMAKE_CXX_FLAGS_RELEASE "-ggdb -Ofast -fstrict-aliasing -DNDEBUG -march=native")
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-g -ggdb -Ofast -fstrict-aliasing -march=native")
endif()



include_directories(${CMAKE_CURRENT_SOURCE_DIR})
add_library(${PROJECT_NAME} STATIC
  TagRelabelAlgo.cpp
)



#### Modules
if(NOT MYMOD_SOURCE_DIR)
  set(MYMOD_SOURCE_DIR ${CMAKE_SOURCE_DIR}/module)
  set(MYMOD_BINARY_DIR ${CMAKE_BINARY_DIR}/module)
  add_subdirectory(${MYMOD_SOURCE_DIR}/Basics ${MYMOD_BINARY_DIR}/Basics)
endif()
include_directories(${MYMOD_SOURCE_DIR}/Basics)



#### TEST
#### To enable this, add "-DTESTING=1" or "-DTESTING_ALL=1" (for testing all subprojects) when running cmake
if(TESTING OR TESTING_ALL)
  if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
    #### Download and unpack googletest at configure time
    configure_file(gtest.cmake.in gtest/CMakeLists.txt)
    execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
                    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/gtest")
    execute_process(COMMAND "${CMAKE_COMMAND}" --build .
                    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/gtest")

    #### Prevent GoogleTest from overriding our compiler/linker options
    #### when building with Visual Studio
    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

    #### Add googletest directly to our build. This adds the following targets:
    #### gtest, gtest_main, gmock and gmock_main
    add_subdirectory("${CMAKE_BINARY_DIR}/gtest/googletest-src"
                     "${CMAKE_BINARY_DIR}/gtest/googletest-build")

    #### The gtest/gmock targets carry header search path
    #### dependencies automatically when using CMake 2.8.11 or
    #### later. Otherwise we have to add them here ourselves.
    if(CMAKE_VERSION VERSION_LESS 2.8.11)
      include_directories("${gtest_SOURCE_DIR}/include"
                          "${gmock_SOURCE_DIR}/include")
    endif()
  endif()


  if((CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR) OR TESTING_ALL)
    message("Testing ${PROJECT_NAME} is enabled")

    add_subdirectory(test)
    enable_testing()

    add_test(NAME PSumWithValue_Test COMMAND PSumWithValue_Test)
    # add_test(NAME PSum_Test COMMAND PSum_Test)
  endif()
endif()



#### Doxygen
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  # list(APPEND DOXYGEN_EXCLUDE "${CMAKE_SOURCE_DIR}/build")
  find_package(Doxygen)
  if(DOXYGEN_FOUND)
    set(DOXYGEN_CONF_FILE "doxygen.conf")
    set(DOXYGEN_CONF_PATH ${CMAKE_CURRENT_BINARY_DIR}/${DOXYGEN_CONF_FILE})
    set(DOXYGEN_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
    set(DOXYGEN_TARGET "doc")

    configure_file(${DOXYGEN_CONF_FILE}.in ${DOXYGEN_CONF_PATH})

    add_custom_target(${DOXYGEN_TARGET}
                      ${DOXYGEN_EXECUTABLE} ${DOXYGEN_CONF_PATH}
                      DEPENDS ${DOXYGEN_CONF_PATH})
    message("\"$ make doc\" to run Doxygen")
  else(DOXYGEN_FOUND)
    message(WARNING "doxygen binary couldn't be found")
  endif(DOXYGEN_FOUND)
endif(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)


install (TARGETS BTree DESTINATION lib)