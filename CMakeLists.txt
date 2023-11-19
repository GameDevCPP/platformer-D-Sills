cmake_minimum_required(VERSION 3.21)
project(CMakeSFMLProject LANGUAGES CXX)

#### setup Directories ####
#Main output directory
set(OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/")
# Ouput all DLLs from all libs into main build folder
set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_DIRECTORY})

add_custom_target(copy_resources ALL COMMAND ${CMAKE_COMMAND}
        -E copy_directory
        "${PROJECT_SOURCE_DIR}/res"
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$<CONFIGURATION>/res
)
add_custom_target(copy_resources_rootdir ALL COMMAND ${CMAKE_COMMAND}
        -E copy_directory
        "${PROJECT_SOURCE_DIR}/res"
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/res
)

#### Add External Dependencies ####
# SFML

include(FetchContent)
FetchContent_Declare(SFML
        GIT_REPOSITORY https://github.com/SFML/SFML.git
        GIT_TAG 2.6.x)
FetchContent_MakeAvailable(SFML)

# B2D - Box2D physics library
add_subdirectory("lib/b2d/Box2D")

# Let's simplify the include and link process for Box2D
target_include_directories(Box2D INTERFACE "lib/b2d/Box2D/")

#### Add Engine and our own library projects####

file(GLOB_RECURSE RESOURCE_FILES "res/*.*")
####
file(GLOB_RECURSE SOURCE_FILES engine/*.cpp engine/*.h)
add_library(lib_engine STATIC ${SOURCE_FILES} ${RESOURCE_FILES})
target_include_directories(lib_engine INTERFACE "${CMAKE_SOURCE_DIR}/engine/" PUBLIC SYSTEM ${B2D_INCS} )
target_link_libraries(lib_engine lib_tile_level_loader lib_maths lib_ecm Box2D sfml-graphics sfml-audio)

## Maths lib
add_library(lib_maths INTERFACE)
target_sources(lib_maths INTERFACE "${CMAKE_SOURCE_DIR}/lib_maths/maths.h")


# Include the directories of lib_maths and SFML
target_include_directories(lib_maths INTERFACE
        "${CMAKE_SOURCE_DIR}/lib_maths"
        ${SFML_SOURCE_DIR}/include
)

## Tile loader lib
file(GLOB_RECURSE SOURCE_FILES lib_tile_level_loader/*.cpp lib_tile_level_loader/*.h)
add_library(lib_tile_level_loader STATIC ${SOURCE_FILES} ${RESOURCE_FILES})
target_include_directories(lib_tile_level_loader INTERFACE "${CMAKE_SOURCE_DIR}/lib_tile_level_loader/" )
target_link_libraries(lib_tile_level_loader lib_maths sfml-graphics)

## ECM lib
file(GLOB_RECURSE SOURCE_FILES lib_ecm/*.cpp lib_ecm/*.h)
add_library(lib_ecm STATIC ${SOURCE_FILES})
target_include_directories(lib_ecm INTERFACE "${CMAKE_SOURCE_DIR}/lib_ecm" )
target_link_libraries(lib_ecm PRIVATE lib_maths)



file(GLOB_RECURSE SOURCES lab_7_platformer/*.cpp lab_7_platformer/*.h)
file(GLOB_RECURSE CMPNTS lab_7_platformer/components/*.cpp lab_7_platformer/components/*.h)
file(GLOB_RECURSE SCENES lab_7_platformer/scenes/*.cpp lab_7_platformer/scenes/*.h)
add_executable(CMakeSFMLProject ${SOURCES} ${RESOURCE_FILES})

source_group("components" FILES ${CMPNTS})
source_group("resources" FILES ${RESOURCE_FILES})

target_link_libraries(CMakeSFMLProject PRIVATE lib_engine  sfml-graphics Box2D)
set(EXECUTABLES ${EXECUTABLES} CMakeSFMLProject)

add_dependencies(CMakeSFMLProject copy_resources copy_resources_rootdir )


set_target_properties(CMakeSFMLProject
        PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$(Configuration)
)
