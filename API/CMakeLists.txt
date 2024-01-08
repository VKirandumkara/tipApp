#[==[
    Description: This File simply tells the parent CMake file that there are two directories to add.

]==]
add_subdirectory(BSP)       # Selection on which BSP should be used will be made here.
add_subdirectory(RTOS)      # Selection on which RTOS should be used will be made here.


