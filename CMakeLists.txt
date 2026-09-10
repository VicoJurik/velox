cmake_minimum_required(VERSION 4.0)

file(STRINGS
    "${CMAKE_CURRENT_SOURCE_DIR}/version.txt"
    VELOX_VERSION
)

project(
    Velox
    VERSION ${VELOX_VERSION}
    LANGUAGES C
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.h.in"
    "${CMAKE_CURRENT_BINARY_DIR}/generated/velox/version.h"
    @ONLY
)

set(CMAKE_CONFIGURATION_TYPES
    Debug
    DebugOpt
    Release
    CACHE STRING "Supported build configurations" FORCE
)

set(CMAKE_C_STANDARD 23)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(VELOX_ENABLE_TESTS "Build Velox tests" ON)
option(VELOX_ENABLE_CLANG_TIDY "Enable clang-tidy checks" OFF)
option(VELOX_ENABLE_WERROR "Treat warnings as errors" OFF)

# ---------------------------------------------------------------------------
# Build types
# ---------------------------------------------------------------------------

set(CMAKE_C_FLAGS_DEBUG
    "-O0 -g3"
    CACHE STRING "Flags used for the Debug build" FORCE
)

set(CMAKE_C_FLAGS_DEBUGOPT
    "-O2 -g3"
    CACHE STRING "Flags used for the DebugOpt build" FORCE
)

set(CMAKE_C_FLAGS_SANITIZE
    "-O1 -g3 -fsanitize=address,undefined -fno-omit-frame-pointer"
    CACHE STRING "Flags used for the Sanitize build" FORCE
)

set(CMAKE_C_FLAGS_TSAN
    "-O1 -g3 -fsanitize=thread -fno-omit-frame-pointer"
    CACHE STRING "Flags used for the ThreadSanitizer build" FORCE
)

set(CMAKE_C_FLAGS_RELEASE
    "-O3 -DNDEBUG"
    CACHE STRING "Flags used for the Release build" FORCE
)

# ---------------------------------------------------------------------------
# Compiler warnings
# ---------------------------------------------------------------------------

set(VELOX_WARNING_FLAGS
    -Wall
    -Wextra
    -Wpedantic

    # Conversions
    -Wconversion
    -Wsign-conversion
    -Wdouble-promotion

    # Declarations / definitions
    -Wstrict-prototypes
    -Wmissing-prototypes
    -Wold-style-definition
    -Wmissing-declarations

    # Control flow
    -Wswitch-enum
    -Wunreachable-code

    # Types / casts
    -Wcast-align
    -Wcast-qual
    -Wwrite-strings

    # Shadowing / suspicious constructs
    -Wshadow
    -Wundef
    -Wvla

    # Format / strings
    -Wformat=2

    # Miscellaneous
    -Wnull-dereference
    -Werror=implicit-function-declaration

    # Linkage / compatibility
    -fno-common
)

if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
    list(APPEND VELOX_WARNING_FLAGS
        -Wduplicated-branches
        -Wduplicated-cond
        -Wlogical-op
        -Wuseless-cast
        -Warray-bounds
        -Wstringop-overflow
        -Wstringop-overread
        -Wuse-after-free
    )
endif()

if(VELOX_ENABLE_WERROR)
    list(APPEND VELOX_WARNING_FLAGS
        -Werror
    )
endif()

# ---------------------------------------------------------------------------
# Sources
# ---------------------------------------------------------------------------

file(GLOB_RECURSE VELOX_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c"
)

add_executable(velox ${VELOX_SOURCES})

set_target_properties(velox PROPERTIES
    OUTPUT_NAME
        "velox-${PROJECT_VERSION}-$<$<CONFIG:Debug>:debug>$<$<CONFIG:DebugOpt>:debug-opt>$<$<CONFIG:Release>:current>"
)

set_target_properties(velox PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY
        "${CMAKE_BINARY_DIR}/bin"
)

target_include_directories(velox
    PRIVATE
        "${CMAKE_CURRENT_SOURCE_DIR}/include"
        "${CMAKE_CURRENT_BINARY_DIR}/generated"
)

target_compile_options(velox
    PRIVATE
        ${VELOX_WARNING_FLAGS}
)

# ---------------------------------------------------------------------------
# clang-tidy
# ---------------------------------------------------------------------------

if(VELOX_ENABLE_CLANG_TIDY)
    find_program(
        CLANG_TIDY_EXECUTABLE
        NAMES clang-tidy-21 clang-tidy
        REQUIRED
    )

    add_custom_target(velox-tidy-check
        COMMAND
            "${CLANG_TIDY_EXECUTABLE}"
            -p "${CMAKE_BINARY_DIR}"
            ${VELOX_SOURCES}
        WORKING_DIRECTORY
            "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT
            "Running clang-tidy"
        VERBATIM
    )
endif()

# ---------------------------------------------------------------------------
# Tests
# ---------------------------------------------------------------------------

include(CTest)

if(VELOX_ENABLE_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

find_program(CLANG_FORMAT_EXECUTABLE clang-format)

if(CLANG_FORMAT_EXECUTABLE)
    add_custom_target(velox-format-check
        COMMAND
            "${CLANG_FORMAT_EXECUTABLE}"
            --dry-run
            --Werror
            ${VELOX_SOURCES}
        WORKING_DIRECTORY
            "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT
            "Checking C/C++ formatting"
        VERBATIM
    )

    if(TARGET velox-test-format-check)
        add_dependencies(velox-format-check velox-test-format-check)
    endif()
endif()

add_custom_target(check)

if(TARGET velox-format-check)
    add_dependencies(check velox-format-check)
endif()

if(TARGET velox-tidy-check)
    add_dependencies(check velox-tidy-check)
endif()

if(VELOX_ENABLE_TESTS)
    add_custom_command(
        TARGET check
        POST_BUILD
        COMMAND
            "${CMAKE_CTEST_COMMAND}"
            --test-dir "${CMAKE_BINARY_DIR}"
            --output-on-failure
        WORKING_DIRECTORY
            "${CMAKE_BINARY_DIR}"
        COMMENT
            "Running tests"
        VERBATIM
    )
endif()
