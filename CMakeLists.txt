cmake_minimum_required(VERSION 3.18)
project(compact25519 C)

cmake_policy(SET CMP0048 NEW)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS ON)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(GNUInstallDirs)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_C_FLAGS         "-Wall -Wextra -Wno-unused-parameter -pipe -fstack-protector-all")
set(CMAKE_C_FLAGS_DEBUG   "-O0 -g -ggdb -gdwarf-2 -gstrict-dwarf -fno-omit-frame-pointer -fno-inline")
set(CMAKE_C_FLAGS_RELEASE "-O2")

option(COMPACT25519_BUILD_LIBRARY_SHARED "Build shared library"   ON)
option(COMPACT25519_BUILD_LIBRARY_STATIC "Build static library"   OFF)
option(COMPACT25519_BUILD_TEST           "Build test application" ON)

set(COMPACT25519_SOURCE_FILES
	src/compact_ed25519.c
	src/compact_wipe.c
	src/compact_x25519.c
	src/c25519/c25519.c
	src/c25519/ed25519.c
	src/c25519/edsign.c
	src/c25519/f25519.c
	src/c25519/fprime.c
	src/c25519/sha512.c
)

MESSAGE(STATUS "Build type:                    " ${CMAKE_BUILD_TYPE})
MESSAGE(STATUS "Build shared library:          " ${COMPACT25519_BUILD_LIBRARY_SHARED})
MESSAGE(STATUS "Build static library:          " ${COMPACT25519_BUILD_LIBRARY_STATIC})
MESSAGE(STATUS "Build test app:                " ${COMPACT25519_BUILD_TEST})

add_library(compact25519 OBJECT ${COMPACT25519_SOURCE_FILES})
set_target_properties(compact25519 PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_include_directories(compact25519 PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include/compact25519")

# Shared lib
if (COMPACT25519_BUILD_LIBRARY_SHARED)
	add_library(compact25519_shared SHARED $<TARGET_OBJECTS:compact25519>)
	set_target_properties(compact25519_shared PROPERTIES OUTPUT_NAME compact25519)
	install(TARGETS compact25519_shared LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

# Static lib
if (COMPACT25519_BUILD_LIBRARY_STATIC)
	add_library(compact25519_static STATIC $<TARGET_OBJECTS:compact25519>)
	set_target_properties(compact25519_static PROPERTIES OUTPUT_NAME compact25519)
	install(TARGETS compact25519_static LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif()

if (COMPACT25519_BUILD_TEST)
	add_executable(compact25519_bin test/run-all.c)
	target_link_libraries(compact25519_bin compact25519)
	set_target_properties(compact25519_bin PROPERTIES OUTPUT_NAME compact25519test)
endif()
