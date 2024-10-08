# Copyright © SixtyFPS GmbH <info@slint.dev>
# SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-Slint-Royalty-free-2.0 OR LicenseRef-Slint-Software-3.0

idf_component_register(
    SRCS "src/slint-esp.cpp"
    INCLUDE_DIRS "include"
    REQUIRES "esp_lcd" "esp_lcd_touch")

if (CONFIG_IDF_TARGET_ARCH_XTENSA)
    set(Rust_CARGO_TARGET "xtensa-${IDF_TARGET}-none-elf")
elseif(CONFIG_IDF_TARGET_ARCH_RISCV)
    if (CONFIG_IDF_TARGET_ESP32C6 OR CONFIG_IDF_TARGET_ESP32C5 OR CONFIG_IDF_TARGET_ESP32H2)
        set(Rust_CARGO_TARGET "riscv32imac-esp-espidf")
    elseif (CONFIG_IDF_TARGET_ESP32P4)
        set(Rust_CARGO_TARGET "riscv32imafc-esp-espidf")
    else ()
        set(Rust_CARGO_TARGET "riscv32imc-esp-espidf")
    endif()
else()
    message(FATAL_ERROR "Architecture currently not supported")
endif()

set(SLINT_FEATURE_FREESTANDING ON)
set(SLINT_FEATURE_RENDERER_SOFTWARE ON)
set(SLINT_LIBRARY_CARGO_FLAGS "-Zbuild-std=core,alloc")
set(DEFAULT_SLINT_EMBED_RESOURCES "embed-for-software-renderer" CACHE STRING "")
set(CMAKE_BUILD_TYPE Release)
set(BUILD_SHARED_LIBS OFF)


if (SLINT_ESP_LOCAL_EXAMPLE)
    add_subdirectory(../.. slint_build)
else(SLINT_ESP_LOCAL_EXAMPLE)
    include(FetchContent)
    FetchContent_Declare(
        Slint
        GIT_REPOSITORY https://github.com/slint-ui/slint
        GIT_TAG v1.8.0
        SOURCE_SUBDIR api/cpp
    )
    FetchContent_MakeAvailable(Slint)
endif(SLINT_ESP_LOCAL_EXAMPLE)

target_link_libraries(${COMPONENT_LIB} PUBLIC Slint::Slint)
