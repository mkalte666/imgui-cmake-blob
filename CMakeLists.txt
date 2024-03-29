#    CMakeLists.txt for somewhat nice imgui loading
#
#    Copyright (C) 2018  Malte Kießling
#
#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    any later version.

#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.

#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.1.3 FATAL_ERROR)
project(imgui)

#cpp 11, we need find_sdl2 for the includes and also disallow in-source builds 
set (CMAKE_CXX_STANDARD 11)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

# gl3w
set(gl3wGen ${CMAKE_CURRENT_SOURCE_DIR}/gl3w/gl3w_gen.py)
set(gl3wTargetDir ${CMAKE_BINARY_DIR}/gl3w)
set(gl3wIncludeDir ${gl3wTargetDir}/include)
set(gl3wSource ${gl3wTargetDir}/src/gl3w.c)

# we dont use gl3ws cmake because we want the files at config time
find_package(Python COMPONENTS Interpreter REQUIRED)
if(Python_Interpreter_FOUND)
    message("Generating gl3w with Python ${Python_VERSION} into ${gl3wTargetDir}")
    file(MAKE_DIRECTORY ${gl3wTargetDir})
    execute_process(COMMAND ${Python_EXECUTABLE} ${gl3wGen} --root=${gl3wTargetDir} --ext
        WORKING_DIRECTORY ${gl3wTargetDir})
endif(Python_Interpreter_FOUND)

add_library(gl3w STATIC ${gl3wSource})
target_include_directories(gl3w PUBLIC ${gl3wIncludeDir})
if(NOT WIN32)
    target_link_libraries(gl3w dl)
endif()
set_property(TARGET gl3w PROPERTY POSITION_INDEPENDENT_CODE ON)

# imgui 
find_package(SDL2 REQUIRED)
# these sources are quite api specific, but i dont care much 
set(imguiSources 
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_demo.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_draw.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_internal.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_tables.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_widgets.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imstb_rectpack.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imstb_textedit.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imstb_truetype.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/misc/cpp/imgui_stdlib.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/misc/cpp/imgui_stdlib.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends/imgui_impl_sdl2.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends/imgui_impl_sdl2.h
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends/imgui_impl_opengl3.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends/imgui_impl_opengl3.h)
   
add_library(imgui STATIC ${imguiSources})
target_link_libraries(imgui gl3w)
target_compile_definitions(imgui PUBLIC IMGUI_IMPL_OPENGL_LOADER_GL3W)
target_include_directories(imgui SYSTEM PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui/)
target_include_directories(imgui PRIVATE ${SDL2_INCLUDE_DIR})
set_property(TARGET imgui PROPERTY POSITION_INDEPENDENT_CODE ON)

# done! i think...