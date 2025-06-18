cmake_minimum_required(VERSION 3.11)

set(PROJECT "JMap")
set(PROJECT_VERSION "0.0.1")

project(${PROJECT} VERSION ${PROJECT_VERSION} LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_CXX_EXTENSIONS OFF)

set(JMAP_SRCS
    src/jmap.cc
)
set(JMAP_PUBLIC_HEADERS
    src/include/jmap.h
)

include(FetchContent)

FetchContent_Declare(
    nlohmann_json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.12.0 
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE 
)

FetchContent_MakeAvailable(nlohmann_json)

add_library(${PROJECT} STATIC ${JMAP_SRCS})

target_include_directories(
    ${PROJECT}
    PUBLIC
        $<INSTALL_INTERFACE:include/${PROJECT}>
    PRIVATE
        ${nlohmann_json_SOURCE_DIR}/include
)

target_include_directories(
    ${PROJECT}
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/include>
        $<INSTALL_INTERFACE:include/${PROJECT}>
)

set_target_properties(${PROJECT} PROPERTIES
    PUBLIC_HEADER "${JMAP_PUBLIC_HEADERS}"
)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(${PROJECT} PRIVATE
        -Wall -Wextra -Wpedantic -Werror
    )
elseif(MSVC)
    target_compile_options(${PROJECT} PRIVATE
        /W4 /WX
    )
endif()

set_target_properties(${PROJECT} PROPERTIES DEBUG_POSTFIX "d")

include(GNUInstallDirs)

if(DEFINED CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    message(STATUS "CMAKE_INSTALL_PREFIX is not set, defaulting to ${CMAKE_SOURCE_DIR}/install")
    set(CMAKE_INSTALL_PREFIX "${CMAKE_SOURCE_DIR}/install" CACHE PATH "Where the library will be installed to" FORCE)
else()
    message(STATUS "CMAKE_INSTALL_PREFIX was already set to ${CMAKE_INSTALL_PREFIX}")
endif()

install(TARGETS ${PROJECT}
    EXPORT "${PROJECT}Targets" # For creating config files
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} # Install the .a/.lib file
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} # For static libraries on some systems
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT} # Install public headers to e.g., /usr/local/include/JMap/
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY
    ${nlohmann_json_SOURCE_DIR}/include/nlohmann
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(EXPORT "${PROJECT}Targets"
    FILE "${PROJECT}Targets.cmake" 
    NAMESPACE "${PROJECT}::"
    DESTINATION cmake
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT}ConfigVersion.cmake"
    VERSION "${PROJECT_VERSION}"
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT}Config.cmake"
    INSTALL_DESTINATION cmake
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT}ConfigVersion.cmake"
    DESTINATION cmake
)
