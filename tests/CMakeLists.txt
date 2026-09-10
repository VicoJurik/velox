add_executable(velox-test-smoke
    smoke/test_smoke.c
)

target_include_directories(velox-test-smoke
    PRIVATE
        "${PROJECT_SOURCE_DIR}/include"
        "${CMAKE_BINARY_DIR}/generated"
)

target_compile_options(velox-test-smoke
    PRIVATE
        ${VELOX_WARNING_FLAGS}
)

add_test(
    NAME smoke
    COMMAND velox-test-smoke
)

find_program(CLANG_FORMAT_EXECUTABLE clang-format)

if(CLANG_FORMAT_EXECUTABLE)
    add_custom_target(velox-test-format-check
        COMMAND
            "${CLANG_FORMAT_EXECUTABLE}"
            --dry-run
            --Werror
            ${CMAKE_CURRENT_SOURCE_DIR}/smoke/test_smoke.c
        WORKING_DIRECTORY
            "${CMAKE_CURRENT_SOURCE_DIR}"
        COMMENT
            "Checking test formatting"
        VERBATIM
    )
endif()
