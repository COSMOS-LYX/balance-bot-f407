# balance-bot-f407

STM32F407 + 标准外设库（SPL）+ CMake 的最小工程。

## 依赖
- Arm GNU Toolchain（arm-none-eabi-gcc）
- CMake ≥ 3.20、Ninja

## 编译
cmake -S . -B build -G Ninja "-DCMAKE_TOOLCHAIN_FILE=cmake/arm-none-eabi.cmake" "-DCMAKE_BUILD_TYPE=Debug"
cmake --build build

## 说明
- 晶振 8MHz：已定义 HSE_VALUE=8000000，system_stm32f4xx.c 中 PLL_M=8
- 链接脚本：STM32F417IG_FLASH.ld
