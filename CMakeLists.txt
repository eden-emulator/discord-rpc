cmake_minimum_required (VERSION 3.22)
project (DiscordRPC)

include(GNUInstallDirs)

option(BUILD_EXAMPLES "Build example apps" ON)
option(DRPC_CLANG_FORMAT "Enable clang-format target" OFF)

# format
file(GLOB_RECURSE ALL_SOURCE_FILES
    examples/*.cpp examples/*.h examples/*.c
    include/*.h
    src/*.cpp src/*.h src/*.c
)

if (DRPC_CLANG_FORMAT)
    # Set CLANG_FORMAT_SUFFIX if you are using custom clang-format, e.g. clang-format-5.0
    find_program(CLANG_FORMAT_CMD clang-format${CLANG_FORMAT_SUFFIX})

    if (CLANG_FORMAT_CMD)
        add_custom_target(
            clangformat
            COMMAND ${CLANG_FORMAT_CMD}
            -i -style=file -fallback-style=none
            ${ALL_SOURCE_FILES}
            DEPENDS
            ${ALL_SOURCE_FILES}
        )
    endif(CLANG_FORMAT_CMD)
endif(DRPC_CLANG_FORMAT)

# thirdparty stuff
execute_process(
    COMMAND mkdir ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty
    ERROR_QUIET
)
# new commit that fixes c++17
set(RAPIDJSON_SHA 3b2441b87f99ab65f37b141a7b548ebadb607b96)

find_file(RAPIDJSONTEST NAMES rapidjson rapidjson-${RAPIDJSON_SHA} PATHS ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty CMAKE_FIND_ROOT_PATH_BOTH)
if (NOT RAPIDJSONTEST)
    message("no rapidjson, download")
    set(RJ_TAR_FILE ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/${RAPIDJSON_SHA}.tar.gz)
    file(DOWNLOAD https://github.com/miloyip/rapidjson/archive/${RAPIDJSON_SHA}.tar.gz ${RJ_TAR_FILE})
    execute_process(
        COMMAND ${CMAKE_COMMAND} -E tar xzf ${RJ_TAR_FILE}
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty
    )
    file(REMOVE ${RJ_TAR_FILE})
endif(NOT RAPIDJSONTEST)

find_file(RAPIDJSON NAMES rapidjson rapidjson-${RAPIDJSON_SHA} PATHS ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty CMAKE_FIND_ROOT_PATH_BOTH)

add_library(rapidjson STATIC IMPORTED ${RAPIDJSON})

# add subdirs

add_subdirectory(src)
if (BUILD_EXAMPLES)
    add_subdirectory(examples/send-presence)
endif(BUILD_EXAMPLES)
