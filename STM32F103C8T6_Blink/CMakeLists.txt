cmake_minimum_required(VERSION 3.22)

# ---------------------------------------------------------------------------
# STM32F103C8T6_Blink —— 由 STM32CubeMX 生成，CLion 打开时自动生成/维护本文件
#
# 编译器：CLion 的「工具链」里把 C Compiler 指到 arm-none-eabi-gcc.exe 即可，
#         也可以在命令行指定：
#         cmake -G "MinGW Makefiles" ^
#               -DCMAKE_C_COMPILER=arm-none-eabi-gcc ^
#               -DCMAKE_ASM_COMPILER=arm-none-eabi-gcc ..
# ---------------------------------------------------------------------------

# 裸机交叉编译：告诉 CMake 目标不是 Windows
if(NOT CMAKE_SYSTEM_NAME)
    set(CMAKE_SYSTEM_NAME Generic)
    set(CMAKE_SYSTEM_PROCESSOR arm)
endif()
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(STM32F103C8T6_Blink C ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_EXECUTABLE_SUFFIX ".elf")

set(MCU_FLAGS -mcpu=cortex-m3 -mthumb)

add_compile_definitions(USE_HAL_DRIVER STM32F103xB)
add_compile_options(${MCU_FLAGS} -Wall -fdata-sections -ffunction-sections -g3 $<$<CONFIG:Debug>:-Og>)

# ---------------------------------------------------------------- sources ---
set(APP_SOURCES
    Core/Src/main.c
    Core/Src/gpio.c
    Core/Src/stm32f1xx_it.c
    Core/Src/stm32f1xx_hal_msp.c
    Core/Src/system_stm32f1xx.c
    Core/Src/syscalls.c
    Core/Src/sysmem.c
    Startup/startup_stm32f103xb.s
)

file(GLOB HAL_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32F1xx_HAL_Driver/Src/*.c)
list(FILTER HAL_SOURCES EXCLUDE REGEX "_template\\.c$")

add_executable(${PROJECT_NAME} ${APP_SOURCES} ${HAL_SOURCES})

target_include_directories(${PROJECT_NAME} PRIVATE
    Core/Inc
    Drivers/STM32F1xx_HAL_Driver/Inc
    Drivers/STM32F1xx_HAL_Driver/Inc/Legacy
    Drivers/CMSIS/Device/ST/STM32F1xx/Include
    Drivers/CMSIS/Include
)

target_link_options(${PROJECT_NAME} PRIVATE
    ${MCU_FLAGS}
    -T${CMAKE_CURRENT_SOURCE_DIR}/STM32F103C8TX_FLASH.ld
    -specs=nano.specs
    -Wl,-Map=${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.map,--cref
    -Wl,--gc-sections
    -Wl,--print-memory-usage
)

target_link_libraries(${PROJECT_NAME} PRIVATE c m)

# ------------------------------------------------------ hex / bin 产物 ------
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex   $<TARGET_FILE:${PROJECT_NAME}>
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary -S $<TARGET_FILE:${PROJECT_NAME}>
            ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin
    COMMENT "Building ${PROJECT_NAME}.hex and ${PROJECT_NAME}.bin"
    VERBATIM
)
