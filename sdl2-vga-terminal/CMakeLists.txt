﻿cmake_minimum_required (VERSION 3.8)

set (CMAKE_CXX_STANDARD 17)
set (CMAKE_C_STANDARD 11)
set (CMAKE_DEBUG_POSTFIX d)

find_package(SDL2 CONFIG REQUIRED)


############################ Shared lib ######################################
include(GenerateExportHeader)
add_library(vga-terminal SHARED)
generate_export_header(
    vga-terminal
    BASE_NAME vga-terminal
    EXPORT_MACRO_NAME VGA_TERMINAL_EXPORT
    EXPORT_FILE_NAME vga-terminal-export.h
    STATIC_DEFINE vga-terminal-lib_BUILD_AS_STATIC
)
install(
    TARGETS vga-terminal 
    EXPORT vga-terminal-config
    DESTINATION lib
    PUBLIC_HEADER DESTINATION include
)
install(FILES 
    ${CMAKE_CURRENT_SOURCE_DIR}/include/vga-terminal.h
    $<TARGET_FILE_DIR:vga-terminal>/vga-terminal-export.h 
    DESTINATION include
)
install(FILES
    $<TARGET_FILE:SDL2::SDL2>
    DESTINATION lib
)
export(TARGETS vga-terminal 
    FILE ${CMAKE_BINARY_DIR}/vga-terminal.cmake
)
install(EXPORT
    vga-terminal-config DESTINATION cmake
)
target_include_directories(vga-terminal PUBLIC 
    $<BUILD_INTERFACE:$<TARGET_FILE_DIR:vga-terminal>>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
target_sources(vga-terminal PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/VgaTerminal.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vga-terminal.cpp
)
target_link_libraries(vga-terminal SDL2::SDL2 SDL2::SDL2main)


########################### Static Lib #######################################
add_library(vga-terminal-static STATIC)
install(
    TARGETS vga-terminal-static
    EXPORT vga-terminal-static-config
    DESTINATION static/lib
    PUBLIC_HEADER DESTINATION static/include
)
install(FILES 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/VgaTerminal.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.hpp
    DESTINATION static/include
)

export(TARGETS vga-terminal-static
    FILE ${CMAKE_BINARY_DIR}/vga-terminal-static.cmake
)
install(EXPORT
    vga-terminal-static-config DESTINATION static/cmake
)
target_include_directories(vga-terminal-static PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:static/include>
)
target_sources(vga-terminal-static PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/VgaTerminal.cpp
)
target_link_libraries(vga-terminal-static SDL2::SDL2 SDL2::SDL2main)


################ Shared code among exe and tests #############################
add_library(vga-terminal-obj OBJECT)
target_sources(vga-terminal-obj PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/VgaTerminal.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Window.cpp
)

target_include_directories(vga-terminal-obj PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)
if(WITH_SDL2_STATIC)
    target_link_libraries(vga-terminal-obj PUBLIC SDL2::SDL2-static SDL2::SDL2main)
else()
    install(FILES
        $<TARGET_FILE:SDL2::SDL2>
        DESTINATION static/lib
    )
    target_link_libraries(vga-terminal-obj PUBLIC SDL2::SDL2 SDL2::SDL2main)
endif()

### Examples
if(WITH_EXAMPLES)
    add_subdirectory("examples")
endif()


############################### tests ########################################
if(WITH_TEST)
add_subdirectory("test")
endif()
