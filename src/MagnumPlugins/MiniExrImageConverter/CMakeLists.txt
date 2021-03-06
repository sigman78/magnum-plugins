#
#   This file is part of Magnum.
#
#   Copyright © 2010, 2011, 2012, 2013, 2014, 2015, 2016, 2017
#             Vladimír Vondruš <mosra@centrum.cz>
#
#   Permission is hereby granted, free of charge, to any person obtaining a
#   copy of this software and associated documentation files (the "Software"),
#   to deal in the Software without restriction, including without limitation
#   the rights to use, copy, modify, merge, publish, distribute, sublicense,
#   and/or sell copies of the Software, and to permit persons to whom the
#   Software is furnished to do so, subject to the following conditions:
#
#   The above copyright notice and this permission notice shall be included
#   in all copies or substantial portions of the Software.
#
#   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#   IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#   THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#   LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#   FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
#   DEALINGS IN THE SOFTWARE.
#

if(BUILD_STATIC)
    set(MAGNUM_MINIEXRIMAGECONVERTER_BUILD_STATIC 1)
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/configure.h.cmake
               ${CMAKE_CURRENT_BINARY_DIR}/configure.h)

set(MiniExrImageConverter_SRCS
    MiniExrImageConverter.cpp)

set(MiniExrImageConverter_HEADERS
    MiniExrImageConverter.h)

# Objects shared between plugin and test library
add_library(MiniExrImageConverterObjects OBJECT
    ${MiniExrImageConverter_SRCS}
    ${MiniExrImageConverter_HEADERS})
target_include_directories(MiniExrImageConverterObjects PUBLIC
    $<TARGET_PROPERTY:Magnum::Magnum,INTERFACE_INCLUDE_DIRECTORIES>
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_BINARY_DIR}/src)
target_compile_definitions(MiniExrImageConverterObjects PRIVATE "MiniExrImageConverterObjects_EXPORTS")
# Include the stb_*.h files as a system directory to supress warnings
target_include_directories(MiniExrImageConverterObjects SYSTEM PRIVATE ${PROJECT_SOURCE_DIR}/src/external/miniexr)
if(NOT BUILD_STATIC OR BUILD_STATIC_PIC)
    set_target_properties(MiniExrImageConverterObjects PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

# MiniExrImageConverter plugin
add_plugin(MiniExrImageConverter
    "${MAGNUM_PLUGINS_IMAGECONVERTER_DEBUG_BINARY_INSTALL_DIR};${MAGNUM_PLUGINS_IMAGECONVERTER_DEBUG_LIBRARY_INSTALL_DIR}"
    "${MAGNUM_PLUGINS_IMAGECONVERTER_RELEASE_BINARY_INSTALL_DIR};${MAGNUM_PLUGINS_IMAGECONVERTER_RELEASE_LIBRARY_INSTALL_DIR}"
    MiniExrImageConverter.conf
    $<TARGET_OBJECTS:MiniExrImageConverterObjects>
    pluginRegistration.cpp)
if(BUILD_STATIC_PIC)
    set_target_properties(MiniExrImageConverter PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()
target_include_directories(MiniExrImageConverter PUBLIC
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_BINARY_DIR}/src)
target_link_libraries(MiniExrImageConverter Magnum::Magnum)

install(FILES ${MiniExrImageConverter_HEADERS} DESTINATION ${MAGNUM_PLUGINS_INCLUDE_INSTALL_DIR}/MiniExrImageConverter)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/configure.h DESTINATION ${MAGNUM_PLUGINS_INCLUDE_INSTALL_DIR}/MiniExrImageConverter)

if(BUILD_TESTS)
    add_library(MagnumMiniExrImageConverterTestLib STATIC
        $<TARGET_OBJECTS:MiniExrImageConverterObjects>
        ${PROJECT_SOURCE_DIR}/src/dummy.cpp) # XCode workaround, see file comment for details
    target_include_directories(MagnumMiniExrImageConverterTestLib PUBLIC
        ${PROJECT_SOURCE_DIR}/src
        ${PROJECT_BINARY_DIR}/src)
    target_link_libraries(MagnumMiniExrImageConverterTestLib Magnum::Magnum)
    add_subdirectory(Test)
endif()

# MagnumPlugins MiniExrImageConverter target alias for superprojects
add_library(MagnumPlugins::MiniExrImageConverter ALIAS MiniExrImageConverter)
