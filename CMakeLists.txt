cmake_minimum_required(VERSION 3.15)
project(libquickjs C)

file(READ ${PROJECT_SOURCE_DIR}/VERSION CONFIG_VERSION)
string(STRIP ${CONFIG_VERSION} CONFIG_VERSION)

# Set runtime statically linked library for MSVC projects
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# Global Linux compiler definitions
if(UNIX AND NOT APPLE)
    set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

# libbf
add_library(libbf STATIC "")
target_include_directories(libbf PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libbf)
target_sources(libbf PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libbf/libbf.c
    ${PROJECT_SOURCE_DIR}/lib/libbf/libbf.h)

# libunicode
add_library(libunicode STATIC "")
target_include_directories(libunicode PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libunicode)
target_sources(libunicode PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode-table.h
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode.c
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode.h)

# libregexp
add_library(libregexp STATIC "")
target_include_directories(libregexp PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libregexp)
target_sources(libregexp PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp-opcode.h
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp.c
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp.h)
target_link_libraries(libregexp PUBLIC libunicode)

# libquickjs
add_library(${PROJECT_NAME} SHARED "")
target_include_directories(${PROJECT_NAME}
    PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libbf
    ${PROJECT_SOURCE_DIR}/lib/libregexp
    ${PROJECT_SOURCE_DIR}/lib/libunicode)
target_sources(${PROJECT_NAME} PRIVATE
    ${PROJECT_SOURCE_DIR}/src/cutils.c
    ${PROJECT_SOURCE_DIR}/src/quickjs-libc.c
    ${PROJECT_SOURCE_DIR}/src/quickjs.c)
target_compile_definitions(${PROJECT_NAME} PUBLIC CONFIG_VERSION=\"${CONFIG_VERSION}\" CONFIG_BIGNUM)
if(WIN32 AND MINGW)
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-deprecated-declarations)
endif()
target_link_libraries(${PROJECT_NAME} PRIVATE libbf libregexp libunicode)
if(WIN32 AND MINGW)
    target_link_libraries(${PROJECT_NAME} PRIVATE pthread)
endif()

# qjs
add_executable(qjs ${PROJECT_SOURCE_DIR}/src/qjs.c)
target_link_libraries(qjs libquickjs)
if(UNIX AND NOT APPLE)
    target_link_libraries(qjs dl m)
endif()
