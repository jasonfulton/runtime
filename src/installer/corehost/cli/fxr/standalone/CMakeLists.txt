# Licensed to the .NET Foundation under one or more agreements.
# The .NET Foundation licenses this file to you under the MIT license.
# See the LICENSE file in the project root for more information.

project(hostfxr)

set(DOTNET_PROJECT_NAME "hostfxr")

# Include directories
include_directories(../../json)
include_directories(../../fxr)

# CMake does not recommend using globbing since it messes with the freshness checks
set(SOURCES
    ./hostpolicy_resolver.cpp
)

set(HEADERS
    ../command_line.h
    ../corehost_init.h
    ../fx_muxer.h
    ../fx_resolver.h
    ../framework_info.h
    ../host_context.h
    ../sdk_info.h
    ../sdk_resolver.h
    ../hostpolicy_resolver.h
)

if(CLR_CMAKE_TARGET_WIN32)
    list(APPEND SOURCES
        hostfxr.def)
else(CLR_CMAKE_TARGET_WIN32)
    set(DEF_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/hostfxr_unixexports.src)
    set(EXPORTS_FILE ${CMAKE_CURRENT_BINARY_DIR}/hostfxr.exports)
    generate_exports_file(${DEF_SOURCES} ${EXPORTS_FILE})

    set_exports_linker_option(${EXPORTS_FILE})
endif(CLR_CMAKE_TARGET_WIN32)

include(../../lib.cmake)

if(CLR_CMAKE_HOST_UNIX)
    add_custom_target(hostfxr_exports DEPENDS ${EXPORTS_FILE})
    add_dependencies(hostfxr hostfxr_exports)

    set_property(TARGET hostfxr APPEND_STRING PROPERTY LINK_FLAGS ${EXPORTS_LINKER_OPTION})
    set_property(TARGET hostfxr APPEND_STRING PROPERTY LINK_DEPENDS ${EXPORTS_FILE})
endif(CLR_CMAKE_HOST_UNIX)

install_with_stripped_symbols(hostfxr TARGETS corehost)
target_link_libraries(hostfxr libhostcommon libhostfxr_static)
