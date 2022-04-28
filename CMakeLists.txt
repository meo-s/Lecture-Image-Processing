cmake_minimum_required(VERSION 3.23)

if (POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW)
endif (POLICY CMP0048)

set(OPENCV_INCLUDE_PATH "./libraries/opencv/include"
    CACHE STRING "A path to OpenCV include folder.")
set(OPENCV_LIBS_PATH "./libraries/opencv/libs"
    CACHE STRING "A path to OpenCV libs folder.")
set(OPENCV_BINS_PATH "./libraries/opencv/bins"
    CACHE STRING "A path to OpenCV bins folder.")
    
project(LectureImageProcessing VERSION 1.0.0)

if (NOT DEFINED CMAKE_BUILD_TYPE OR ${CMAKE_BUILD_TYPE} STREQUAL "")
    message(STATUS "CMAKE_BUILD_TYPE is not specified. Default value \"Debug\" will be used.")
    set(CMAKE_BUILD_TYPE "DEBUG")
endif ()
    
file(GLOB SRC_FILES "*.cc" "*.hh" "**/*.cc" "**/*.hh" "**/*.hpp")
add_executable(${PROJECT_NAME} ${SRC_FILES})
target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_17)

# Link OpenCV
target_include_directories(${PROJECT_NAME} PRIVATE "${OPENCV_INCLUDE_PATH}")
file(GLOB OPENCV_LIBS "${OPENCV_LIBS_PATH}/*.lib" "${OPENCV_LIBS_PATH}/*.a")
file(GLOB OPENCV_BINS "${OPENCV_BINS_PATH}/*.dll" "${OPENCV_BINS_PATH}/*.so")
if (CMAKE_BUILD_TYPE STREQUAL "RELEASE")
    list(FILTER OPENCV_LIBS INCLUDE REGEX ".*/opencv_.*[^d]\\.(lib|a)$")
    list(FILTER OPENCV_BINS INCLUDE REGEX ".*/opencv_.*[^d]\\.(dll|so)$")
endif ()
if (CMAKE_BUILD_TYPE STREQUAL "DEBUG")
    list(FILTER OPENCV_LIBS INCLUDE REGEX ".*/opencv_.*d\\.(lib|a)$")
    list(FILTER OPENCV_BINS INCLUDE REGEX ".*/opencv_.*d\\.(dll|so)$")
endif ()

target_link_libraries(${PROJECT_NAME} PRIVATE ${OPENCV_LIBS})
foreach (DYNAMIC_LIBRARY IN LISTS OPENCV_BINS)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different ${DYNAMIC_LIBRARY} $<TARGET_FILE_DIR:${PROJECT_NAME}>)
endforeach()
