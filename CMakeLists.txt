cmake_minimum_required(VERSION 3.22)

# if(${CMAKE_VERSION} VERSION_LESS 3.20)
#     cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
# endif()

# Setup cmake module path and compiler settings
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
message("Build type: "              ${CMAKE_BUILD_TYPE})
set(CMAKE_C_STANDARD                17)
set(CMAKE_C_STANDARD_REQUIRED       ON)
set(CMAKE_C_EXTENSIONS              ON)
set(CMAKE_CXX_STANDARD              17)
set(CMAKE_CXX_STANDARD_REQUIRED     ON)
set(CMAKE_CXX_EXTENSIONS            ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS	ON)
set(PROJ_PATH                       ${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_TOOLCHAIN_FILE			gcc-arm-none-eabi)

# Firmware Version Number
	# TODO: Automate a script to adjust these values during the commision process
set(MAJOR_VERSION 0)
set(MINOR_VERSION 9)
set(PATCH_VERSION 0)
#VERSION ${MAJOR_VERSION}.${MINOR_VERSION}.${PATCH_VERSION}

# Project Name    --- MUST EDIT ---
project(TipNovus_App)		# TODO: ADD VERSION NUMBER AND DESCRIPTION
		
enable_language(C CXX ASM)

# TODO: Change the MCU_FAMILY Variable to a user changeable var
set(MCU_FAMILY "STM32H753xx")
set(RTOS_FAMILY "FreeRTOS")
set(EXECUTABLE ${CMAKE_PROJECT_NAME})

## Adding all Subdirectories to CMake Build
add_subdirectory(App)
add_subdirectory(API)
# add_subdirectory(Drivers)
#add_subdirectory(extern)

# Read in the properties from the BSP
get_property(src_linker_script TARGET STM32H7_BSP_Lib PROPERTY LINKER_PATH)
get_property(src_parameter_path TARGET STM32H7_BSP_Lib PROPERTY PARAMETER_PATH)

# Verify Linker and Parameter values
if(src_linker_script STREQUAL "")
	message("ERROR: Invalid Linker Script.")
else()
	message("Linker added: ${src_linker_script}")
endif()

if(src_parameter_path STREQUAL "")
	message("ERROR: Invalid Parameter Script.")
else()
	message("Parameter added: ${src_parameter_path}")
endif()

# Add CPU and Compiler Parameters for the BSP
include(${src_parameter_path})

# Project symbols
target_compile_definitions(${EXECUTABLE}
	PRIVATE ${compiler_define}		# `compiler_define` is from the `STM32XXXxx_HAL_PARA.cmake` files
	PUBLIC -DDebug
)

get_target_property(OUT ${EXECUTABLE} LINK_LIBRARIES)
message(STATUS ${OUT})

# Compiler options
target_compile_options(${EXECUTABLE} PUBLIC
	-mcpu=cortex-m7
	-std=gnu17
	-g3
	-Og
	-ffunction-sections
	-fdata-sections
	-Wall
	# -fstack-usage
	# -fcyclomatic-complexity
	-mfpu=fpv5-d16
	-mfloat-abi=hard
	-mthumb
	# -Wall
	# -Wpedantic
	# -Wno-unused-parameter
	# -O0								# Optimize for size (that uses O2)

	#${CPU_PARAMETERS}				# `CPU_PARAMETERS` is from the `STM32XXXxx_HAL_PARA.cmake` files
)

# Linker options
target_link_options(${EXECUTABLE} PUBLIC
	-T${src_linker_script}
	-Wl,-Map=${CMAKE_PROJECT_NAME}.map
	-mcpu=cortex-m7
	--specs=nosys.specs
	-Wl,--gc-sections
	-static
	-mfpu=fpv5-d16
	-mfloat-abi=hard
	-mthumb -Wl,
	--start-group
	-lc 
	-lm 
	-Wl,
	--end-group
	#${CPU_PARAMETERS}
	###
	# -mcpu=cortex-m7
	# -Wl,--gc-sections 
	# -static 
	# #--specs=nano.specs 
	# -mfpu=fpv5-d16
	# -mfloat-abi=hard 
	# -mthumb
	# -g3
	# -Og
	# -Wl,--start-group 
	# -lc 
	# -lm 
	# -lstdc++ 
	# -lsupc++ 
	# -Wl,--end-group
	###
	# -Wl,--start-group
	# -lc
	# -lm
	# -lstdc++
	# -lsupc++
	# -Wl,--end-group
	# -Wl,--print-memory-usage
)

# get_target_property(cflags ${EXECUTABLE} COMPILE_OPTIONS)
# message("The project has set the following flags: ${cflags}")

get_target_property(MAIN_CFLAGS ${EXECUTABLE} COMPILE_OPTIONS)
# also see: COMPILE_DEFINITIONS INCLUDE_DIRECTORIES
message("-- Target compiler flags are: ${MAIN_CFLAGS}")