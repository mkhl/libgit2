link_directories(${LIBGIT2_LIBDIRS})
include_directories(${LIBGIT2_INCLUDES})
include_directories(SYSTEM ${LIBGIT2_SYSTEM_INCLUDES})

if(BUILD_FUZZERS AND NOT USE_STANDALONE_FUZZERS)
	add_c_flag(-fsanitize=fuzzer)
endif()

file(GLOB SRC_FUZZ RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *_fuzzer.c)
foreach(fuzz_target_src ${SRC_FUZZ})
	string(REPLACE ".c" "" fuzz_target_name ${fuzz_target_src})
	string(REPLACE "_fuzzer" "" fuzz_name ${fuzz_target_name})

	set(${fuzz_target_name}_SOURCES ${fuzz_target_src} ${LIBGIT2_OBJECTS})
	if(USE_STANDALONE_FUZZERS)
		list(APPEND ${fuzz_target_name}_SOURCES "standalone_driver.c")
	endif()
	add_executable(${fuzz_target_name} ${${fuzz_target_name}_SOURCES})
	set_target_properties(${fuzz_target_name} PROPERTIES C_STANDARD 90)
	target_link_libraries(${fuzz_target_name} ${LIBGIT2_LIBS})

	add_test(${fuzz_target_name} "${CMAKE_CURRENT_BINARY_DIR}/${fuzz_target_name}" "${CMAKE_CURRENT_SOURCE_DIR}/corpora/${fuzz_name}")
endforeach()
