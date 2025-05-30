cmake_minimum_required(VERSION 3.16)
project(px4-gpsdrivers VERSION 1.0 LANGUAGES CXX)

find_package(Qt6 REQUIRED COMPONENTS Core)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(GPS_SOURCES
    src/ashtech.cpp
    src/crc.cpp
    src/emlid_reach.cpp
    src/femtomes.cpp
    src/gps_helper.cpp
    src/mtk.cpp
    src/nmea.cpp
    src/rtcm.cpp
    src/sbf.cpp
    src/ubx.cpp
    src/unicore.cpp
)

add_library(gps ${GPS_SOURCES})

target_include_directories(gps
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
        $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

set_target_properties(gps PROPERTIES POSITION_INDEPENDENT_CODE ON)

add_library(${PROJECT_NAME}::gps ALIAS gps)

install(TARGETS gps
    EXPORT ${PROJECT_NAME}Targets
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src/
    DESTINATION include/${PROJECT_NAME}
    FILES_MATCHING PATTERN "*.h"
)

install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION lib/cmake/${PROJECT_NAME}
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION lib/cmake/${PROJECT_NAME}
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION lib/cmake/${PROJECT_NAME}
)

include(CTest)
enable_testing()

add_executable(gps-parser-test
    gps-parser-test.cpp
)

target_link_libraries(gps-parser-test PRIVATE ${PROJECT_NAME}::gps)
add_test(NAME GPSParserTest COMMAND gps-parser-test)
