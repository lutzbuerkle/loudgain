# loudgain CMakeLists.txt
# Copyright (C) 2014 Alessandro Ghedini <alessandro@ghedini.me>
# Modifications Copyright (C) 2019 Matthias C. Hormann <mhormann@gmx.de>
# This file is released under the 2 clause BSD license, see COPYING

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT(loudgain C CXX)

SET(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/" ${CMAKE_MODULE_PATH})

SET(VERSION_MAJOR 0)
SET(VERSION_MINOR 6)
SET(VERSION_PATCH 8)

SET(loudgain_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})

SET(INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})

INCLUDE(FindPkgConfig)

IF (APPLE)
  # Fix linking on 10.14+. See https://stackoverflow.com/questions/54068035
  LINK_DIRECTORIES(/usr/local/lib)
ENDIF (APPLE)


PKG_CHECK_MODULES(LAVC libavcodec REQUIRED)
PKG_CHECK_MODULES(LAVF libavformat REQUIRED)
PKG_CHECK_MODULES(LAVR libswresample REQUIRED)
PKG_CHECK_MODULES(LAVU libavutil REQUIRED)
PKG_CHECK_MODULES(LTAG taglib REQUIRED)

FIND_PACKAGE(EBUR128)

IF (NOT EBUR128_FOUND)
  MESSAGE(FATAL_ERROR "libebur128 not found.")
ENDIF (NOT EBUR128_FOUND)

FILE(GLOB SOURCES RELATIVE ${CMAKE_SOURCE_DIR} "src/*.cc" "src/*.c")

ADD_EXECUTABLE(loudgain ${SOURCES})

INCLUDE_DIRECTORIES(
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${EBUR128_INCLUDE_DIR}
  ${LAVC_INCLUDE_DIRS}
  ${LAVF_INCLUDE_DIRS}
  ${LAVR_INCLUDE_DIRS}
  ${LAVU_INCLUDE_DIRS}
  ${LTAG_INCLUDE_DIRS}
  ${CMAKE_CURRENT_BINARY_DIR}
)

# Mac OS X has no pty.h
INCLUDE (CheckIncludeFiles)
CHECK_INCLUDE_FILES (pty.h HAVE_PTY_H)

CONFIGURE_FILE("config.h.in" "config.h")

SET(LIBS )

TARGET_LINK_LIBRARIES(loudgain
  ${EBUR128_LIBRARY}
  ${LAVC_LIBRARIES}
  ${LAVF_LIBRARIES}
  ${LAVR_LIBRARIES}
  ${LAVU_LIBRARIES}
  ${LTAG_LIBRARIES}
)

SET_TARGET_PROPERTIES(loudgain PROPERTIES
  COMPILE_FLAGS "-Wall -pedantic -g"
)

SET(CMAKE_C_FLAGS "-std=gnu99 -D_GNU_SOURCE")

SET(CMAKE_CXX_FLAGS "-std=gnu++11 -D_GNU_SOURCE")

INSTALL(TARGETS loudgain DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

INSTALL(FILES
  ${PROJECT_SOURCE_DIR}/docs/loudgain.1
  DESTINATION ${CMAKE_INSTALL_PREFIX}/share/man/man1
)
INSTALL(PROGRAMS
  ${PROJECT_SOURCE_DIR}/bin/rgbpm
  DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
)

# provide 'uninstall' target
if(NOT TARGET uninstall)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)

    add_custom_target(uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif()
