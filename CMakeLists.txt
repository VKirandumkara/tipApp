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
	# See https://onlinedocs.microchip.com/oxy/GUID-BD1C16C8-7FA3-4D73-A4BE-241EE05EF592-en-US-5/GUID-AF6DD9C4-F930-4F63-A3DA-C94759DBCD75.html
target_compile_options(${EXECUTABLE} PUBLIC
	${CPU_PARAMETERS}			# `CPU_PARAMETERS` is from the `STM32XXXxx_HAL_PARA.cmake` files
	-DSTM32H753xx
	-std=gnu17					# C Language Standard
	
	-Wall						# Enablement of all warnings
	-Wno-comment				# Warnings from suspicious comments
	-Wno-switch					# Warnings from missing or extraneous case values
	-Wno-unused-parameter		# Warnings from unused Parameters
	-pedantic					# Warning demanded by strict ANSI C, rejects all programs with forbidden extensions

	# Garbage Collection Flags: Requires "--gc-sections" in Linker
	-ffunction-sections			# Places Functions into their own section for garbage collection
	-fdata-sections				# Places objects in its own section for garbage collection

	-Og							# Best optimization option for debugging

	## MakeFile Flags: We use Ninja
	# -MMD						# Generate Make rule for user headers
	# -MP						# add Phony Target for Dependency
	# -ggdb						# Produce debugging info for use by GDB

	## Useful for getting more system metrics later
	# -Wl, –print-memory-usage	# Returns size of program segments (Linker Flag)
	# -fstack-usage				# Useful for analyzing the stack usage of individual functions
	# -fcyclomatic-complexity	# quantitative measure of the number of linearly independent paths in a code section
)

# Linker options
target_link_options(${EXECUTABLE} PUBLIC
	-T${src_linker_script}
	-Wl,-Map=${CMAKE_PROJECT_NAME}.map
	${CPU_PARAMETERS}
	
	#--specs=nosys.specs		# Defines that System Calls should be implemented as stubs
	-Wl,--gc-sections			# Conducts the garbage collection
	-static						# Links the Program statically, does not require a dependency on dynamic libraries at runtime to run. Handles .a files
)

# get_target_property(cflags ${EXECUTABLE} COMPILE_OPTIONS)
# message("The project has set the following flags: ${cflags}")

get_target_property(MAIN_CFLAGS ${EXECUTABLE} COMPILE_OPTIONS)
# also see: COMPILE_DEFINITIONS INCLUDE_DIRECTORIES
message("-- Target compiler flags are: ${MAIN_CFLAGS}")