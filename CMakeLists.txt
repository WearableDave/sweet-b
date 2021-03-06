#
# CMakeLists.txt: abandon all hope, etc etc etc
#
# This file is part of Sweet B, a safe, compact, embeddable elliptic curve
# cryptography library.
#
# Sweet B is provided under the terms of the included LICENSE file. All
# other rights are reserved.
#
# Copyright 2017 Wearable Inc.
#

cmake_minimum_required(VERSION 3.5)

project(sweet_b LANGUAGES C VERSION 0.1.0)
include(GNUInstallDirs)

set(CMAKE_C_STANDARD 11)

if("${CMAKE_C_COMPILER_ID}" STREQUAL "Clang" OR
        "${CMAKE_C_COMPILER_ID}" STREQUAL "AppleClang")
    message(STATUS "Enabling clang options")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Weverything -Wno-padded")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -fsanitize=undefined -fsanitize=address")
endif()
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_POSIX_C_SOURCE=200112L -Wall -pedantic -Wextra")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -Werror")

include_directories(src)

set(SB_PUBLIC_HEADERS
        src/sb_sw_lib.h
        src/sb_mont_lib.h
        src/sb_fe.h
        src/sb_hmac_drbg.h
        src/sb_hmac_sha256.h
        src/sb_sha256.h
        src/sb_sw_context.h
        src/sb_mont_context.h
        src/sb_types.h
        src/sb_error.h)

set(SB_PRIVATE_HEADERS
        src/sb_sw_curves.h
        src/sb_mont_curves.h
        src/sb_test.h
        src/sb_test_list.h)

set(SB_SOURCES
        ${SB_PUBLIC_HEADERS}
        ${SB_PRIVATE_HEADERS}

        src/sb_sha256.c
        src/sb_hmac_sha256.c
        src/sb_hmac_drbg.c

        src/sb_fe.c
        src/sb_sw_lib.c
        src/sb_mont_lib.c)

set_property(SOURCE src/sb_test.c APPEND PROPERTY OBJECT_DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/src/sb_test_list.h")

add_executable(sb_test ${SB_SOURCES} src/sb_test.c src/sb_test_cavp.c)

# 16x16->32 operations are most likely to accidentally trigger UB if there is
# a missing cast from implicitly promoted int to uint32_t

# If you update the SB_HMAC_DRBG options here to make unit tests pass, you
# must also update the checks in sb_hmac_drbg.h

target_compile_definitions(sb_test PRIVATE SB_TEST SB_MUL_SIZE=2
        SB_DEBUG_ASSERTS
        SB_UNROLL=3
        SB_HMAC_DRBG_RESEED_INTERVAL=8
        SB_HMAC_DRBG_MAX_BYTES_PER_REQUEST=128
        SB_HMAC_DRBG_MAX_ENTROPY_INPUT_LENGTH=128)

add_library(sweet_b SHARED ${SB_SOURCES})

set_target_properties(sweet_b PROPERTIES
        VERSION ${PROJECT_VERSION}
        PUBLIC_HEADER "${SB_PUBLIC_HEADERS}")

install(TARGETS sweet_b
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
