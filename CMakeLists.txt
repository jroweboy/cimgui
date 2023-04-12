cmake_minimum_required(VERSION 3.1)
project(cimgui)
set (CMAKE_CXX_STANDARD 11)

# Implementation options
option(IMPL_WIN32 "add Win32 implementation" OFF)
option(IMPL_DX9 "add DX9 implementation" OFF)
option(IMPL_DX11 "add DX11 implementation" OFF)
option(IMPL_GLFW "add GLFW implementation" OFF)
option(IMPL_OGL3 "add OpenGL3 implementation" OFF)
option(IMPL_OGL2 "add OpenGL2 implementation" OFF)
option(IMPL_SDL2 "add SDL2 implementation" OFF)
# option(IMPL_SDL3 "add SDL3 implementation" OFF)
option(IMPL_VULKAN "add Vulkan implementation" OFF)
option(IMPL_DX12 "add DX12 implementation" OFF)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_tables.cpp)
	set(TABLES_SOURCE "imgui/imgui_tables.cpp")
else()
	set(TABLES_SOURCE "")
endif()


#general settings
file(GLOB IMGUI_SOURCES
    cimgui.cpp
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_demo.cpp
    imgui/imgui_widgets.cpp
	${TABLES_SOURCE}
)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")
set(IMGUI_LIBRARIES )

if(IMGUI_FREETYPE)
	FIND_PACKAGE(freetype REQUIRED PATHS ${FREETYPE_PATH})
	list(APPEND IMGUI_LIBRARIES freetype)
	list(APPEND IMGUI_SOURCES imgui/misc/freetype/imgui_freetype.cpp)
	add_definitions("-DCIMGUI_FREETYPE=1")
endif(IMGUI_FREETYPE)

# implementations
if(IMPL_WIN32)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_win32.cpp)
endif(IMPL_WIN32)

if(IMPL_DX9)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_dx9.cpp)
endif(IMPL_DX9)

if(IMPL_DX11)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_dx11.cpp)
endif(IMPL_DX11)

if(IMPL_GLFW)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_glfw.cpp)
endif(IMPL_GLFW)

if(IMPL_OGL3)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_opengl3.cpp)
endif(IMPL_OGL3)

if(IMPL_OGL2)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_opengl2.cpp)
endif(IMPL_OGL2)

if(IMPL_SDL2)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_sdl2.cpp)
endif(IMPL_SDL2)

if(IMPL_SDL3)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_sdl3.cpp)
endif(IMPL_SDL3)

if(IMPL_VULKAN)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_vulkan.cpp)
endif(IMPL_VULKAN)

if(IMPL_DX12)
    list(APPEND IMGUI_SOURCES ./imgui/backends/imgui_impl_dx12.cpp)

	# 32-bit support requires a define
	if (${CMAKE_SIZEOF_VOID_P} MATCHES 4)
		add_compile_definitions("ImTextureID=ImU64")
	endif ()
endif(IMPL_DX12)

#add library and link
if (IMGUI_STATIC)
    add_library(cimgui STATIC ${IMGUI_SOURCES})
else (IMGUI_STATIC)
    add_library(cimgui SHARED ${IMGUI_SOURCES})
endif (IMGUI_STATIC)

target_compile_definitions(cimgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)
if (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t__declspec\(dllexport\))
    list(APPEND IMGUI_LIBRARIES imm32)
else (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t)
endif (WIN32)

target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
set_target_properties(cimgui PROPERTIES PREFIX "")
target_link_libraries(cimgui ${IMGUI_LIBRARIES})

# implementations part 2

if(IMPL_OGL2)
	if(WIN32)
		target_link_libraries(cimgui opengl32.lib)
	endif()
endif()

if(IMPL_GLFW)
	If(MSVC)
	    target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui/examples/libs/glfw/include)
		# Support both 32 and 64 bit builds
		if (${CMAKE_SIZEOF_VOID_P} MATCHES 8)
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/imgui/examples/libs/glfw/lib-vc2010-64/glfw3.lib")
		else ()
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/imgui/examples/libs/glfw/lib-vc2010-32/glfw3.lib")
		endif ()
		target_link_libraries(cimgui "legacy_stdio_definitions.lib")
    else()
        find_package(PkgConfig REQUIRED)
        pkg_search_module(GLFW REQUIRED glfw3)
        include_directories(${GLFW_INCLUDE_DIRS})
        if(IMGUI_STATIC)
            target_link_libraries(cimgui ${GLFW_STATIC_LIBRARIES})
        else()
            target_link_libraries(cimgui ${GLFW_LIBRARIES})
        endif()
	endif()
endif()

if(IMPL_DX12)
	target_link_libraries(cimgui d3d12.lib dxgi.lib d3dcompiler.lib dxguid.lib)
endif()

if(IMPL_SDL2)
	if(MSVC)
		list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win")
		set(SDL2_INCLUDE_DIRS "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/include")

		# Support both 32 and 64 bit builds
		if (${CMAKE_SIZEOF_VOID_P} MATCHES 8)
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/lib/x64/SDL2.lib")
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/lib/x64/SDL2main.lib")
		else ()
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/lib/x86/SDL2.lib")
			target_link_libraries(cimgui "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/lib/x86/SDL2main.lib")
		endif ()
		
		set(SDL2_DIR "${CMAKE_CURRENT_LIST_DIR}/deps/sdl-win/cmake")
	endif()
	
	find_package(SDL2 REQUIRED)
	include_directories(${SDL2_INCLUDE_DIRS})
	target_link_libraries(cimgui ${SDL2_LIBRARIES})
endif(IMPL_SDL2)

if(IMPL_VULKAN)
	find_package(Vulkan REQUIRED)
	target_include_directories(cimgui PUBLIC "$ENV{VULKAN_SDK}/Include")
	target_link_libraries(cimgui ${Vulkan_LIBRARIES})
endif(IMPL_VULKAN)

#install
install(TARGETS cimgui
    RUNTIME DESTINATION  .
    LIBRARY DESTINATION  .
    ARCHIVE DESTINATION  .
)

#test
set(CIMGUI_TEST "no" CACHE STRING "Enable compilation of a test unit based on imgui null")

if (CIMGUI_TEST)
  add_subdirectory(test)
endif ()