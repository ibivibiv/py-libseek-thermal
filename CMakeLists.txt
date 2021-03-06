cmake_minimum_required(VERSION 3.3)
project("pylibseek_ext")

#----------------------------CMAKE & GLOBAL PROPERTIES-------------------------#
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

###============= C++11 support====================================
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if (COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif (COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else ()
    message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif ()
#=================================================================

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -v -L/opt/local/lib -L/usr/local/lib -lseek -lopencv_core -fpic") 

#=================================================================
# PYTHON option

set(PYTHON_OPTIONS "2.X" "3.X")
set(PYTHON_DESIRED_VERSION "2.X" CACHE STRING "Choose which python version to use, options are: ${PYTHON_OPTIONS}.")
set_property(CACHE PYTHON_DESIRED_VERSION PROPERTY STRINGS ${PYTHON_OPTIONS})

#=============== Find Packages 
find_package(OpenCV COMPONENTS core REQUIRED)
include("DetectPython")
if (${PYTHON_DESIRED_VERSION} STREQUAL "2.X")
    set(Python_ADDITIONAL_VERSIONS ${PYTHON2_VERSION_MAJOR}.${PYTHON2_VERSION_MINOR})
    find_package(Boost COMPONENTS python-py${PYTHON2_VERSION_MAJOR}${PYTHON2_VERSION_MINOR} REQUIRED)
else ()
    set(Python_ADDITIONAL_VERSIONS ${PYTHON3_VERSION_MAJOR}.${PYTHON3_VERSION_MINOR})
    find_package(Boost COMPONENTS python-py${PYTHON3_VERSION_MAJOR}${PYTHON3_VERSION_MINOR} REQUIRED)
endif ()


#========pick python stuff========================================
if (${PYTHON_DESIRED_VERSION} STREQUAL "2.X")
    SET(PYTHON_INCLUDE_DIRS ${PYTHON2_INCLUDE_DIR} ${PYTHON2_INCLUDE_DIR2} ${PYTHON2_NUMPY_INCLUDE_DIRS})
    SET(PYTHON_LIBRARIES ${PYTHON2_LIBRARY})
    SET(PYTHON_EXECUTABLE ${PYTHON2_EXECUTABLE})
    SET(PYTHON_PACKAGES_PATH ${PYTHON2_PACKAGES_PATH})
    SET(ARCHIVE_OUTPUT_NAME pbcvt_py2)
else ()
    SET(PYTHON_INCLUDE_DIRS ${PYTHON3_INCLUDE_DIR} ${PYTHON3_INCLUDE_DIR2} ${PYTHON3_NUMPY_INCLUDE_DIRS})
    SET(PYTHON_LIBRARIES ${PYTHON3_LIBRARY})
    SET(PYTHON_EXECUTABLE ${PYTHON3_EXECUTABLE})
    SET(PYTHON_PACKAGES_PATH ${PYTHON3_PACKAGES_PATH})
    SET(ARCHIVE_OUTPUT_NAME pbcvt_py3)
endif ()

if(NOT WIN32)
    add_definitions(-DBOOST_ALL_DYN_LINK=1)
    add_definitions(-DBOOST_TEST_DYN_LINK)
endif()
add_library(${PROJECT_NAME} SHARED src/SeekThermal.pyboost.cpp)
target_include_directories(${PROJECT_NAME} PUBLIC
        "${CMAKE_CURRENT_SOURCE_DIR}/include"
        ${Boost_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
        ${PYTHON_INCLUDE_DIRS}
	"/usr/local/include/seek"
        )

target_link_libraries(${PROJECT_NAME}
        ${Boost_LIBRARIES}
        ${OpenCV_LIBRARIES}
        ${PYTHON_LIBRARIES}
#	"/usr/local/lib/python2.7/dist-packages/pbcvt.so"
        )

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -v -L/usr/local/lib -lseek") 
set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".so")
