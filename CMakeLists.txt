cmake_minimum_required(VERSION 2.6)
project(lib43)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
link_directories(${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
include_directories(src)

enable_language(ASM)
set(CMAKE_C_FLAGS "-std=c99 -Os -ffreestanding -fno-stack-protector -nostdlib")

# OS detection
if (NOT DEFINED OS)
    if (APPLE)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -static")
        set(CMAKE_SKIP_RPATH TRUE)
        set(OS "osx")
    elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
        set(OS "linux")
    elseif (CMAKE_SYSTEM_NAME MATCHES "NetBSD")
        set(OS "netbsd")
    else()
        message(FATAL_ERROR "Unsupported OS: ${CMAKE_SYSTEM_NAME}")
    endif()
endif()

# CPU detection
if (NOT DEFINED CPU)
    if (CMAKE_SYSTEM_PROCESSOR MATCHES "^i[36]86$")
        set(CPU "i386")
    elseif (CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64)$")
        set(CPU "x86_64")
    elseif (CMAKE_SYSTEM_PROCESSOR MATCHES "^mips")
        set(CPU "mips")
    elseif (CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
        set(CPU "arm")
    else()
        message(FATAL_ERROR "Unsupported processor: ${CMAKE_SYSTEM_PROCESSOR}")
    endif()
endif()

if (CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64)" AND CPU MATCHES "i386")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
    set(CMAKE_ASM_FLAGS "${CMAKE_ASM_FLAGS} -m32")
endif()

file(GLOB_RECURSE LIB43_SOURCE src/*.c)
file(GLOB_RECURSE LIB43_PLATFORM_SOURCE arch/${CPU}/${OS}/*.s arch/${CPU}/${OS}/*.c)
file(GLOB LIB43_ARCH_SOURCE arch/${CPU}/*.s arch/${CPU}/*.c)
add_library(43 STATIC ${LIB43_SOURCE} ${LIB43_PLATFORM_SOURCE} ${LIB43_ARCH_SOURCE})

add_executable(test1 example/test.c)
target_link_libraries(test1 43)
set_target_properties(test1 PROPERTIES OUTPUT_NAME "test")
