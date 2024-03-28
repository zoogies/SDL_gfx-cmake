cmake_minimum_required(VERSION 3.16)

set(MAJOR_VERSION 2)
set(MINOR_VERSION 0)
set(MICRO_VERSION 27)

project(SDL_gfx
    LANGUAGES C
)

message(STATUS "Configuring ${PROJECT_NAME} ${PROJECT_VERSION}")

add_library(SDL_gfx STATIC
    SDL_framerate.c
    SDL_gfxBlitFunc.c
    SDL_imageFilter.c
    SDL2_gfxPrimitives.c
    SDL2_rotozoom.c
)

target_include_directories(SDL_gfx
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
)