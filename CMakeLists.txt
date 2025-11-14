cmake_minimum_required(VERSION 3.28)
project(vulkan_tutorial LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)

find_program(GLSLC_EXECUTABLE NAMES glslc
    HINTS
        $ENV{VULKAN_SDK}/Bin
    REQUIRED
)

message(STATUS ">>> glslc: ${GLSLC_EXECUTABLE}")

set(GLSL_SRC
    "${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders/shader.vert"
    "${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders/shader.frag"
)

file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/assets/shaders)

set(SPV_BINARIES)

foreach(GLSL ${GLSL_SRC})
    get_filename_component(GLSL_EXT ${GLSL} LAST_EXT)
    string(REPLACE "." "" EXT_STR ${GLSL_EXT})
    set(SPV ${CMAKE_CURRENT_BINARY_DIR}/assets/shaders/${EXT_STR}.spv)

    add_custom_command(OUTPUT ${SPV}
        COMMAND ${GLSLC_EXECUTABLE} -o ${SPV} ${GLSL}
        DEPENDS ${GLSL}
        COMMENT "Compiling ${GLSL} -> ${EXT_STR}.spv"
        VERBATIM
    )

    list(APPEND SPV_BINARIES ${SPV})
endforeach()

add_custom_target(shaders ALL DEPENDS ${SPV_BINARIES})

add_executable(vulkan_tutorial src/main.cpp)

target_include_directories(vulkan_tutorial
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

add_dependencies(vulkan_tutorial shaders)

add_custom_command(TARGET vulkan_tutorial POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_BINARY_DIR}/assets/shaders"
        "$<TARGET_FILE_DIR:vulkan_tutorial>/assets/shaders"
)

add_custom_command(TARGET vulkan_tutorial POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/assets/textures"
        "$<TARGET_FILE_DIR:vulkan_tutorial>/assets/textures"
)

add_custom_command(TARGET vulkan_tutorial POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/assets/models"
        "$<TARGET_FILE_DIR:vulkan_tutorial>/assets/models"
)

target_link_libraries(vulkan_tutorial
    PRIVATE
        Vulkan::Vulkan
        glfw
        glm::glm
)

source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/src" FILES src/main.cpp)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/assets/shaders" FILES ${GLSL_SRC})