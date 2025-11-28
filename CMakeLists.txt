cmake_minimum_required(VERSION 3.22)
project(ForceCloseOreUI LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(FetchContent)

FetchContent_Declare(
    nlohmann_json
    GIT_REPOSITORY https://github.com/nlohmann/json.git
    GIT_TAG v3.11.3
)
FetchContent_MakeAvailable(nlohmann_json)

FetchContent_Declare(
    fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG 10.2.1
)
FetchContent_MakeAvailable(fmt)

FetchContent_Declare(
    preloader_android
    GIT_REPOSITORY https://github.com/LiteLDev/preloader-android.git
    GIT_TAG main
)
FetchContent_GetProperties(preloader_android)
if(NOT preloader_android_POPULATED)
    FetchContent_Populate(preloader_android)
    
    file(READ "${preloader_android_SOURCE_DIR}/src/pl/Logger.h" LOGGER_CONTENT)
    string(REPLACE "#include <format>" "#include <fmt/format.h>" LOGGER_CONTENT "${LOGGER_CONTENT}")
    string(REPLACE "std::vformat" "fmt::vformat" LOGGER_CONTENT "${LOGGER_CONTENT}")
    string(REPLACE "std::make_format_args" "fmt::make_format_args" LOGGER_CONTENT "${LOGGER_CONTENT}")
    file(WRITE "${preloader_android_SOURCE_DIR}/src/pl/Logger.h" "${LOGGER_CONTENT}")
    
    add_subdirectory(${preloader_android_SOURCE_DIR} ${preloader_android_BINARY_DIR})
endif()

target_link_libraries(preloader PUBLIC fmt::fmt)

file(GLOB_RECURSE SOURCES "src/*.cpp")

add_library(ForceCloseOreUI SHARED ${SOURCES})

target_include_directories(ForceCloseOreUI PRIVATE src)
target_compile_options(ForceCloseOreUI PRIVATE -Os -fvisibility=hidden -ffunction-sections -fdata-sections)
target_link_options(ForceCloseOreUI PRIVATE -Wl,--gc-sections -Wl,--strip-all)

find_library(log-lib log)

target_link_libraries(ForceCloseOreUI
    PRIVATE
    preloader
    nlohmann_json::nlohmann_json
    ${log-lib}
)

