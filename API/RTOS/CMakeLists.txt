#[===[
Description: This file will select which RTOS to use for the current build.

]===]

set(RTOS_ROOT_LIB ${CMAKE_CURRENT_LIST_DIR})

message("Importing RTOS Items.")
if(RTOS_FAMILY STREQUAL "FreeRTOS")

    message("FreeRTOS Port: ${FREERTOS_PORT}")

    add_subdirectory(freertos/FreeRTOS/Source)
    target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC freertos_kernel)

    message("FreeRTOS added.")
else()
    message("Not FreeRTOS")
endif()