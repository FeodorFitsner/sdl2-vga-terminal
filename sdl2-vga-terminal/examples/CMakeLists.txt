cmake_minimum_required (VERSION 3.8)


############################## Main Example ##################################
add_executable (${PROJECT_NAME} "sdl2-vga-terminal.cpp")
add_dependencies(${PROJECT_NAME} vga-terminal-obj) 
target_link_libraries(${PROJECT_NAME} vga-terminal-obj)


################## C example using SO/DLL with lazy loading ##################
add_executable (${PROJECT_NAME}-so "sdl2-vga-terminal-so.c")
add_dependencies(${PROJECT_NAME}-so vga-terminal) 
target_link_libraries(${PROJECT_NAME}-so vga-terminal)
add_custom_command(TARGET "${PROJECT_NAME}-so" POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:vga-terminal> $<TARGET_FILE_DIR:${PROJECT_NAME}-so>
)
if(MSVC)
    target_link_options(${PROJECT_NAME}-so PRIVATE 
        "/INCREMENTAL" 
        "/DELAYLOAD:$<TARGET_FILE_NAME:vga-terminal>" 
        "/DELAY:UNLOAD"
    )
endif()


################## C example using SO/DLL run-time loading ##################
add_executable (${PROJECT_NAME}-load-so "sdl2-vga-terminal-load-so.c")
target_link_libraries(${PROJECT_NAME}-load-so PUBLIC SDL2::SDL2 SDL2::SDL2main)
add_custom_command(TARGET "${PROJECT_NAME}-load-so" POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different $<TARGET_FILE:vga-terminal> $<TARGET_FILE_DIR:${PROJECT_NAME}-load-so>
)
