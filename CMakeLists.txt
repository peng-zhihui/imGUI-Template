cmake_minimum_required(VERSION 3.0)

project(imgui_cmake_starter)

add_compile_options(/source-charset:utf-8 /execution-charset:utf-8)

# add header path
include_directories(
	${CMAKE_CURRENT_SOURCE_DIR}/imgui-1.83
	${CMAKE_CURRENT_SOURCE_DIR}/imgui-1.83/backends
)

if (APPLE)
    # for <GLFW/glfw3.h>
    include_directories(
        /usr/local/include
        /opt/local/include
        /opt/homebrew/include
    )
endif()

# set common source
file(GLOB SRC
    ./imgui-1.83/*.h
    ./imgui-1.83/*.cpp
)

# set specific source and other option for platform
if (WIN32)
    file (GLOB PLATFORM_SRC
        ./imgui-1.83/backends/imgui_impl_win32.*
        ./imgui-1.83/backends/imgui_impl_dx12.*
        ./src/win/main.cpp
    )
elseif (UNIX)
    # support both mac and linux
    add_definitions(-DIMGUI_IMPL_OPENGL_LOADER_GL3W)

    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/imgui-1.83/examples/libs/gl3w # for GL/gl3w.h
    )

    file (GLOB PLATFORM_SRC
        ./imgui-1.83/examples/libs/gl3w/GL/gl3w.*
        ./imgui-1.83/backends/imgui_impl_glfw.*
        ./imgui-1.83/backends/imgui_impl_opengl3.*
        ./src/unix/main.cpp
    )
endif()

# add link path
if (APPLE)
    link_directories(
        /usr/local/lib
        /opt/local/lib
        /opt/homebrew/lib
    )
endif()

# generate binary
add_executable(${PROJECT_NAME} ${SRC} ${PLATFORM_SRC})

# link lib, should install glfw first or prebuild lib and embed in project
if (WIN32)
    target_link_libraries(${PROJECT_NAME}
        d3d12.lib
        d3dcompiler.lib
        dxgi.lib
    )
elseif (APPLE)
    # mac: brew install glfw3
    find_library(OPENGL_LIBRARY OpenGL REQUIRED)
    find_library(COCOA_LIBRARY Cocoa REQUIRED)
    find_library(IOKIT_LIBRARY IOKit REQUIRED)
    find_library(COREVID_LIBRARY CoreVideo REQUIRED)
    message(${COCOA_LIBRARY})
    message(${IOKIT_LIBRARY})
    message(${COREVID_LIBRARY})

    target_link_libraries(${PROJECT_NAME}
        ${OPENGL_LIBRARY}
        ${COCOA_LIBRARY}
        ${IOKIT_LIBRARY}
        ${COREVID_LIBRARY}
        glfw # use this lib name
    )
elseif (UNIX AND NOT APPLE)
    # linux: sudo apt install libglfw3-dev
    target_link_libraries(${PROJECT_NAME}
        GL
        glfw # use this lib name
        dl
    )
endif()
