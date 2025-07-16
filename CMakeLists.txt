cmake_minimum_required(VERSION 3.28)
project(vulkan_tutorial LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)

set(GLSL_SRC
    ${CMAKE_CURRENT_SOURCE_DIR}/shaders/shader.vert
    ${CMAKE_CURRENT_SOURCE_DIR}/shaders/shader.frag
)

set(SPV_BINARIES "")

foreach(GLSL ${GLSL_SRC})
    get_filename_component(GLSL_NAME ${GLSL} NAME)
    set(SPV_FILE ${CMAKE_CURRENT_BINARY_DIR}/${GLSL_NAME}.spv)

    add_custom_command(
        OUTPUT ${SPV_FILE}
        COMMAND ${Vulkan_GLSLC_EXECUTABLE} -o ${SPV_FILE} ${GLSL}
        DEPENDS ${GLSL}
        COMMENT "Compiling ${GLSL_NAME} -> SPIR-V"
        VERBATIM
    )

    list(APPEND SPV_BINARIES ${SPV_FILE})
endforeach()

add_custom_target(shaders ALL DEPENDS ${SPV_BINARIES})

add_executable(vulkan_tutorial
    src/main.cpp
)

add_dependencies(vulkan_tutorial shaders)

target_link_libraries(vulkan_tutorial
    PRIVATE
        Vulkan::Vulkan
        glfw
        glm::glm
)

source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/src" FILES src/main.cpp)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/shaders" FILES ${GLSL_SRC})