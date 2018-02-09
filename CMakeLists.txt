#
# Copyright (C) 2015-2017 mengdi zhang
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
# 3. Neither the name of the project nor the names of its contributors
#    may be used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE PROJECT AND CONTRIBUTORS ``AS IS'' AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE PROJECT OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
# OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
# OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
# SUCH DAMAGE.
#

cmake_minimum_required(VERSION 3.0)
# Ignore warnings about rpath behavior on OS X
cmake_policy(SET CMP0042 NEW)
project(zmdsim)


option(ENABLE_PLUGINS "Enable Official Plugins" ON)
option(ENABLE_LC3EDIT "Build lc3edit" OFF)
option(ENABLE_COMP "Build comp" OFF)
option(ENABLE_COMPLX "Build complx" ON)
option(ENABLE_LC3TEST "Build lc3test" ON)
option(ENABLE_DEV "Build dev version" OFF)
option(ENABLE_TESTS "Enable testing" OFF)
option(ENABLE_COVERAGE "Enable testing coverage" OFF)
option(ENABLE_PYLC3 "Enable python bindings to liblc3" OFF)


if(ENABLE_TESTS)
    enable_testing()
endif(ENABLE_TESTS)


# Additional debug flags
set(CMAKE_C_FLAGS_DEBUG "-g -pg")
# Install directories
set(CMAKE_INSTALL_LIBDIR lib CACHE STRING "where to install libraries/plugins")
set(CMAKE_INSTALL_BINDIR bin CACHE STRING "where to install programs")


# Add code coverage stuff.
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)
if(ENABLE_COVERAGE)
    include(CodeCoverage)
    append_coverage_compiler_flags()
endif(ENABLE_COVERAGE)


#################################################
# COMPILER SETTINGS
#################################################
# SETTINGS FOR UNIX COMPILER
if ("${CMAKE_C_COMPILER_ID}" MATCHES "Clang" OR "${CMAKE_C_COMPILER_ID}" MATCHES "AppleClang" OR "${CMAKE_C_COMPILER_ID}" MATCHES "GNU")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -std=c99 -pedantic -Wall -Wextra")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c99 -pedantic -Wall -Wextra")

    check_c_compiler_flag(-Wfloat-equal has_wfloat_equal)
    if (has_wfloat_equal)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wfloat-equal")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wfloat-equal")
    endif ()

    check_c_compiler_flag(-Wshadow has_wshadow)
    if (has_wshadow)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wshadow")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wshadow")
    endif ()

    check_c_compiler_flag(-Wpointer-arith has_wpointer_aritih)
    if (has_wpointer_aritih)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wpointer-arith")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wpointer-arith")
    endif ()

    check_c_compiler_flag(-Wunreachable-code has_wunreachable_code)
    if (has_wunreachable_code)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wunreachable-code")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wunreachable-code")
    endif ()

    check_c_compiler_flag(-Winit-self has_winit_self)
    if (has_winit_self)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Winit-self")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Winit-self")
    endif ()

    check_c_compiler_flag(-Wno-unused-function has_wno_unused_function)
    if (has_wno_unused_function)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wno-unused-function")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-function")
    endif ()

    check_c_compiler_flag(-Wno-unused-parameter has_wno_unused_parameter)
    if (has_wno_unused_parameter)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wno-unused-parameter")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-parameter")
    endif ()

    check_c_compiler_flag(-Wno-unreachable-code has_wno_unreachable_code)
    if (has_wno_unreachable_code)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Wno-unreachable-code")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unreachable-code")
    endif ()

    if (zmdnet_werror)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -Werror")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
    endif ()

    if (zmdnet_sanitizer_address)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -fsanitize=address -fno-omit-frame-pointer -g")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer -g")
    endif ()

    if (zmdnet_sanitizer_memory)
        set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -fsanitize=memory -fno-omit-frame-pointer -g -fsanitize-memory-track-origins")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=memory -fno-omit-frame-pointer -g -fsanitize-memory-track-origins")
    endif ()
endif ()


# SETTINGS FOR VISUAL STUDIO COMPILER
if ("x${CMAKE_C_COMPILER_ID}" STREQUAL "xMSVC")
    if (CMAKE_C_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W3" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
    else ()
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W3")
    endif ()

    if (WERROR)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /WX")
    endif ()
endif ()


# Default to release build if not specified
if(NOT CMAKE_BUILD_TYPE)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif(NOT CMAKE_BUILD_TYPE)


if(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_EXE_LINKER_FLAGS "-fprofile-arcs -ftest-coverage")
endif(CMAKE_BUILD_TYPE MATCHES "Debug")


add_definitions(-DPREFIX=${CMAKE_INSTALL_PREFIX})
add_definitions(-DSO_SUFFIX="${CMAKE_SHARED_LIBRARY_SUFFIX}")


if(ENABLE_COMPLX OR ENABLE_LC3TEST OR ENABLE_TESTS)
    set(wxWidgets_USE_LIBS stc xrc xml html adv net core base)
    find_package(wxWidgets REQUIRED)
    set(CMAKE_SKIP_RPATH TRUE)
    include(${wxWidgets_USE_FILE})
endif(ENABLE_COMPLX OR ENABLE_LC3TEST OR ENABLE_TESTS)


if(CMAKE_HOST_WIN32)
    # Needed for htons
    set(LIBLC3_EXTRA_LIBS ws2_32)
    # Windows doesn't set include directories correctly?
    include_directories("C:/msys/1.0/local/lib/wx/include/msw-unicode-3.0")
    include_directories("C:/msys/1.0/local/include/wx-3.0")
endif(CMAKE_HOST_WIN32)


include_directories(${zmdsim_SOURCE_DIR}/lc3test)
include_directories(${zmdsim_SOURCE_DIR}/liblc3)
include_directories(${zmdsim_SOURCE_DIR}/zlog/src)


# set(SRC_LIBLC3
#     liblc3/ExpressionEvaluator.cpp
#     liblc3/lc3_assemble.cpp
#     liblc3/lc3.cpp
#     liblc3/lc3_debug.cpp
#     liblc3/lc3_execute.cpp
#     liblc3/lc3_expressions.cpp
#     liblc3/lc3_os.cpp
#     liblc3/lc3_params.cpp
#     liblc3/lc3_parser.cpp
#     liblc3/lc3_plugin.cpp
#     liblc3/lc3_runner.cpp
#     liblc3/lc3_symbol.cpp
#     liblc3/lc3_test.cpp
#     liblc3/lc3_cmp.cpp
# )
# add_library(lc3 SHARED ${SRC_LIBLC3})
# target_link_libraries(lc3 dl ${LIBLC3_EXTRA_LIBS})
# install(TARGETS lc3 LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR})

# set(SRC_LOGGING logging/logger.cpp)
# add_library(logger ${SRC_LOGGING})

# set(SRC_AS2OBJ as2obj/main.c)
# add_executable(as2obj ${SRC_AS2OBJ})
# target_link_libraries(as2obj lc3)
# install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/as2obj DESTINATION ${CMAKE_INSTALL_BINDIR})

# set(SRC_LC3RUNNER lc3runner/main.cpp)
# add_executable(lc3runner ${SRC_LC3RUNNER})
# target_link_libraries(lc3runner lc3)
# install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/lc3runner DESTINATION ${CMAKE_INSTALL_BINDIR})


add_subdirectory(zlog)
# if(ENABLE_PLUGINS)
#     add_subdirectory(plugins)
# endif(ENABLE_PLUGINS)
# if(ENABLE_LC3EDIT)
#     add_subdirectory(lc3edit)
# endif(ENABLE_LC3EDIT)
# if(ENABLE_COMP)
#     add_subdirectory(comp)
# endif(ENABLE_COMP)
# if(ENABLE_COMPLX)
#     add_subdirectory(complx)
# endif(ENABLE_COMPLX)
# if(ENABLE_LC3TEST)
#     add_subdirectory(lc3test)
# endif(ENABLE_LC3TEST)
# if(ENABLE_TESTS)
#     add_subdirectory(tests)
# endif(ENABLE_TESTS)
# if (ENABLE_PYLC3)
#     add_subdirectory(pylc3)
# endif(ENABLE_PYLC3)


# uninstall target
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)
add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
