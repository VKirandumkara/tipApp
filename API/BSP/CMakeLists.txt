#[===[
Description: This file will select which BSP to use for the ongoing project build.

]===]

set(BSP_ROOT_LIB ${CMAKE_CURRENT_LIST_DIR})

# TODO: Move to Fetch_Content once BSP is released.

message("Importing BSP Items.")
if(MCU_FAMILY STREQUAL "STM32H753xx")
    add_subdirectory(stm32h753zi_bsp)

    # Linking Library Target to the Project
    target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC STM32H7_BSP_Lib)

    message("STM32H7 BSP added.")
else()
    message("Not the H7")
endif()