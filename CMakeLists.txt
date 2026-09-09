cmake_minimum_required(VERSION 3.15...3.31)
project(Defblade VERSION 1.0
                  DESCRIPTION "Pixel art platformer"
                  LANGUAGES CXX)

set(CMAKE_CPP_COMPILER g++)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(PORTMASTER_BUILD "Force a fullscreen window for the PortMaster handheld build" OFF)

# sources
set(SOURCES main.cpp src/anim.cpp src/sparks.cpp src/entities.cpp src/health_bars.cpp src/particles.cpp src/player.cpp src/timer.cpp src/weapons.cpp src/water.cpp src/coin.cpp)

add_executable(Defblade ${SOURCES})

if (WIN32)
    # for release
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mwindows")

    # -Iinclude (bundled Windows/MinGW SDL2 headers + nlohmann JSON)
    target_include_directories(Defblade PRIVATE ${CMAKE_SOURCE_DIR}/include)
    # -Llib (bundled Windows/MinGW-built SDL2 static libs)
    target_link_directories(Defblade PRIVATE ${CMAKE_SOURCE_DIR}/lib)

    # libraries to compile with -lSDL2main ...
    set(SDL2_LIBRARIES mingw32 SDL2main SDL2 SDL2_image SDL2_mixer SDL2_ttf)
    target_link_libraries(Defblade PUBLIC ${SDL2_LIBRARIES})

    target_sources(Defblade PRIVATE resources.rc)
else()
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SDL2 REQUIRED IMPORTED_TARGET sdl2)
    pkg_check_modules(SDL2_IMAGE REQUIRED IMPORTED_TARGET SDL2_image)
    pkg_check_modules(SDL2_MIXER REQUIRED IMPORTED_TARGET SDL2_mixer)
    pkg_check_modules(SDL2_TTF REQUIRED IMPORTED_TARGET SDL2_ttf)

    # Only expose the bundled nlohmann JSON header (include/JSON/json.hpp),
    # copied into its own include root at configure time. The sibling
    # bundled include/SDL2 (Windows/MinGW headers) must never be exposed
    # here: GCC always searches -I paths before -isystem ones regardless
    # of command-line order (and CMake silently drops a plain -I for
    # /usr/include or /usr/local/include as redundant with the compiler's
    # own implicit search path, so parent-dir reordering can't fix this
    # either) - a plain "-Iinclude" would make the game's own quoted
    # "SDL2/SDL.h" includes resolve against the bundled Windows headers
    # instead of PkgConfig::SDL2's system ones below, even though the
    # binary links against the system .so.
    file(COPY ${CMAKE_SOURCE_DIR}/include/JSON DESTINATION ${CMAKE_BINARY_DIR}/json_include)
    target_include_directories(Defblade PRIVATE ${CMAKE_BINARY_DIR}/json_include)

    target_link_libraries(Defblade PUBLIC
        PkgConfig::SDL2
        PkgConfig::SDL2_IMAGE
        PkgConfig::SDL2_MIXER
        PkgConfig::SDL2_TTF
    )

    if (PORTMASTER_BUILD)
        target_compile_definitions(Defblade PRIVATE PORTMASTER_BUILD)
    endif()
endif()
