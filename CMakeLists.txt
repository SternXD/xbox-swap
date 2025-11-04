cmake_minimum_required( VERSION 3.16 )

if(CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
    # If not set via command line, default to 10.0
    if("${CMAKE_SYSTEM_VERSION}" STREQUAL "")
        set(CMAKE_SYSTEM_VERSION "10.0" CACHE STRING "Windows Store version" FORCE)
    endif()
endif()

project(xbox-fmalloc)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 11)

if(CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
    add_definitions(-D_XBOX_UWP)
endif()

add_library( xbox-fmalloc SHARED src/dllmain.cpp src/fmalloc.cpp externals/o1heap/o1heap/o1heap.c )

# UWP requires dynamic runtime, regular Windows can use static
if(CMAKE_SYSTEM_NAME STREQUAL "WindowsStore")
    set_property(TARGET xbox-fmalloc PROPERTY
                 MSVC_RUNTIME_LIBRARY "$<$<CONFIG:Debug>:MultiThreadedDebugDLL>$<$<CONFIG:Release>:MultiThreadedDLL>")
else()
    set_property(TARGET xbox-fmalloc PROPERTY
                 MSVC_RUNTIME_LIBRARY "$<$<CONFIG:Debug>:MultiThreadedDebug>$<$<CONFIG:Release>:MultiThreaded>")
endif()
target_compile_definitions( xbox-fmalloc PRIVATE -D_CRT_SECURE_NO_WARNINGS -DWIN32_LEAN_AND_MEAN -DXBOXFMALLOC_EXPORTS -D_WINDOWS -D_USRDLL )
target_include_directories( xbox-fmalloc 
    INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
    PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/src"
            "${CMAKE_CURRENT_SOURCE_DIR}/externals/o1heap/o1heap" )
target_link_libraries( xbox-fmalloc PRIVATE onecore.lib)