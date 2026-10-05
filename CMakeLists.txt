cmake_minimum_required(VERSION 3.30)

# Set vcpkg toolchain BEFORE project() initialization
if(NOT CMAKE_TOOLCHAIN_FILE)
    set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_SOURCE_DIR}/vcpkg/scripts/buildsystems/vcpkg.cmake" 
        CACHE STRING "Vcpkg toolchain file")
endif()

project(LiteEngine VERSION 0.1 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Find vcpkg packages
find_package(raylib REQUIRED)
find_package(imgui REQUIRED)

# Collect source files dynamically
file(GLOB_RECURSE lite_sources 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp"
)

# lite_sources already contains main.cpp
add_executable(LiteEngine ${lite_sources})

target_include_directories(LiteEngine PRIVATE 
    "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

target_link_libraries(LiteEngine PRIVATE 
    raylib
    imgui::imgui
    rlImGui
)

if(MSVC)
    target_compile_options(LiteEngine PRIVATE
        /W3
        /MP
        /WX
        /Zi
    )
else()
    target_compile_options(LiteEngine PRIVATE
        -Wall
        -Wextra
        -Wpedantic
    )
endif()

add_library(rlImGui STATIC
    "${CMAKE_CURRENT_SOURCE_DIR}/vendor/rlImGui/rlImGui.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/vendor/rlImGui/rlImGui.h"
)

target_include_directories(rlImGui PUBLIC 
    "${CMAKE_CURRENT_SOURCE_DIR}/vendor/rlImGui"
)

target_link_libraries(rlImGui PUBLIC 
    raylib
    imgui::imgui
)

# Now link your main executable to it as usual
target_link_libraries(LiteEngine PRIVATE 
    raylib
    imgui::imgui
    rlImGui
)

# IDE solution folder organization
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/src" PREFIX "Source Files" FILES ${lite_sources})