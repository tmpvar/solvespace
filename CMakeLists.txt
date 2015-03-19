# cmake configuration

cmake_minimum_required(VERSION 3.0)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH}
    "${CMAKE_SOURCE_DIR}/cmake/")

include(CheckIncludeFile)

include_directories(
  ${CMAKE_SOURCE_DIR}/src
  ${CMAKE_SOURCE_DIR}/include
  ${CMAKE_SOURCE_DIR}/exposed
  ${CMAKE_BINARY_DIR}
)

# for /MT on MSVC
set(CMAKE_USER_MAKE_RULES_OVERRIDE
   "${CMAKE_SOURCE_DIR}/cmake/c_flag_overrides.cmake")
set(CMAKE_USER_MAKE_RULES_OVERRIDE_CXX
   "${CMAKE_SOURCE_DIR}/cmake/cxx_flag_overrides.cmake")

# project

project(solvespace)
set(solvespace_VERSION_MAJOR 2)
set(solvespace_VERSION_MINOR 1)

# compiler

if(WIN32)
    add_definitions(
        -D_CRT_SECURE_NO_DEPRECATE=1
        -D_CRT_SECURE_NO_WARNINGS=1
        -D_WIN32_WINNT=0x500
        -D_WIN32_IE=_WIN32_WINNT
        -DISOLATION_AWARE_ENABLED=1
        -DWIN32=1
        -DWIN32_LEAN_AND_MEAN=1)
endif()

# dependencies

CHECK_INCLUDE_FILE("stdint.h" HAVE_STDINT_H)

find_package(OpenGL REQUIRED)

find_package(Perl)
find_package(PerlModules COMPONENTS GD)
if(NOT (PERL_FOUND AND PERLMODULES_FOUND))
    message(STATUS "Perl with GD not found; icons will not be regenerated if modified")
endif()

if(WIN32)
    find_package(PNG)

    if(NOT PNG_FOUND)
        message(STATUS "Using prebuilt libpng")

        set(PNG_FOUND TRUE)
        set(PNG_LIBRARIES
            "${CMAKE_SOURCE_DIR}/extlib/libpng/libpng.lib")
        set(PNG_INCLUDE_DIRS
            "${CMAKE_SOURCE_DIR}/extlib/libpng")
    endif()

    find_package(ZLIB)

    if(NOT ZLIB_FOUND)
        message(STATUS "Using prebuilt zlib")

        set(ZLIB_FOUND TRUE)
        set(ZLIB_LIBRARIES
            "${CMAKE_SOURCE_DIR}/extlib/zlib/zlib.lib")
        set(ZLIB_INCLUDE_DIRS
            "${CMAKE_SOURCE_DIR}/extlib/zlib")
    endif()

    message(STATUS "Using prebuild SpaceWare")
    set(SPACEWARE_FOUND TRUE)
    set(SPACEWARE_INCLUDE_DIR
        "${CMAKE_SOURCE_DIR}/extlib/si")
    set(SPACEWARE_LIBRARIES
        "${CMAKE_SOURCE_DIR}/extlib/si/siapp.lib")
else()
    find_package(PNG REQUIRED)
    find_package(ZLIB REQUIRED)
    find_package(SpaceWare)

    find_package(FLTK REQUIRED)
    CHECK_INCLUDE_FILE("fontconfig/fontconfig.h" HAVE_FONTCONFIG)
endif()

# components

if(WIN32)
    add_subdirectory(tools)
endif()

add_subdirectory(src)

add_library(
    ss
    src/bsp.cpp
    src/clipboard.cpp
#    src/confscreen.cpp
    src/constraint.cpp
    src/constrainteq.cpp
#    src/describescreen.cpp
#    src/draw.cpp
#    src/drawconstraint.cpp
#    src/drawentity.cpp
    src/entity.cpp
    src/export.cpp
    src/exportstep.cpp
    src/exportvector.cpp
    src/expr.cpp
    src/file.cpp
    src/generate.cpp
    src/group.cpp
    src/groupmesh.cpp
    src/lib.cpp
    src/mesh.cpp
    src/modify.cpp
    src/polygon.cpp
    src/request.cpp
#    src/solvespace.cpp
#    src/style.cpp
#    src/system.cpp
#    src/textscreens.cpp
#    src/textwin.cpp
#    src/toolbar.cpp
    src/ttf.cpp
    src/undoredo.cpp
    src/util.cpp
)

