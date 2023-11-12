cmake_minimum_required(VERSION 3.28)

project(Glitter VERSION 1.0
    DESCRIPTION "Examples to learn OpenGL"
    LANGUAGES CXX)

find_package(Git QUIET)

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    message(STATUS "Submodule update")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE GIT_SUBMOD_RESULT)

    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()

option(GLFW_BUILD_DOCS OFF)
option(GLFW_BUILD_EXAMPLES OFF)
option(GLFW_BUILD_TESTS OFF)
add_subdirectory(third_party/glfw)

option(ASSIMP_BUILD_ASSIMP_TOOLS OFF)
option(ASSIMP_BUILD_SAMPLES OFF)
option(ASSIMP_BUILD_TESTS OFF)
add_subdirectory(third_party/assimp)

option(BUILD_BULLET2_DEMOS OFF)
option(BUILD_CPU_DEMOS OFF)
option(BUILD_EXTRAS OFF)
option(BUILD_OPENGL3_DEMOS OFF)
option(BUILD_UNIT_TESTS OFF)
add_subdirectory(third_party/bullet3)

# set(CMAKE_CXX_STANDARD 11 CACHE STRING "The C++ standard to use")
# set(CMAKE_CXX_STANDARD_REQUIRED ON)
# set(CMAKE_CXX_EXTENSIONS OFF)
if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -std=c++11")

    if(NOT WIN32)
        set(GLAD_LIBRARIES dl)
    endif()
endif()

include_directories(include/Glitter/
    third_party/assimp/include/
    third_party/bullet/src/
    third_party/glad/include/
    third_party/glfw/include/
    third_party/glm/
    third_party/stb/)

file(GLOB VENDORS_SOURCES third_party/glad/src/glad.c)
file(GLOB PROJECT_HEADERS include/Glitter/*.h)
file(GLOB PROJECT_SOURCES src/Sources/*.cpp)
file(GLOB PROJECT_SHADERS src/Shaders/*.comp
    src/Shaders/*.frag
    src/Shaders/*.geom
    src/Shaders/*.vert)
file(GLOB PROJECT_CONFIGS CMakeLists.txt
    Readme.md
    .gitattributes
    .gitignore
    .gitmodules)

source_group("Headers" FILES ${PROJECT_HEADERS})
source_group("Shaders" FILES ${PROJECT_SHADERS})
source_group("Sources" FILES ${PROJECT_SOURCES})
source_group("Vendors" FILES ${VENDORS_SOURCES})

add_definitions(-DGLFW_INCLUDE_NONE
    -DPROJECT_SOURCE_DIR=\"${PROJECT_SOURCE_DIR}\")
add_executable(demo ${PROJECT_SOURCES} ${PROJECT_HEADERS}
    ${PROJECT_SHADERS} ${PROJECT_CONFIGS}
    ${VENDORS_SOURCES})
target_link_libraries(demo assimp glfw
    ${GLFW_LIBRARIES} ${GLAD_LIBRARIES}
    BulletDynamics BulletCollision LinearMath)
target_include_directories(demo PUBLIC include)
set_target_properties(demo PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${PROJECT_NAME})

add_custom_command(
    TARGET demo POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/src/Shaders $<TARGET_FILE_DIR:demo>
    DEPENDS ${PROJECT_SHADERS})
