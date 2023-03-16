cmake_minimum_required(VERSION 3.23)

#-------------------------------------------------------------------------------------------------------
# Options
#-------------------------------------------------------------------------------------------------------

option(WITH_ZLIB "Enable ZLIB support" OFF)
option(WITH_OPENSSL "Enable OpenSSL support" OFF)
option(WITH_LIBUV "Enable LibUV support" OFF)
option(WITH_QUIC "Enable experimental Http3 examples" OFF)
option(WITH_PROXY "Enable PROXY Protocol v2 support" OFF)
option(BUILD_EXAMPLES "Build examples" OFF)

#-------------------------------------------------------------------------------------------------------
# Use newer policies if possible, up to most recent tested version of CMake.
#-------------------------------------------------------------------------------------------------------
cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})

#-------------------------------------------------------------------------------------------------------
# Custom cmake functions
#-------------------------------------------------------------------------------------------------------

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_LIST_DIR}/cmake" CACHE STRING "Modules for CMake" FORCE)

#-------------------------------------------------------------------------------------------------------
# Version
#-------------------------------------------------------------------------------------------------------

set(UWEBSOCKETS_VERSION 20.37.0)

project(uwebsockets VERSION ${UWEBSOCKETS_VERSION} LANGUAGES CXX)

#-------------------------------------------------------------------------------------------------------
# Set default build to release
#-------------------------------------------------------------------------------------------------------

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build" FORCE)
endif ()

#---------------------------------------------------------------------------------------
# Compiler config
#---------------------------------------------------------------------------------------

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#-------------------------------------------------------------------------------------------------------
# Required Packages
#-------------------------------------------------------------------------------------------------------

# -- Compression
if (WITH_ZLIB)
    find_package(ZLIB REQUIRED)
endif ()

# -- SSL
if (WITH_OPENSSL)
    find_package(OpenSSL REQUIRED)
endif ()

# -- LibUB
if (WITH_LIBUV)
    find_package(libuv CONFIG REQUIRED)
endif ()

# -- QUIC
if (WITH_QUIC)
    find_package(ZLIB REQUIRED)
    find_package(Threads REQUIRED)
endif ()


#-------------------------------------------------------------------------------------------------------
# Examples
#-------------------------------------------------------------------------------------------------------

if (BUILD_EXAMPLES)
    # -- Threading
    set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
    set(THREADS_PREFER_PTHREAD_FLAG TRUE)
    find_package(Threads REQUIRED)

    # -- Common definitions
    set(EXAMPLES_LIBS
        ${ZLIB_LIBRARIES} ${OPENSSL_LIBRARIES} uv uwebsockets)

    set(EXAMPLES_INCLUDES
        ${CMAKE_CURRENT_LIST_DIR}/uSockets/src ${CMAKE_CURRENT_LIST_DIR}/src)

    # -- Echo Server
    add_executable(uwebsockets-echoserver
        ${CMAKE_CURRENT_LIST_DIR}/examples/EchoServer.cpp)

    target_include_directories(uwebsockets-echoserver
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-echoserver
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-echoserver
        RUNTIME DESTINATION bin)

    # -- Echo Server threaded
    add_executable(uwebsockets-echoserverthreaded
        ${CMAKE_CURRENT_LIST_DIR}/examples/EchoServerThreaded.cpp)

    target_include_directories(uwebsockets-echoserverthreaded
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-echoserverthreaded
        PRIVATE ${EXAMPLES_LIBS} Threads::Threads)

    install(TARGETS uwebsockets-echoserverthreaded
        RUNTIME DESTINATION bin)

    # -- Broadcasting Echo Server
    add_executable(uwebsockets-broadcastingechoserver
        ${CMAKE_CURRENT_LIST_DIR}/examples/BroadcastingEchoServer.cpp)

    target_include_directories(uwebsockets-broadcastingechoserver
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-broadcastingechoserver
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-broadcastingechoserver
        RUNTIME DESTINATION bin)

    # -- Hello World
    add_executable(uwebsockets-helloworld
        ${CMAKE_CURRENT_LIST_DIR}/examples/HelloWorld.cpp)

    target_include_directories(uwebsockets-helloworld
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-helloworld
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-helloworld
        RUNTIME DESTINATION bin)

    # -- Hello World Threaded
    add_executable(uwebsockets-helloworldthreaded
        ${CMAKE_CURRENT_LIST_DIR}/examples/HelloWorldThreaded.cpp)

    target_include_directories(uwebsockets-helloworldthreaded
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-helloworldthreaded
        PRIVATE ${EXAMPLES_LIBS} Threads::Threads)

    install(TARGETS uwebsockets-helloworldthreaded
        RUNTIME DESTINATION bin)

    if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 8.0)
        # -- HTTP Server (Needs compiler support for filesystem)
        add_executable(uwebsockets-httpserver
            ${CMAKE_CURRENT_LIST_DIR}/examples/HttpServer.cpp)

        target_include_directories(uwebsockets-httpserver
            PRIVATE ${EXAMPLES_INCLUDES})

        target_link_libraries(uwebsockets-httpserver
            PRIVATE ${EXAMPLES_LIBS} Threads::Threads)

        install(TARGETS uwebsockets-httpserver
            RUNTIME DESTINATION bin)
    endif ()

    # -- Server Name
    add_executable(uwebsockets-servername
        ${CMAKE_CURRENT_LIST_DIR}/examples/ServerName.cpp)

    target_include_directories(uwebsockets-servername
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-servername
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-servername
        RUNTIME DESTINATION bin)

    # -- Upgrade Async
    add_executable(uwebsockets-upgradeasync
        ${CMAKE_CURRENT_LIST_DIR}/examples/UpgradeAsync.cpp)

    target_include_directories(uwebsockets-upgradeasync
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-upgradeasync
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-upgradeasync
        RUNTIME DESTINATION bin)

    # -- Upgrade Sync
    add_executable(uwebsockets-upgradesync
        ${CMAKE_CURRENT_LIST_DIR}/examples/UpgradeSync.cpp)

    target_include_directories(uwebsockets-upgradesync
        PRIVATE ${EXAMPLES_INCLUDES})

    target_link_libraries(uwebsockets-upgradesync
        PRIVATE ${EXAMPLES_LIBS})

    install(TARGETS uwebsockets-upgradesync
        RUNTIME DESTINATION bin)
endif ()

#-------------------------------------------------------------------------------------------------------
# Target
#-------------------------------------------------------------------------------------------------------

# library name
set(TARGET_NAME uwebsockets)

set(SOURCE_FILES
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/bsd.c
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/context.c
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/loop.c
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/socket.c
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/eventing/epoll_kqueue.c
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/eventing/gcd.c
    )

set(HEADERS_FILES
    ${CMAKE_CURRENT_LIST_DIR}/src/App.h
    ${CMAKE_CURRENT_LIST_DIR}/src/AsyncSocket.h
    ${CMAKE_CURRENT_LIST_DIR}/src/AsyncSocketData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/BloomFilter.h
    ${CMAKE_CURRENT_LIST_DIR}/src/ChunkedEncoding.h
    ${CMAKE_CURRENT_LIST_DIR}/src/ClientApp.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3App.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3Context.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3ContextData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3Request.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3Response.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Http3ResponseData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpContext.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpContextData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpParser.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpResponse.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpResponseData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/HttpRouter.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Loop.h
    ${CMAKE_CURRENT_LIST_DIR}/src/LoopData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/MessageParser.h
    ${CMAKE_CURRENT_LIST_DIR}/src/MoveOnlyFunction.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Multipart.h
    ${CMAKE_CURRENT_LIST_DIR}/src/PerMessageDeflate.h
    ${CMAKE_CURRENT_LIST_DIR}/src/ProxyParser.h
    ${CMAKE_CURRENT_LIST_DIR}/src/QueryParser.h
    ${CMAKE_CURRENT_LIST_DIR}/src/TopicTree.h
    ${CMAKE_CURRENT_LIST_DIR}/src/Utilities.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocket.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketContext.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketContextData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketData.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketExtensions.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketHandshake.h
    ${CMAKE_CURRENT_LIST_DIR}/src/WebSocketProtocol.h
    )

# Optional sources
if (WITH_OPENSSL)
    list(APPEND SOURCE_FILES
        ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/crypto/openssl.c
        ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/crypto/sni_tree.cpp)
endif ()

if (WITH_LIBUV)
    list(APPEND SOURCE_FILES
        ${CMAKE_CURRENT_LIST_DIR}/uSockets/src/eventing/libuv.c)
endif ()

if (BUILD_SHARED_LIBS)
    set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
    set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
endif ()

# Add as a library
add_library(${TARGET_NAME} "")
add_library(${TARGET_NAME}::${TARGET_NAME} ALIAS ${TARGET_NAME})

if (BUILD_SHARED_LIBS)
    set_property(TARGET ${TARGET_NAME} PROPERTY CXX_VISIBILITY_PRESET default)
    set_property(TARGET ${TARGET_NAME} PROPERTY VISIBILITY_INLINES_HIDDEN OFF)
endif ()

# Add target sources
target_sources(${TARGET_NAME} PRIVATE ${SOURCE_FILES})
target_sources(${TARGET_NAME} PUBLIC FILE_SET HEADERS TYPE HEADERS BASE_DIRS src FILES ${HEADERS_FILES})

# Compilations definitions
if (WITH_ZLIB)
    target_compile_definitions(${TARGET_NAME} PUBLIC UWS_NO_ZLIB)
endif ()

if (WITH_OPENSSL)
    target_compile_definitions(${TARGET_NAME} PUBLIC LIBUS_USE_OPENSSL)
else ()
    target_compile_definitions(${TARGET_NAME} PUBLIC LIBUS_NO_SSL)
endif ()

if (WITH_LIBUV)
    target_compile_definitions(${TARGET_NAME} PUBLIC LIBUS_USE_LIBUV)
endif ()

if (WITH_QUIC)
    target_compile_definitions(${TARGET_NAME} PUBLIC LIBUS_USE_QUIC)
endif ()

if (WITH_PROXY)
    target_compile_definitions(${TARGET_NAME} PUBLIC UWS_WITH_PROXY)
endif ()

# Link
if (WITH_OPENSSL)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${OPENSSL_LIBRARIES})
endif ()

if (WITH_LIBUV)
    target_link_libraries(${PROJECT_NAME} PRIVATE uv)
endif ()

# Add include directories for this library
target_include_directories(${TARGET_NAME}
    PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/uSockets/src
    PUBLIC
    $<INSTALL_INTERFACE:include>
    $<INSTALL_INTERFACE:include/uwebsockets>)

# Properties
set_target_properties(${TARGET_NAME} PROPERTIES VERSION ${UWEBSOCKETS_VERSION} SOVERSION ${UWEBSOCKETS_VERSION} LINKER_LANGUAGE CXX)

# ---- Install ---- #
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(version_config ${PROJECT_BINARY_DIR}/uwebsockets-config-version.cmake)
set(project_config ${PROJECT_BINARY_DIR}/uwebsockets-config.cmake)
set(pkgconfig ${PROJECT_BINARY_DIR}/uwebsockets.pc)
set(targets_export_name uwebsockets-targets)

set(UWEBSOCKETS_CMAKE_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/uwebsockets CACHE STRING
    "Installation directory for cmake files, relative to ${CMAKE_INSTALL_PREFIX}.")

set(UWEBSOCKETS_LIB_DIR ${CMAKE_INSTALL_LIBDIR} CACHE STRING
    "Installation directory for libraries, relative to ${CMAKE_INSTALL_PREFIX}.")

set(UWEBSOCKETS_INC_DIR ${CMAKE_INSTALL_INCLUDEDIR}/uwebsockets CACHE STRING
    "Installation directory for include files, relative to ${CMAKE_INSTALL_PREFIX}.")

set(UWEBSOCKETS_PKGCONFIG_DIR ${CMAKE_INSTALL_PREFIX}/pkgconfig CACHE PATH
    "Installation directory for pkgconfig (.pc) files, relative to ${CMAKE_INSTALL_PREFIX}.")

# Generate the version, config and target files into the build directory.
# Generate version_config
write_basic_package_version_file(
    ${version_config}
    VERSION ${UWEBSOCKETS_VERSION}
    COMPATIBILITY AnyNewerVersion)

# Generate pkgconfig
configure_file(
    "${CMAKE_CURRENT_LIST_DIR}/cmake/uwebsockets.pc.in"
    "${pkgconfig}"
    @ONLY)

configure_package_config_file(
    ${CMAKE_CURRENT_LIST_DIR}/cmake/uwebsockets-config.cmake.in
    ${project_config}
    INSTALL_DESTINATION ${UWEBSOCKETS_CMAKE_DIR})

# Install version, config files
install(FILES ${project_config} ${version_config}
    DESTINATION ${UWEBSOCKETS_CMAKE_DIR})

# Install the library and headers.
install(TARGETS ${TARGET_NAME} EXPORT ${targets_export_name}
    LIBRARY DESTINATION ${UWEBSOCKETS_LIB_DIR}
    ARCHIVE DESTINATION ${UWEBSOCKETS_LIB_DIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    FILE_SET HEADERS DESTINATION ${UWEBSOCKETS_INC_DIR})

# Export the library
install(EXPORT ${targets_export_name} DESTINATION ${UWEBSOCKETS_CMAKE_DIR} NAMESPACE uwebsockets::)

# Copy pkgconfig
install(FILES "${pkgconfig}" DESTINATION "${UWEBSOCKETS_PKGCONFIG_DIR}")

# @todo Packaging ?
