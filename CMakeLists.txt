cmake_minimum_required(VERSION 3.13)

# pico_sdk_init.cmake says to place this above the project() call:
set(PICO_SDK_PATH "${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk")
set(PICO_PLATFORM "rp2040")
set(PICO_TARGET "PICO")


set(PICO_CONFIG_HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/inc/sdk_config.h")
list(APPEND PICO_CONFIG_HEADER_FILES "${PICO_SDK_PATH}/src/rp2_common/pico_time_adapter/include/pico/time_adapter.h")
list(APPEND PICO_CONFIG_HEADER_FILES "${PICO_SDK_PATH}/src/rp2_common/hardware_timer/include/hardware/timer.h")

# set(PICO_DEFAULT_BOOT_STAGE2 "boot2_w25q080") # second stage bootloader, read from flash chip
include(${PICO_SDK_PATH}/pico_sdk_init.cmake)
include(pico-util.cmake)

project( codal-rp2040 )
enable_language(ASM)

include("${CODAL_UTILS_LOCATION}")

set(HAL_RP2040_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk/src/rp2040")
set(HAL_RP2_COMMON_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk/src/rp2_common")
set(HAL_COMMON_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk/src/common")
set(HAL_HOST_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/pico-sdk/src/host")

# RECURSIVE_FIND_DIR(PROCESSOR_DIR_INCLUDES "${PROCESSOR_INCLUDE_ROOT}", "*.h")
# RECURSIVE_FIND_DIR(HAL_DIR_INCLUDES "${PROCESSOR_INCLUDES}", "*.h")
RECURSIVE_FIND_DIR(TOP_LEVEL_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/inc" "*.h")
RECURSIVE_FIND_FILE(TOP_LEVEL_SOURCE0 "${CMAKE_CURRENT_SOURCE_DIR}/source" "*.cpp")
RECURSIVE_FIND_FILE(TOP_LEVEL_SOURCE1 "${CMAKE_CURRENT_SOURCE_DIR}/source" "*.c")

list(APPEND TOP_LEVEL_SOURCE0 "asm/CortexContextSwitch.s")


pico_sdk_init()



# It seems like the following are not required and they generate errors when included.
# I am probably overlooking something though:
# include("${HAL_COMMON_ROOT}/pico_base_headers/generate_config_header.cmake")
# add_subdirectory(${HAL_COMMON_ROOT})
# add_subdirectory(${HAL_RP2_COMMON_ROOT})
# add_subdirectory(${HAL_RP2040_ROOT})

set(TARGET_DEPS 
    cmsis_core
    hardware_sync
    hardware_irq
    hardware_gpio
    hardware_spi
    hardware_i2c
    hardware_uart
    hardware_pwm
    hardware_flash
    hardware_pio
    hardware_dma
    hardware_irq
    hardware_regs
    hardware_structs
    hardware_exception
    hardware_divider
    pico_runtime
    pico_float
    pico_double)

include_directories(${HAL_INCLUDES})

# create our target
add_library(codal-rp2040 ${TOP_LEVEL_SOURCE} ${TOP_LEVEL_SOURCE0} ${TOP_LEVEL_SOURCE1} ${HAL_SOURCE})

target_link_libraries(codal-rp2040 codal-core ${OBJECT_FILES} ${ARCHIVE_FILES} ${TARGET_DEPS})

#set_target_properties(codal-samd PROPERTIES LINKER_LANGUAGE CXX)

message("${PROCESSOR_DIR_INCLUDES} ${TOP_LEVEL_INCLUDE_DIRS}")
# expose it to parent cmake.
target_include_directories(codal-rp2040 PUBLIC ${TOP_LEVEL_INCLUDE_DIRS} ${PROCESSOR_DIR_INCLUDES} ${HAL_INCLUDES})
