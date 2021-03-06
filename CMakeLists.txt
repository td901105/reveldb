CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(REVELDB)

SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/CMakeModules)
SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR}/)

INCLUDE(CheckFunctionExists)
INCLUDE(CheckIncludeFiles)
INCLUDE(CheckTypeSize)

CHECK_FUNCTION_EXISTS(alloca  C_ALLOCA)
CHECK_FUNCTION_EXISTS(memcmp  HAVE_MEMCMP)
CHECK_FUNCTION_EXISTS(strndup HAVE_STRNDUP)
CHECK_FUNCTION_EXISTS(strnlen HAVE_STRNLEN)

CHECK_INCLUDE_FILES(alloca.h HAVE_ALLOCA_H)
CHECK_INCLUDE_FILES(strings.h HAVE_STRINGS_H)
CHECK_INCLUDE_FILES(string.h HAVE_STRING_H)
CHECK_INCLUDE_FILES(stdlib.h HAVE_STDLIB_H)
CHECK_INCLUDE_FILES(sys/time.h HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILES(sys/times.h HAVE_SYS_TIMES_H)
CHECK_INCLUDE_FILES(unistd.h HAVE_UNISTD_H)
CHECK_INCLUDE_FILES(memory.h HAVE_MEMORY_H)
CHECK_INCLUDE_FILES(stdarg.h HAVE_STDARG_H)
CHECK_INCLUDE_FILES(sys/tree.h HAVE_SYS_TREE_H)
CHECK_INCLUDE_FILES(sys/queue.h HAVE_SYS_QUEUE_H)
CHECK_INCLUDE_FILES(sys/un.h HAVE_SYS_UN)

CHECK_TYPE_SIZE("int" SIZEOF_INT)
CHECK_TYPE_SIZE("long" SIZEOF_LONG)
CHECK_TYPE_SIZE("short" SIZEOF_SHORT)

if (NOT HAVE_SYS_TREE_H)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/compat/sys/tree.h.in
        ${CMAKE_CURRENT_BINARY_DIR}/compat/sys/tree.h)
endif(NOT HAVE_SYS_TREE_H)

if (NOT HAVE_SYS_QUEUE_H)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/compat/sys/queue.h.in
        ${CMAKE_CURRENT_BINARY_DIR}/compat/sys/queue.h)
endif(NOT HAVE_SYS_QUEUE_H)

if (NOT HAVE_STRNDUP)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNO_STRNDUP")
endif(NOT HAVE_STRNDUP)

if (NOT HAVE_STRNLEN)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNO_STRNLEN")
endif(NOT HAVE_STRNLEN)

if (NOT HAVE_SYS_UN)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNO_SYS_UN")
endif(NOT HAVE_SYS_UN)

OPTION(EVHTTPX_DISABLE_SSL       "Disable ssl support"      OFF)
OPTION(EVHTTPX_DISABLE_EVTHR     "Disable evthread support" OFF)
OPTION(EVHTTPX_USE_DEFER_ACCEPT  "Enable TCP_DEFER_ACCEPT"  OFF) 

if (EVHTTPX_USE_DEFER_ACCEPT)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DUSE_DEFER_ACCEPT")
endif(EVHTTPX_USE_DEFER_ACCEPT)

SET(CMAKE_INCLUDE_CURRENT_DIR ON)

include(ReveldbBaseConfig)

FIND_PACKAGE(Libevent REQUIRED)
FIND_PACKAGE(Leveldb REQUIRED)
FIND_PACKAGE(OpenSSL)
FIND_PATH(LIBEVENT_INCLUDE_DIR event2/event.h REQUIRED)
FIND_PATH(LEVELDB_INCLUDE_DIR leveldb/c.h REQUIRED)

if (NOT OPENSSL_FOUND)
    SET(EVHTTPX_DISABLE_SSL ON)
    SET(OPENSSL_CRYPTO_LIBRARY "")
    SET(OPENSSL_INCLUDE_DIR "")
    SET(OPENSSL_LIBRARIES "")
    SET(LIBEVENT_OPENSSL_LIBRARY "")
endif(NOT OPENSSL_FOUND)
    
INCLUDE_DIRECTORIES(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}/compat
    ${CMAKE_CURRENT_SOURCE_DIR}/
    ${CMAKE_CURRENT_SOURCE_DIR}/include/
    ${CMAKE_CURRENT_SOURCE_DIR}/src/
    ${CMAKE_CURRENT_SOURCE_DIR}/src/nedmalloc/
    ${CMAKE_CURRENT_SOURCE_DIR}/src/evhttpx/
    ${OPENSSL_INCLUDE_DIR}
    ${LIBEVENT_INCLUDE_DIR}
    ${LEVELDB_INCLUDE_DIR}
)

SET(REVELDB_EXTERNAL_LIBS
        ${LIBEVENT_LIBRARY}
        ${LEVELDB_LIBRARY}
        ${LIBEVENT_PTHREADS_LIBRARY}
        ${LIBEVENT_OPENSSL_LIBRARY}
        ${OPENSSL_LIBRARIES})

if (NOT ${LIBEVENT_PTHREADS_FOUND})
    SET(EVHTTPX_DISABLE_EVTHR 1)
endif(NOT ${LIBEVENT_PTHREADS_FOUND})

if (NOT ${LIBEVENT_OPENSSL_FOUND})
    SET(EVHTTPX_DISABLE_SSL 1)
endif(NOT ${LIBEVENT_OPENSSL_FOUND})

if (NOT ${CMAKE_BUILD_TYPE} STREQUAL "Debug")
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNDEBUG")
endif (NOT ${CMAKE_BUILD_TYPE} STREQUAL "Debug")

if (EVHTTPX_DISABLE_SSL)
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DEVHTTPX_DISABLE_SSL")
endif()

FIND_LIBRARY(LIB_DL dl)
SET(SYS_LIBS ${LIB_DL})

if (NOT APPLE)
    FIND_LIBRARY(LIB_RT rt)
    SET(SYS_LIBS ${SYS_LIBS} ${LIB_RT})
endif()

ADD_SUBDIRECTORY(src)
