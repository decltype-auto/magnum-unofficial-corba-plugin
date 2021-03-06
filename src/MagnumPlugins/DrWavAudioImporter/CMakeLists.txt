#
#   This file is part of Magnum.
#
#   Copyright © 2010, 2011, 2012, 2013, 2014, 2015, 2016
#             Vladimír Vondruš <mosra@centrum.cz>
#   Copyright © 2016 Alice Margatroid <loveoverwhelming@gmail.com>
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

find_package(Magnum REQUIRED Audio)

if(BUILD_STATIC)
    set(MAGNUM_DRWAVAUDIOIMPORTER_BUILD_STATIC 1)
endif()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/configure.h.cmake
               ${CMAKE_CURRENT_BINARY_DIR}/configure.h)

set(DrWavAudioImporter_SRCS
    DrWavImporter.cpp)

set(DrWavAudioImporter_HEADERS
    DrWavImporter.h)

# Objects shared between plugin and test library
add_library(DrWavAudioImporterObjects OBJECT
    ${DrWavAudioImporter_SRCS}
    ${DrWavAudioImporter_HEADERS})
target_include_directories(DrWavAudioImporterObjects PUBLIC
    $<TARGET_PROPERTY:Magnum::Audio,INTERFACE_INCLUDE_DIRECTORIES>
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_BINARY_DIR}/src)
target_compile_definitions(DrWavAudioImporterObjects PRIVATE "DrWavAudioImporterObjects_EXPORTS")
# Include the drwav_*.h files as a system directory to supress warnings
target_include_directories(DrWavAudioImporterObjects SYSTEM PRIVATE ${PROJECT_SOURCE_DIR}/src/external/dr)
if(NOT BUILD_PLUGINS_STATIC OR BUILD_STATIC_PIC)
    set_target_properties(DrWavAudioImporterObjects PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()

# DrWavImporter plugin
add_plugin(DrWavAudioImporter ${MAGNUM_PLUGINS_AUDIOIMPORTER_DEBUG_INSTALL_DIR} ${MAGNUM_PLUGINS_AUDIOIMPORTER_RELEASE_INSTALL_DIR}
    DrWavAudioImporter.conf
    $<TARGET_OBJECTS:DrWavAudioImporterObjects>
    pluginRegistration.cpp)
if(BUILD_STATIC_PIC)
    set_target_properties(DrWavAudioImporter PROPERTIES POSITION_INDEPENDENT_CODE ON)
endif()
target_include_directories(DrWavAudioImporter PUBLIC
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_BINARY_DIR}/src)
target_link_libraries(DrWavAudioImporter Magnum::Magnum Magnum::Audio)

install(FILES ${DrWavAudioImporter_HEADERS} DESTINATION ${MAGNUM_PLUGINS_INCLUDE_INSTALL_DIR}/DrWavAudioImporter)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/configure.h DESTINATION ${MAGNUM_PLUGINS_INCLUDE_INSTALL_DIR}/DrWavAudioImporter)

if(BUILD_TESTS)
    add_library(MagnumDrWavAudioImporterTestLib STATIC
        $<TARGET_OBJECTS:DrWavAudioImporterObjects>
        ${PROJECT_SOURCE_DIR}/src/dummy.cpp) # XCode workaround, see file comment for details
    target_include_directories(MagnumDrWavAudioImporterTestLib PUBLIC
        ${PROJECT_SOURCE_DIR}/src
        ${PROJECT_BINARY_DIR}/src)
    target_link_libraries(MagnumDrWavAudioImporterTestLib Magnum::Magnum Magnum::Audio)
    add_subdirectory(Test)
endif()

# MagnumPlugins DrWavAudioImporter target alias for superprojects
add_library(MagnumPlugins::DrWavAudioImporter ALIAS DrWavAudioImporter)
