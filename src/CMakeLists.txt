# global

include_directories(
    ${OPENGL_INCLUDE_DIR}
    ${PNG_INCLUDE_DIRS}
    ${ZLIB_INCLUDE_DIRS})

include_directories(
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/built"
    "${CMAKE_CURRENT_BINARY_DIR}")

if(SPACEWARE_FOUND)
    include_directories(
        "${SPACEWARE_INCLUDE_DIR}")
endif()

set(HAVE_FLTK ${FLTK_FOUND})
set(HAVE_SPACEWARE ${SPACEWARE_FOUND})
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
               "${CMAKE_CURRENT_BINARY_DIR}/config.h")

# platform utilities

if(WIN32)
    set(util_SOURCES
        win32/w32util.cpp)
else()
    set(util_SOURCES
        unix/unixutil.cpp)
endif()

# libslvs

set(libslvs_SOURCES
    util.cpp
    entity.cpp
    expr.cpp
    constraint.cpp
    constrainteq.cpp
    system.cpp)

set(libslvs_HEADERS
    solvespace.h)

    add_library(slvs SHARED
    ${libslvs_SOURCES}
    ${libslvs_HEADERS}
    ${util_SOURCES}
    lib.cpp)

target_compile_definitions(slvs
    PRIVATE -DLIBRARY)

target_include_directories(slvs
    PUBLIC "${CMAKE_SOURCE_DIR}/include")

set_property(TARGET slvs PROPERTY
    PUBLIC_HEADER "${CMAKE_SOURCE_DIR}/include/slvs.h")

if(NOT WIN32)
    install(TARGETS slvs
        LIBRARY DESTINATION lib
        PUBLIC_HEADER DESTINATION include)
endif()

# generated files

file(GLOB icons "${CMAKE_CURRENT_SOURCE_DIR}/icons/*.png")

if(PERL_FOUND AND PERLMODULES_FOUND)
    add_custom_command(
        OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/built/icons.h"
               "${CMAKE_CURRENT_SOURCE_DIR}/built/icons-proto.h"
        COMMAND "${PERL_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/png2c.pl"
                "${CMAKE_CURRENT_SOURCE_DIR}/built/icons.h"
                "${CMAKE_CURRENT_SOURCE_DIR}/built/icons-proto.h"
                "${CMAKE_CURRENT_SOURCE_DIR}"
        MAIN_DEPENDENCY "${CMAKE_CURRENT_SOURCE_DIR}/png2c.pl"
        DEPENDENCIES ${icons})

    add_custom_command(
        OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/built/bitmapextra.table.h"
        COMMAND "${PERL_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/pngchar2c.pl"
                "${CMAKE_CURRENT_SOURCE_DIR}/built/bitmapextra.table.h"
                "${CMAKE_CURRENT_SOURCE_DIR}"
        MAIN_DEPENDENCY "${CMAKE_CURRENT_SOURCE_DIR}/pngchar2c.pl"
        DEPENDENCIES ${icons})
endif()

if(WIN32)
    add_custom_command(
        OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/built/bitmapfont.table.h"
        COMMAND ttf2c "${CMAKE_CURRENT_SOURCE_DIR}/built/bitmapfont.table.h")
endif()

set(generated_HEADERS
    built/bitmapextra.table.h
    built/bitmapfont.table.h
    built/icons-proto.h
    built/icons.h)

set_source_files_properties(${generated_HEADERS}
    PROPERTIES GENERATED TRUE)

# platform dependencies

if(WIN32)
    set(platform_HEADERS
        win32/freeze.h)

    set(platform_SOURCES
        win32/freeze.cpp
        win32/w32main.cpp
        win32/resource.rc)
else()
    include_directories(
        "${FLTK_INCLUDE_DIR}")

    set(platform_SOURCES
        fltk/fltkmain.cpp)

    set(platform_LIBS
        ${CMAKE_DL_LIBS}
        "${FLTK_LIBRARIES}")
endif()

# solvespace executable

set(solvespace_HEADERS
    config.h
    dsc.h
    expr.h
    font.table.h
    polygon.h
    sketch.h
    solvespace.h
    ui.h
    srf/surface.h)

set(solvespace_SOURCES
    bsp.cpp
    clipboard.cpp
    confscreen.cpp
    constraint.cpp
    constrainteq.cpp
    describescreen.cpp
    draw.cpp
    drawconstraint.cpp
    drawentity.cpp
    entity.cpp
    export.cpp
    exportstep.cpp
    exportvector.cpp
    expr.cpp
    file.cpp
    generate.cpp
    glhelper.cpp
    graphicswin.cpp
    group.cpp
    groupmesh.cpp
    mesh.cpp
    modify.cpp
    mouse.cpp
    polygon.cpp
    request.cpp
    solvespace.cpp
    style.cpp
    system.cpp
    textscreens.cpp
    textwin.cpp
    toolbar.cpp
    ttf.cpp
    undoredo.cpp
    util.cpp
    view.cpp
    srf/boolean.cpp
    srf/curve.cpp
    srf/merge.cpp
    srf/ratpoly.cpp
    srf/raycast.cpp
    srf/surface.cpp
    srf/surfinter.cpp
    srf/triangulate.cpp)

add_executable(solvespace WIN32
    ${libslvs_HEADERS}
    ${libslvs_SOURCES}
    ${util_SOURCES}
    ${platform_HEADERS}
    ${platform_SOURCES}
    ${generated_HEADERS}
    ${solvespace_HEADERS}
    ${solvespace_SOURCES})

target_link_libraries(solvespace
    "${OPENGL_LIBRARIES}"
    "${PNG_LIBRARIES}"
    "${ZLIB_LIBRARIES}"
    "${platform_LIBS}")

if(WIN32)
    set_target_properties(solvespace PROPERTIES LINK_FLAGS /MANIFEST:NO)
endif()

if(SPACEWARE_FOUND)
    target_link_libraries(solvespace
        "${SPACEWARE_LIBRARIES}")
endif()

install(TARGETS solvespace
    RUNTIME DESTINATION bin)

# valgrind

add_custom_target(solvespace-valgrind
    valgrind
        --tool=memcheck
        --verbose
        --track-fds=yes
        --log-file=vg.%p.out
        --num-callers=50
        --error-limit=no
        --read-var-info=yes
        --leak-check=full
        --leak-resolution=high
        --show-reachable=yes
        --track-origins=yes
        --malloc-fill=0xac
        --free-fill=0xde
        $<TARGET_FILE:solvespace>)
