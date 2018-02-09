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


#################################################
# INCLUDE MODULES AND SETTINGS
#################################################

set(VERSION "1.0.0")

set(prefix 					${CMAKE_INSTALL_PREFIX})
set(exec_prefix 			${prefix})
set(libdir 					${exec_prefix}/lib)
set(includedir 				${prefix}/include/usrsctp)
set(CMAKE_REQUIRED_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_MACOSX_RPATH 		1)

add_definitions(-DZMDNET_USER_SPACE)
add_definitions(-DZMDNET_${CMAKE_SYSTEM_NAME})
add_definitions(-DZMDNET_SIMPLE_ALLOCATOR)
add_definitions(-DZMDNET_PROCESS_LEVEL_LOCKS)


#################################################
# OS DEPENDENT
#################################################

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	add_definitions(-D_GNU_SOURCE)
	option(_GNU_SOURCE "_GNU_SOURCE" 1)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}     -Wno-implicit-function-declaration")
endif ()
# -Werror=implicit-function-declaration
if (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
	add_definitions(-U__FreeBSD__)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}     -Wno-address-of-packed-member")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-address-of-packed-member")
endif ()

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	add_definitions(-U__APPLE__)
	add_definitions(-D__APPLE_USE_RFC_2292)
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}     -Wno-address-of-packed-member -Wno-deprecated-declarations")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-address-of-packed-member -Wno-deprecated-declarations")
endif ()

if (${CMAKE_SYSTEM_NAME} MATCHES "DragonFly")
	add_definitions(-U__DragonFly__)
endif ()

if (${CMAKE_SYSTEM_NAME} MATCHES "NetBSD")
	add_definitions(-U__NetBSD__)
endif ()

if (${CMAKE_SYSTEM_NAME} MATCHES "OpenBSD")
	add_definitions(-U__OpenBSD__)
endif ()


#################################################
# LIBRARY FILES
#################################################

include_directories(${CMAKE_CURRENT_SOURCE_DIR})


list(APPEND zmdnet_headers
)

list(APPEND zmdnet_sources
)

add_library(zmdnet SHARED ${zmdnet_sources} ${zmdnet_headers})
add_library(zmdnet-static STATIC ${zmdnet_sources} ${zmdnet_headers})

if (WIN32)
	target_link_libraries(zmdnet ws2_32)
	target_link_libraries(zmdnet-static ws2_32)
	message(STATUS "link library: ws2_32 done")
	target_link_libraries(zmdnet  PUBLIC "${CMAKE_SOURCE_DIR}/thirdparty/libs/sodiumlib/lib/v141/dynamic/libsodium.lib")
    target_link_libraries(zmdnet-static PUBLIC "${CMAKE_SOURCE_DIR}/thirdparty/libs/sodiumlib/lib/v141/static/libsodium.lib")
else()
    target_link_libraries(zmdnet  PUBLIC "${CMAKE_SOURCE_DIR}/thirdparty/libs/sodiumlib/lib/libsodium.so")
    target_link_libraries(zmdnet-static PUBLIC "${CMAKE_SOURCE_DIR}/thirdparty/libs/sodiumlib/lib/libsodium.a")
endif ()

set_target_properties(zmdnet-static PROPERTIES OUTPUT_NAME "zmdnet")
set_target_properties(zmdnet PROPERTIES IMPORT_SUFFIX "_import.lib")
set_target_properties(zmdnet PROPERTIES SOVERSION 1 VERSION 1.0.0)

if (NOT DEFINED CMAKE_INSTALL_LIBDIR)
	SET(CMAKE_INSTALL_LIBDIR lib)
endif ()

#################################################
# INSTALL LIBRARY AND HEADER
#################################################

install(TARGETS zmdnet zmdnet-static DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES zmdnet.h DESTINATION include)
