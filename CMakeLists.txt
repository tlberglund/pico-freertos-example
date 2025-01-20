cmake_minimum_required(VERSION 3.13)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(USERHOME $ENV{HOME})
set(sdkVersion 2.1.0)
set(toolchainVersion 13_3_Rel1)
set(picotoolVersion 2.1.0)
set(PICO_TOOLCHAIN_PATH "${USERHOME}/.pico-sdk/toolchain/${toolchainVersion}")

set(PROJECT_NAME freeRTOS_hello_world)
set(OUTPUT_NAME freeRTOS_hello_world)

# Choose the variety of Pico. Several parts of the build figure themselves out based on this setting.
# set(PICO_BOARD pico2 CACHE STRING "Board type")
# set(PICO_BOARD pico_w CACHE STRING "Board type")
set(PICO_BOARD pico2_w CACHE STRING "Board type")

# Point to the repo's submodules for the PicoSDK and FreeRTOS
set(PICO_SDK_PATH "../pico-sdk/")
set(FREERTOS_KERNEL_PATH "FreeRTOS-Kernel")

# Figure out board support pakcage based on PICO_BOARD
if(${PICO_BOARD} STREQUAL "pico_w")
    set(FREERTOS_KERNEL_PORT_RELATIVE_PATH "portable/ThirdParty/GCC/RP2040")
elseif(${PICO_BOARD} STREQUAL "pico2" OR ${PICO_BOARD} STREQUAL "pico2_w")
    set(FREERTOS_KERNEL_PORT_RELATIVE_PATH "portable/ThirdParty/Community-Supported-Ports/GCC/RP2350_ARM_NTZ")
else()
    message(WARNING "Unrecognized PICO_BOARD value: ${PICO_BOARD}")
endif()

set(FREERTOS_PORT_PATH ${FREERTOS_KERNEL_PATH}/${FREERTOS_KERNEL_PORT_RELATIVE_PATH})
message("USING FreeRTOS PATH=${FREERTOS_KERNEL_PATH}")
message("USING FreeRTOS PORT PATH=${FREERTOS_PORT_PATH}")

# Import Pico SDK and FreeRTOS CMake configs (this is a nameless evil, do not look upon it)
include(pico_sdk_import.cmake)
include(${FREERTOS_PORT_PATH}/library.cmake)

project(${PROJECT_NAME}
        LANGUAGES C CXX ASM)

pico_sdk_init()

add_executable(${OUTPUT_NAME}
    src/main.cpp
    src/pico_led.c
)

target_include_directories(${OUTPUT_NAME} PUBLIC 
    ${FREERTOS_KERNEL_PATH}/include 
    include/ 
)

# This makes printf() work over the USB serial port
pico_enable_stdio_usb(${OUTPUT_NAME} 1)

target_link_libraries(${OUTPUT_NAME} 
    pico_stdlib
    pico_runtime
    pico_stdio_usb
    FreeRTOS-Kernel-Heap4)

if(${PICO_BOARD} STREQUAL "pico_w")
    target_link_libraries(${OUTPUT_NAME} pico_cyw43_arch_none)
    target_compile_definitions(FreeRTOS-Kernel INTERFACE PICO_RP2040=1)
endif()

if(${PICO_BOARD} STREQUAL "pico2_w")
    target_link_libraries(${OUTPUT_NAME} pico_cyw43_arch_none)
    target_compile_definitions(FreeRTOS-Kernel INTERFACE PICO_RP2350=1)
endif()

if(${PICO_BOARD} STREQUAL "pico2")
    target_compile_definitions(FreeRTOS-Kernel INTERFACE PICO_RP2350=1)
endif()


# create map/bin/hex/uf2 file in addition to ELF.
pico_add_extra_outputs(${OUTPUT_NAME})
