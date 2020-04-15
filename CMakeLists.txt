cmake_minimum_required(VERSION 3.15)
project(quickjs C)

file(READ ${PROJECT_SOURCE_DIR}/VERSION CONFIG_VERSION)
string(STRIP ${CONFIG_VERSION} CONFIG_VERSION)

# Global Linux compiler definitions
if(UNIX AND NOT APPLE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
endif()

# libbf
add_library(bf STATIC "")
target_include_directories(bf PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libbf)
target_sources(bf PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libbf/libbf.c
    ${PROJECT_SOURCE_DIR}/lib/libbf/libbf.h)

# libunicode
add_library(unicode STATIC "")
target_include_directories(unicode PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libunicode)
target_sources(unicode PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode-table.h
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode.c
    ${PROJECT_SOURCE_DIR}/lib/libunicode/libunicode.h)

# libregexp
add_library(regexp STATIC "")
target_include_directories(regexp PUBLIC
    ${PROJECT_SOURCE_DIR}/include
    ${PROJECT_SOURCE_DIR}/lib/libregexp)
target_sources(regexp PRIVATE
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp-opcode.h
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp.c
    ${PROJECT_SOURCE_DIR}/lib/libregexp/libregexp.h)
target_link_libraries(regexp PUBLIC unicode)

# libquickjs
add_library(${PROJECT_NAME} STATIC "")
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
if(MINGW)
    target_compile_options(${PROJECT_NAME} PRIVATE -Wno-deprecated-declarations)
    target_link_options(${PROJECT_NAME} PRIVATE -Wl,-Bstatic -lpthread -static-libgcc)
endif()
target_link_libraries(${PROJECT_NAME} PRIVATE bf regexp unicode)
if(MINGW)
    target_link_libraries(${PROJECT_NAME} PRIVATE pthread)
endif()

# qjs
add_executable(qjs ${PROJECT_SOURCE_DIR}/src/qjs.c)
target_link_libraries(qjs quickjs)
if(UNIX AND NOT APPLE)
    target_link_libraries(qjs dl m)
endif()
