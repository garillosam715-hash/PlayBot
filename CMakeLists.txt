cmake_minimum_required(VERSION 3.21)

# ---------------------------------------------------------------------------
# PlayBot - a Geode mod for Geometry Dash
# Targets: Geode v5.7.1 / Geometry Dash 2.2081
# ---------------------------------------------------------------------------
# Build targets:
#   Windows (win)          -> geode build -p win
#   macOS  (mac / mac-arm)  -> geode build -p mac
#   Android 64-bit          -> geode build -p android64
#   Android 32-bit (v7a)    -> geode build -p android32   <-- "armeabi-v7a"
#
# Geode's CLI drives the Android NDK toolchain for you, but this file makes
# the armeabi-v7a target explicit in case you invoke CMake/NDK directly
# instead of through `geode build`.
# ---------------------------------------------------------------------------

project(PlayBot VERSION 1.0.0)

if (ANDROID)
    # Explicitly support the 32-bit ARM ABI ("armeabi-v7a") alongside 64-bit.
    # Geode's CLI normally sets ANDROID_ABI itself when you pass
    # `-p android32`, but we pin a sane default here too.
    if (NOT DEFINED ANDROID_ABI)
        set(ANDROID_ABI "armeabi-v7a" CACHE STRING "Android ABI" FORCE)
    endif()

    message(STATUS "PlayBot: building for Android ABI: ${ANDROID_ABI}")

    if (ANDROID_ABI STREQUAL "armeabi-v7a")
        add_compile_definitions(PLAYBOT_ANDROID32=1)
    elseif (ANDROID_ABI STREQUAL "arm64-v8a")
        add_compile_definitions(PLAYBOT_ANDROID64=1)
    endif()
endif()

file(GLOB SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
)

add_library(${PROJECT_NAME} SHARED ${SOURCES})

# Pull in the Geode SDK (expects the `geode-sdk` submodule/dependency to be
# resolved by the Geode CLI / CMake package, as in a normal Geode mod).
if (NOT DEFINED GEODE_LOADER_PATH)
    message(STATUS "PlayBot: expecting Geode CLI to supply the SDK via CPM/FetchContent")
endif()

target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

# This line is what actually links the Geode SDK + Cocos2d headers when
# building through the official Geode CLI / CMake toolchain file.
setup_geode_mod(${PROJECT_NAME})
