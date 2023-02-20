set(SOURCES "")
set(LIBRARIES "tarxx" "utils")

if (WITH_LZ4)
    list(APPEND SOURCES lz4.cpp)
endif()

list(APPEND SOURCES tar.cpp)

set(TARGET unit-tests)
if (NOT WITH_QNX_SUPPORT)
    find_package(GTest REQUIRED)
    pkg_check_modules(GMock REQUIRED "gmock")
    list(APPEND LIBRARIES GTest::GTest GTest::Main)
else()
    list(APPEND LIBRARIES "c++fs" "gtest" "gtest_main" "socket")
endif()

enable_testing()

add_executable(
        ${TARGET}
        ${SOURCES}
)

target_link_libraries(
        ${TARGET}
        ${LIBRARIES}
)

add_custom_target(
        unit-test-run
        ${TARGET}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS unit-tests
)

set_target_properties(${TARGET} PROPERTIES
        CXX_STANDARD 17
        CXX_STANDARD_REQUIRED ON
        CXX_EXTENSIONS ${CXX_EXTENSIONS_REQUIRED})
if (NOT WITH_QNX_SUPPORT)
    include(GoogleTest)
    gtest_discover_tests(${TARGET})
endif()
