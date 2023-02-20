cmake_minimum_required(VERSION 3.8)
project(tarxx VERSION 0.2.0 LANGUAGES CXX)

IF (WIN32)
    message(FATAL_ERROR Windows is not support at the moment)
ENDIF()

include(CheckCXXSourceCompiles)
check_cxx_source_compiles("
int main() {
    #ifndef __QNX__
    #error no qnx support
    #endif
}" WITH_QNX_SUPPORT)

option(WITH_COMPRESSION "Set to ON to enable compression support without any algorithms" ON)
option(WITH_EXAMPLE "Set to ON to build the example" ON)
option(WITH_TESTS "Set to ON to build tests" ON)
option(WITH_LZ4 "Set to ON to enable lz4 support" ON)

set(LIB_NAME tarxx)
set(${LIB_NAME}_COMPILE_DEFINITIONS "")
set(${LIB_NAME}_LINK_LIBRARIES "")
set(${LIB_NAME}_INCLUDE_DIRECTORIES "")

find_package(PkgConfig REQUIRED)

add_library(${LIB_NAME} INTERFACE)
target_include_directories(${LIB_NAME} INTERFACE ${CMAKE_CURRENT_LIST_DIR}/include)
set(CXX_EXTENSIONS_REQUIRED OFF)
if (WITH_QNX_SUPPORT)
    set(CXX_EXTENSIONS_REQUIRED ON)
endif()

if(${CMAKE_VERSION} VERSION_LESS "3.19.0")
    target_compile_features(${LIB_NAME} INTERFACE cxx_std_17)
else()
    set_target_properties(${LIB_NAME} PROPERTIES
        CXX_STANDARD 17
        CXX_STANDARD_REQUIRED ON
        CXX_EXTENSIONS ${CXX_EXTENSIONS_REQUIRED})
endif()

if (WITH_COMPRESSION)
    list(APPEND ${LIB_NAME}_COMPILE_DEFINITIONS WITH_COMPRESSION=ON)
endif()

if (WITH_LZ4)
    if (NOT WITH_COMPRESSION)
        message(FATAL_ERROR "LZ4 option needs compression to be turned on with '-DCOMPRESSION=ON'")
    endif()
    if (WITH_QNX_SUPPORT)
        set(LZ4_INCLUDE_DIRS ${CMAKE_SOURCE_DIR}/qnx-libs/lz4/lib)
        set(LZ4 liblz4.a)
        target_link_directories(${LIB_NAME} INTERFACE ${CMAKE_BINARY_DIR}/qnx-libs/lz4/build/cmake/)
    else()
        find_library(NAME liblz4.a)
    endif()


    list(APPEND ${LIB_NAME}_COMPILE_DEFINITIONS WITH_LZ4=ON)
    list(APPEND ${LIB_NAME}_LINK_LIBRARIES ${LZ4})
    list(APPEND ${LIB_NAME}_INCLUDE_DIRECTORIES ${LZ4_INCLUDE_DIRS})
endif()

target_compile_definitions(${LIB_NAME} INTERFACE ${${LIB_NAME}_COMPILE_DEFINITIONS})
target_link_libraries(${LIB_NAME} INTERFACE ${${LIB_NAME}_LINK_LIBRARIES})
target_include_directories(${LIB_NAME} INTERFACE ${${LIB_NAME}_INCLUDE_DIRECTORIES})

if (WITH_EXAMPLE)
    set(EXAMPLE_NAME tarxx-example)
    add_executable(${EXAMPLE_NAME} tarxx-example.cpp)
    set_target_properties(${EXAMPLE_NAME} PROPERTIES
            CXX_STANDARD 17
            CXX_STANDARD_REQUIRED ON
            CXX_EXTENSIONS ${CXX_EXTENSIONS_REQUIRED})

    target_link_libraries(${EXAMPLE_NAME} tarxx)
    target_compile_definitions(${EXAMPLE_NAME} PRIVATE ${${LIB_NAME}_COMPILE_DEFINITIONS})
endif()

if (WITH_TESTS)
    add_subdirectory(tests)
endif()

message(STATUS "WITH_EXAMPLE = ${WITH_EXAMPLE}" )
message(STATUS "WITH_TESTS = ${WITH_TESTS}" )
message(STATUS "WITH_COMPRESSION = ${WITH_COMPRESSION}" )
message(STATUS "WITH_LZ4 = ${WITH_LZ4}" )

