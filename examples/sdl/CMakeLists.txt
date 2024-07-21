cmake_minimum_required(VERSION 3.5)

#LIST(APPEND CMAKE_MODULE_PATH "../../../cmake-build-debug")

project(sdl_example)

if(MSVC AND NOT VCPKG_TOOLCHAIN)
  message(FATAL_ERROR "MSVC is only supported when using vcpkg")
endif()

add_executable(sdl_example "sdl.cpp")
install(TARGETS sdl_example)

#set(LibXml2_DIR ../../../Common/cmake-install-release-emscripten/lib/cmake/libxml2-2.11.0)
find_package(LibXml2 REQUIRED CONFIG)

#set(tmx_DIR ../../../Common/cmake-install-release-emscripten/lib/cmake/tmx)
find_package(tmx REQUIRED CONFIG)
target_link_libraries(sdl_example PRIVATE tmx)

if( ${CMAKE_SYSTEM_NAME} MATCHES "Emscripten")
  set(CMAKE_EXECUTABLE_SUFFIX .html)

  set(EMSCRIPTEN_SDL_OPTS
          "SHELL: -s USE_SDL=2 -s USE_SDL_TTF=2 -s USE_SDL_MIXER=2 -s USE_SDL_IMAGE=2 -s SDL2_IMAGE_FORMATS=[png] -s USE_SDL_GFX=2")

  target_compile_options(${PROJECT_NAME} PUBLIC
          "${EMSCRIPTEN_SDL_OPTS}")
  target_link_options(${PROJECT_NAME} PUBLIC
          "${EMSCRIPTEN_SDL_OPTS}"
          --preload-file ../../data
          --emrun)
elseif(VCPKG_TOOLCHAIN)
  #find_package(sdl2 REQUIRED) # transitively included?
  find_package(sdl2-image REQUIRED)
  target_link_libraries(sdl_example PRIVATE sdl2-image)
else()
  find_package(PkgConfig REQUIRED)
  #pkg_search_module(SDL2 REQUIRED IMPORTED_TARGET sdl2 SDL2_image) # does not work for unknown reasons
  pkg_search_module(SDL2_IMAGE REQUIRED IMPORTED_TARGET SDL2_image) # transitively includes SDL2
  target_link_libraries(sdl_example PRIVATE PkgConfig::SDL2_IMAGE)
endif()
