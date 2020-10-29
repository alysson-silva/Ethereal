cmake_minimum_required(VERSION 3.10)

project(Ethereal)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED true)

file(GLOB ETHEREAL_SRC *.c)
if (MSVC)
    add_executable(ethereal ${ETHEREAL_SRC} pyrrhic/tbprobe.c)
else()
    add_executable(ethereal WIN32 ${ETHEREAL_SRC} pyrrhic/tbprobe.c)
endif()
install(TARGETS ethereal DESTINATION bin)

target_include_directories(ethereal PUBLIC "{PROJECT_BINARY_DIR}")
target_compile_options(ethereal PRIVATE -flto -Wall -Wextra -Wshadow)
# target_link_libraries(ethereal -lm)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CXX_FLAGS)
  set(CMAKE_BUILD_TYPE "Release")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release")
    add_link_options(ethereal -static)
endif()

# Options
option(USE_POPCNT "Use POPCNT extension" OFF)
option(USE_PEXT "Use PEXT extension" ON)
option(USE_AVX2 "Use AVX2 extensions" ON)
option(USE_ARMV7 "Use ARMV7 extensions" OFF)
option(USE_ARMV8 "Use ARMV8 extensions" OFF)

# x86 flags
if(USE_POPCNT OR USE_PEXT)
    # target_link_libraries(ethereal -lpthread)
    target_compile_definitions(ethereal PRIVATE USE_POPCNT)
    target_compile_options(ethereal PRIVATE -msse3 -mpopcnt -march=native)
endif()

if(USE_PEXT)
    target_compile_definitions(ethereal PRIVATE USE_PEXT)
    target_compile_options(ethereal PRIVATE -mbmi2)
endif()

if(USE_AVX2)
    # target_link_libraries(ethereal -lpthread)
    target_compile_options(ethereal PRIVATE -msse -msse3 -mpopcnt -mavx2 -msse4.1 -mssse3 -msse2 -march=native)
endif()

# ARM Flags
if(USE_ARMV7)
    target_compile_options(ethereal PRIVATE -march=armv7-a -m32 -mfloat-abi=softfp -mfpu=vfpv3-d16 -mthumb -Wl,--fix-cortex-a8)
endif()

if(USE_ARMV8)
    target_compile_options(ethereal PRIVATE -march=armv8-a -m64)
endif()