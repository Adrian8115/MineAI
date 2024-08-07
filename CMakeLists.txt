cmake_minimum_required(VERSION 3.12)

# Mod Options
project(MineAI CXX ASM_NASM)
set(MOD_VERSION "0.1.0")
set(MOD_AUTHOR "Adrian8115")

# Amethyst Minecraft Folder
set(AmethystFolder "$ENV{localappdata}/Packages/Microsoft.MinecraftUWP_8wekyb3d8bbwe/LocalState/games/com.mojang/amethyst/") 

# Configure to build within the Amethyst Folder
configure_file(mod.json.in "${AmethystFolder}/mods/${PROJECT_NAME}@${MOD_VERSION}/mod.json" @ONLY)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO "${AmethystFolder}/mods/${PROJECT_NAME}@${MOD_VERSION}")

# Define only RelWithDebInfo as the available build configuration
set(CMAKE_CXX_STANDARD 23)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_CONFIGURATION_TYPES "RelWithDebInfo" CACHE STRING "Build configurations" FORCE)
set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: RelWithDebInfo" FORCE)

# Project Files
file(GLOB_RECURSE ${PROJECT_NAME}_All
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.asm
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.h
)

add_library(${PROJECT_NAME} SHARED ${${PROJECT_NAME}_All})

# Pass mod options to the source code
target_compile_definitions(${PROJECT_NAME} PRIVATE MOD_VERSION="${MOD_VERSION}")
target_compile_definitions(${PROJECT_NAME} PRIVATE MOD_AUTHOR="${MOD_AUTHOR}")
target_compile_definitions(${PROJECT_NAME} PRIVATE MOD_NAME="${PROJECT_NAME}")

# Link libraries
target_link_libraries(${PROJECT_NAME} PRIVATE
        AmethystAPI
        libhat
        $ENV{amethyst_src}/AmethystAPI/lib/fmt.lib
)

link_directories("{PROJECT}")

# Link Rust library
target_link_libraries(${PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/crate_out/mineai.lib
)

# Link Torch libraries
target_link_libraries(${PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/torch.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/torch_cpu.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/asmjit.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/c10.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/clog.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/cpuinfo.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/dnnl.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/fbgemm.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/fbjni.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/kineto.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/libprotobuf.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/libprotobuf-lite.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/libprotoc.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/pthreadpool.lib
        ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/pytorch_jni.lib
        # ${CMAKE_CURRENT_SOURCE_DIR}/libtorch/XNNPACK.lib
)

# Enable multi processor compilation for C++, to make it go brrrrrr
target_compile_options(${PROJECT_NAME} PRIVATE
        $<$<COMPILE_LANGUAGE:CXX>:/MP>
)

# EnTT Config Options
target_compile_definitions(${PROJECT_NAME} PUBLIC ENTT_PACKED_PAGE=128)

# Get the AmethystAPI Lib
add_subdirectory("$ENV{amethyst_src}/AmethystAPI" "build")
