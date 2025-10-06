cmake_minimum_required(VERSION 3.11)

project(wren-lang
        VERSION 0.4.0
        DESCRIPTION "The Wren Programming Language. Wren is a small, fast, class-based concurrent scripting language."
        HOMEPAGE_URL "https://wren.io"
        LANGUAGES C
)

option(WREN_SHARED "Build the library as a Shared Library" OFF)
option(WREN_EXAMPLE "Build the main embedding example" OFF)
option(WREN_OPT_OUT_META "Don't build the optional meta programing library" OFF)
option(WREN_OPT_OUT_RAND "Don't build the optional random library" OFF)

if (WREN_OPT_OUT_META AND WREN_OPT_OUT_RAND)
    set(WREN_OPTIONAL OFF)
else ()
    set(WREN_OPTIONAL ON)
endif ()

set(WREN_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../..)

file(GLOB WREN_SOURCES "${WREN_ROOT_DIR}/src/vm/*.c")

if (WREN_OPTIONAL)
    file(GLOB WREN_OPTIONAL "${WREN_ROOT_DIR}/src/optional/*.c")
    list(APPEND WREN_SOURCES ${WREN_OPTIONAL})
endif ()

if (WREN_SHARED)
    add_library(wren SHARED ${WREN_SOURCES})
else ()
    add_library(wren STATIC ${WREN_SOURCES})
endif ()

target_include_directories(wren PUBLIC ${WREN_ROOT_DIR}/src/include)
target_include_directories(wren PRIVATE ${WREN_ROOT_DIR}/src/vm)
target_include_directories(wren PRIVATE ${WREN_ROOT_DIR}/src/optional)

if (WREN_OPT_OUT_META)
    target_compile_definitions(wren PUBLIC WREN_OPT_META=0)
else ()
    target_compile_definitions(wren PUBLIC WREN_OPT_META=1)
endif ()

if (WREN_OPT_OUT_RAND)
    target_compile_definitions(wren PUBLIC WREN_OPT_RANDOM=0)
else ()
    target_compile_definitions(wren PUBLIC WREN_OPT_RANDOM=1)
endif ()

add_library(wren-lang:wren ALIAS wren)

if (WREN_EXAMPLE)
    add_executable(wren-embedding ${WREN_ROOT_DIR}/example/embedding/main.c)
    target_link_libraries(wren-embedding PUBLIC wren-lang:wren)
endif ()