cmake_minimum_required(VERSION 3.20)
project(f407_spl C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ---- MCU 参数（F407 = Cortex-M4F，硬件浮点）----
set(MCU_FLAGS -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard)

add_compile_options(
    ${MCU_FLAGS}
    -Wall -fdata-sections -ffunction-sections
    "$<$<CONFIG:Debug>:-Og>"
    "$<$<CONFIG:Debug>:-g3>"
    "$<$<CONFIG:Release>:-O2>"
)

add_compile_definitions(
    STM32F40_41xxx
    USE_STDPERIPH_DRIVER
    HSE_VALUE=8000000    # 晶振是 8MHz 时取消注释，并同步修改 system_stm32f4xx.c 的 PLL_M
    # USE_FULL_ASSERT      # 需要参数检查时取消注释，并在 main.c 实现 assert_failed()
)

add_link_options(
    ${MCU_FLAGS}
    -T${CMAKE_SOURCE_DIR}/STM32F417IG_FLASH.ld
    --specs=nano.specs --specs=nosys.specs
    -Wl,--gc-sections
    -Wl,-Map=${PROJECT_NAME}.map
    -u _printf_float     # 需要 printf 打印浮点时取消注释
)

# ---- 标准外设库：用到哪个外设就加哪个 ----
set(SPL_DIR Libraries/STM32F4xx_StdPeriph_Driver)

set(SPL_SRCS
    ${SPL_DIR}/src/misc.c
    ${SPL_DIR}/src/stm32f4xx_rcc.c
    ${SPL_DIR}/src/stm32f4xx_gpio.c
    ${SPL_DIR}/src/stm32f4xx_exti.c
    ${SPL_DIR}/src/stm32f4xx_syscfg.c
    ${SPL_DIR}/src/stm32f4xx_tim.c
    ${SPL_DIR}/src/stm32f4xx_usart.c
    ${SPL_DIR}/src/stm32f4xx_i2c.c
    ${SPL_DIR}/src/stm32f4xx_dma.c
    ${SPL_DIR}/src/stm32f4xx_adc.c
)

# ---- 用户代码（包含 User/system_stm32f4xx.c）----
file(GLOB USER_SRCS User/*.c)

add_executable(${PROJECT_NAME}.elf
    ${USER_SRCS}
    ${SPL_SRCS}
    startup_stm32f40_41xxx.s
)

target_include_directories(${PROJECT_NAME}.elf PRIVATE
    User
    Libraries/CMSIS/Include
    Libraries/CMSIS/Device/ST/STM32F4xx/Include
    ${SPL_DIR}/inc
)

# ---- 生成 hex/bin 并打印体积 ----
add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND ${CMAKE_OBJCOPY} -O ihex   $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}.elf> ${PROJECT_NAME}.bin
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${PROJECT_NAME}.elf>
)