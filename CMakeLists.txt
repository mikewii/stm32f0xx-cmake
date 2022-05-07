cmake_minimum_required(VERSION 3.5)

set(CMAKE_TRY_COMPILE_TARGET_TYPE "STATIC_LIBRARY")
project(STM32F051R8T6_Template LANGUAGES C CXX ASM)


##### Configuretion #####
set(C_STANDARD c17)
set(C_STANDARD_REQUIRED ON)
set(CXX_STANDARD c++17)
set(CXX_STANDARD_REQUIRED ON)

set(CPU "-march=armv6-m -mtune=cortex-m0")
set(MCU "${CPU} -mthumb -mfloat-abi=soft")
set(LINKER_SCRIPT "${CMAKE_CURRENT_SOURCE_DIR}/STM32F051R8Tx_FLASH.ld")
set(OPT -Og)
set(DEBUG ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_VERBOSE_MAKEFILE ON)

set(STARTUP_FILE ${CMAKE_CURRENT_SOURCE_DIR}/startup_stm32f051x8.s)
set(DEVICE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Device/ST/STM32F0xx/Include/stm32f0xx.h)
#########################


set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)

file(GLOB_RECURSE PROJECT_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Src/*c"
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Src/*cpp")

file(GLOB_RECURSE PROJECT_ASM CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Src/*s")

file(GLOB_RECURSE PROJECT_HEADERS CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Src/*h"
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Src/*hpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc/*h"
    "${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc/*hpp")

file(GLOB_RECURSE MAKEFILE CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Makefile")

file(GLOB_RECURSE HAL CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32F0xx_HAL_Driver/Src/*c"
    "${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32F0xx_HAL_Driver/Inc/*h")
list(FILTER HAL EXCLUDE REGEX ".*template\\.c$")

add_executable(${PROJECT_NAME}.elf
    ${PROJECT_SOURCES} ${PROJECT_HEADERS} ${PROJECT_ASM}
    ${HAL}
    ${STARTUP_FILE}
    ${DEVICE_FILE}
    ${MAKEFILE}
)


##### Target Configuration #####
target_compile_definitions(${PROJECT_NAME}.elf PRIVATE STM32F051x8 USE_HAL_DRIVER)
target_link_libraries(${PROJECT_NAME}.elf -lc -lm -lnosys)
###############################


set(WARNINGS "-Wall -Werror")
set(CMAKE_C_FLAGS "${MCU} ${OPT} ${WARNINGS} -fdata-sections -ffunction-sections")
set(CMAKE_CXX_FLAGS -fno-rtti -fno-exceptions -std=c++17)
set(CMAKE_ASM_FLAGS "${CMAKE_C_FLAGS} -x assembler-with-cpp")
set(CMAKE_EXE_LINKER_FLAGS "${MCU} -specs=nano.specs -T${LINKER_SCRIPT} -Wl,-Map=${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}.map,--cref -Wl,--gc-sections")

if(DEFINED DEBUG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -gdwarf-2")
endif()

#set_property(SOURCE ${PROJECT_ASM} APPEND PROPERTY COMPILE_OPTIONS "-x" "assembler-with-cpp")

target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Core/Inc)
target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32F0xx_HAL_Driver/Inc)
target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/STM32F0xx_HAL_Driver/Inc/Legacy)
target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Device/ST/STM32F0xx/Include)
target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Core/Include)
target_include_directories(${PROJECT_NAME}.elf PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/CMSIS/Include)


add_custom_command(
    TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND arm-none-eabi-objcopy -O ihex ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.elf ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.hex
    VERBATIM
)

add_custom_command(
    TARGET ${PROJECT_NAME}.elf POST_BUILD
    COMMAND arm-none-eabi-objcopy -O binary -S ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.elf ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin
    VERBATIM
)
