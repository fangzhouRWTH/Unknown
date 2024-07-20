cmake_minimum_required(VERSION 3.25.0)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
project(UnknownEngine VERSION 0.1.0 LANGUAGES C CXX)
project(Demo VERSION 0.1.0 LANGUAGES C CXX)

#include(CTest)
#enable_testing()

#find_package(Vulkan REQUIRED)
add_subdirectory(${CMAKE_SOURCE_DIR}/source/external/GLAD)
add_subdirectory(${CMAKE_SOURCE_DIR}/source/external/CATCH2/Catch2)
add_subdirectory(${CMAKE_SOURCE_DIR}/source/external/VKBOOTSTRAP/vk-bootstrap)
link_directories(${CMAKE_SOURCE_DIR}/source/external/GLFW/glfw/build/src/)
link_directories(${CMAKE_SOURCE_DIR}/source/external/SPD/spdlog/build)
link_directories(${CMAKE_SOURCE_DIR}/source/external/FMT/fmt/build)

link_directories(/home/fzl/vulkanSDK/1.3.283.0/x86_64/lib)

#todo use correct way to link dynamic lib
link_directories(${CMAKE_SOURCE_DIR}/source/external/ASSIMP/assimp/bin)

#find_library(Assimp assimp.so ${CMAKE_SOURCE_DIR}/build/libassimp.so)
#add_library( Assimp SHARED IMPORTED )
#set_target_properties( Assimp PROPERTIES IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/build/libassimp.so )

set(RENDERER_SOURCE_FILES
    ${CMAKE_SOURCE_DIR}/source/project/app/main.cpp
    ${CMAKE_SOURCE_DIR}/source/project/app/engine.cpp
    ${CMAKE_SOURCE_DIR}/source/project/app/application.cpp
    ${CMAKE_SOURCE_DIR}/source/project/app/getApp.cpp
    ${CMAKE_SOURCE_DIR}/source/project/app/stringTable.cpp

    ${CMAKE_SOURCE_DIR}/source/project/renderer/renderer.cpp

    ${CMAKE_SOURCE_DIR}/source/project/renderer/vulkan/vkCore.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/vulkan/vkInitializer.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/vulkan/vkUtils.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/vulkan/vkDescriptor.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/vulkan/vkPipeline.cpp

    #${CMAKE_SOURCE_DIR}/source/project/renderer/shader.cpp

    ${CMAKE_SOURCE_DIR}/source/project/configuration/globalValues.cpp

    ${CMAKE_SOURCE_DIR}/source/project/debug/log.cpp

    ${CMAKE_SOURCE_DIR}/source/project/core/math.cpp
    ${CMAKE_SOURCE_DIR}/source/project/core/hash.cpp

    ${CMAKE_SOURCE_DIR}/source/project/core/input.cpp

    ${CMAKE_SOURCE_DIR}/source/project/core/clock.cpp

    ${CMAKE_SOURCE_DIR}/source/project/framework/framework.cpp

    ${CMAKE_SOURCE_DIR}/source/project/memory/allocator.cpp

    ${CMAKE_SOURCE_DIR}/source/project/ecs/ecs.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/renderSystem/renderSystem.cpp
    ${CMAKE_SOURCE_DIR}/source/project/renderer/rendererHandles.cpp

    ${CMAKE_SOURCE_DIR}/source/project/asset/modelLoader.cpp
    ${CMAKE_SOURCE_DIR}/source/project/asset/resourceManager.cpp
    #${CMAKE_SOURCE_DIR}/source/project/asset/assetManager.cpp
    ${CMAKE_SOURCE_DIR}/source/project/asset/textureLoader.cpp
    #{CMAKE_SOURCE_DIR}/source/project/asset/texture.cpp

    ${CMAKE_SOURCE_DIR}/source/project/world/world.cpp
    ${CMAKE_SOURCE_DIR}/source/project/world/scene.cpp

    ${CMAKE_SOURCE_DIR}/source/project/renderer/gui/simpleGUI.cpp
    #todo move out as dependency
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui.cpp
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui_draw.cpp
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui_tables.cpp
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui_widgets.cpp
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui_impl_vulkan.cpp
    ${CMAKE_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/src/imgui_impl_glfw.cpp
    )

    add_executable(UnknownEngine ${RENDERER_SOURCE_FILES})
#===================================shaders==================================

# function(add_shaders TARGET_NAME)
#   set(SHADER_SOURCE_FILES ${ARGN}) # the rest of arguments to this function will be assigned as shader source files
  
#   # Validate that source files have been passed
#   list(LENGTH SHADER_SOURCE_FILES FILE_COUNT)
#   if(FILE_COUNT EQUAL 0)
#     message(FATAL_ERROR "Cannot create a shaders target without any source files")
#   endif()

#   set(SHADER_COMMANDS)
#   set(SHADER_PRODUCTS)

#   foreach(SHADER_SOURCE IN LISTS SHADER_SOURCE_FILES)
#     cmake_path(ABOSULTE_PATH SHADER_SOURCE NORMALIZE)
#     cmake_path(GET SHADER_SOURCE FILENAME SHADER_NAME)
    
#     # Build command
#     list(APPEND SHADER_COMMAND COMMAND)
#     list(APPEND SHADER_COMMAND Vulkan::glslc)
#     list(APPEND SHADER_COMMAND "${SHADER_SOURCE}")
#     list(APPEND SHADER_COMMAND "-o")
#     list(APPEND SHADER_COMMAND "${CMAKE_CURRENT_BINARY_DIR}/${SHADER_NAME}.spv")

#     # Add product
#     list(APPEND SHADER_PRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/${SHADER_NAME}.spv")

#   endforeach()

#   add_custom_targets(${TARGET_NAME} ALL
#     ${SHADER_COMMANDS}
#     COMMENT "Compiling Shaders [${TARGET_NAME}]"
#     SOURCES ${SHADER_SOURCE_FILES}
#     BYPRODUCTS ${SHADER_PRODUCTS}
#   )
# endfunction()

#/home/fzl/vulkan/1.3.283.0/x86_64/bin                        
find_program(GLSL_VALIDATOR glslangValidator HINTS /usr/bin /usr/local/bin $ENV{VULKAN_SDK}/Bin/ $ENV{VULKAN_SDK}/Bin32/)
file(GLOB_RECURSE GLSL_SOURCE_FILES
    "${PROJECT_SOURCE_DIR}/source/project/renderer/shaders/*.frag"
    "${PROJECT_SOURCE_DIR}/source/project/renderer/shaders/*.vert"
    "${PROJECT_SOURCE_DIR}/source/project/renderer/shaders/*.comp"
    )

foreach(GLSL ${GLSL_SOURCE_FILES})
  message(STATUS "BUILDING SHADER")
  get_filename_component(FILE_NAME ${GLSL} NAME)
  set(SPIRV "${PROJECT_SOURCE_DIR}/source/project/renderer/shaders/${FILE_NAME}.spv")
  message(STATUS COMMAND\ ${GLSL_VALIDATOR}\ -V\ ${GLSL}\ -o\ ${SPIRV})
  add_custom_command(
    OUTPUT ${SPIRV}
    COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
    DEPENDS ${GLSL})
  list(APPEND SPIRV_BINARY_FILES ${SPIRV})
endforeach(GLSL)

add_custom_target(Shaders DEPENDS ${SPIRV_BINARY_FILES})
#===================================shaders==================================
add_dependencies(UnknownEngine Shaders)

#target_link_libraries(UnknownEngine PRIVATE GLAD)
target_link_libraries(UnknownEngine PRIVATE libspdlog.a)
target_link_libraries(UnknownEngine PRIVATE libglfw3.a)
target_link_libraries(UnknownEngine PRIVATE libfmt.a)
target_link_libraries(UnknownEngine PRIVATE libvulkan.so)

target_link_libraries(UnknownEngine PRIVATE vk-bootstrap::vk-bootstrap)

target_link_libraries(UnknownEngine PUBLIC libassimp.so)

target_include_directories(UnknownEngine PUBLIC
                          "${PROJECT_BINARY_DIR}"
                          #"${PROJECT_SOURCE_DIR}/source/external/GLAD/include"
                          "${PROJECT_SOURCE_DIR}/source/external/EIGEN/eigen"
                          "${PROJECT_SOURCE_DIR}/source/external/SPD/spdlog/include"
                          )

target_include_directories(UnknownEngine PRIVATE
                          "/home/fzl/vulkan/1.3.283.0/x86_64/include"
                          "${PROJECT_SOURCE_DIR}/source/external/GLAD/include"
                          "${PROJECT_SOURCE_DIR}/source/external/GLFW/glfw/include"
                          "${PROJECT_SOURCE_DIR}/source/external/STB"
                          "${PROJECT_SOURCE_DIR}/source/external/FMT/fmt/include"
                          "${PROJECT_SOURCE_DIR}/source/external/ASSIMP/assimp/include"
                          "${PROJECT_SOURCE_DIR}/source/external/NLOHMANN/json/include"
                          "${PROJECT_SOURCE_DIR}/source/external/VKBOOTSTRAP/vk-bootstrap/src"
                          "${PROJECT_SOURCE_DIR}/source/external/IMGUI/vulkan_glfw/include"
                          )
                          
target_include_directories(UnknownEngine PUBLIC
                          "${PROJECT_SOURCE_DIR}/source/project"
                          )
###############################
set(
    TEST_SOURCE_FILES
    ${CMAKE_SOURCE_DIR}/source/project/configuration/globalValues.cpp
    ${CMAKE_SOURCE_DIR}/source/project/debug/log.cpp
    ${CMAKE_SOURCE_DIR}/source/project/core/math.cpp
    ${CMAKE_SOURCE_DIR}/source/project/memory/allocator.cpp
    #${CMAKE_SOURCE_DIR}/source/project/ecs/ecs.cpp
    )

add_executable(
    UNIT_TEST_ECS 
    ${CMAKE_SOURCE_DIR}/source/project/UNIT_TESTS/test_ecs.cpp
    ${CMAKE_SOURCE_DIR}/source/project/ecs/ecs.cpp
    ${TEST_SOURCE_FILES}
    )

target_link_libraries(UNIT_TEST_ECS PRIVATE Catch2::Catch2WithMain)
target_link_libraries(UNIT_TEST_ECS PRIVATE libspdlog.a)
target_link_libraries(UNIT_TEST_ECS PRIVATE libfmt.a)

target_include_directories(UNIT_TEST_ECS PRIVATE
                          "${PROJECT_SOURCE_DIR}/source/external/CATCH2/Catch2/src"
                          )

target_include_directories(UNIT_TEST_ECS PRIVATE
                          "${PROJECT_SOURCE_DIR}/source/project"
                          "${PROJECT_SOURCE_DIR}/source/external/EIGEN/eigen"
                          "${PROJECT_SOURCE_DIR}/source/external/SPD/spdlog/include"
                          "${PROJECT_SOURCE_DIR}/source/external/FMT/fmt/include"
                          )

add_executable(
                            UNIT_TEST_STRUCTURE 
                            ${CMAKE_SOURCE_DIR}/source/project/UNIT_TESTS/test_structure.cpp
                            )     

 target_link_libraries(UNIT_TEST_STRUCTURE PRIVATE Catch2::Catch2WithMain)
                            target_link_libraries(UNIT_TEST_ECS PRIVATE libspdlog.a)
                            target_link_libraries(UNIT_TEST_ECS PRIVATE libfmt.a)

target_include_directories(UNIT_TEST_STRUCTURE PRIVATE
                            "${PROJECT_SOURCE_DIR}/source/project")

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)
