cmake_minimum_required(VERSION 3.28)
project(vulkan_tutorial LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)

file(GLOB SHADERS "${CMAKE_CURRENT_SOURCE_DIR}/shaders/*.frag"
    "${CMAKE_CURRENT_SOURCE_DIR}/shaders/*.vert"
)

foreach(SHADER ${SHADERS})
    get_filename_component(FILE ${SHADER} NAME)
    add_custom_command(
        OUTPUT "${CMAKE_CURRENTBINARY_DIR}/${FILE}.spv"
        COMMAND ${Vulkan_GLSLC_EXECUTABLE} -o ${FILE}.spv ${SHADER}
        DEPENDS ${SHADER}
        COMMENT "Compiling ${FILE} to SPIR-V"
    )

    list(APPEND SPV_BINARIES "${CMAKE_CURRENT_BINARY_DIR}/${FILE}.spv")
endforeach()

add_executable(vulkan_tutorial src/main.cpp ${SPV_BINARIES})

target_link_libraries(vulkan_tutorial
    PRIVATE
        Vulkan::Vulkan
        glfw
        glm::glm
)

source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/src" FILES src/main.cpp)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}/shaders" FILES ${SHADERS})