
message("In the ${EXECUTABLE} level CMake...")

# Executable files
add_executable(${EXECUTABLE} "Src/app.c")	# `source_list` is from the `SourceList.cmake` file
#set_target_properties(${EXECUTABLE} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")

#target_include_directories(${EXECUTABLE} PUBLIC "Inc/")

add_custom_command(TARGET ${EXECUTABLE} PRE_BUILD
	COMMENT("pre-build here...")
)

add_custom_command(TARGET ${EXECUTABLE} PRE_LINK
	COMMENT("pre-link here...")
)

# Execute post-build to print size
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
	COMMENT("TESTING HELLO WORLD")
	COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>
)

# Convert output to hex and binary
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
	COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.hex
)

# Convert to bin file -> add conditional check?
add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
	COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.bin
)