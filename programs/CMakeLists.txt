
# # FIXME: it would be more portable to use cmake builtin function detection
# execute_process(
#   RESULT_VARIABLE DETECT_RESULT_VARIABLE
#   WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/programs
#   COMMAND bash ${PROJECT_SOURCE_DIR}/scripts/detect.sh
#   OUTPUT_FILE config.h
#   ERROR_VARIABLE env_ev
# )
# 
# if(NOT ${DETECT_RESULT_VARIABLE} STREQUAL "0")
#   message(FATAL_ERROR "detect script failed: ${DETECT_RESULT_VARIABLE}")
# endif()

# See 
# https://go.googlesource.com/gollvm/+/master/cmake/modules/ConfigSetup.cmake
#
check_symbol_exists(clock_gettime "time.h" HAVE_CLOCK_GETTIME)
check_symbol_exists(futimens "fcntl.h;sys/stat.h" HAVE_FUTIMENS)
check_symbol_exists(futimes "sys/time.h" HAVE_FUTIMES)
check_symbol_exists(posix_fadvise "fcntl.h" HAVE_POSIX_FADVISE)
check_symbol_exists(posix_madvise "sys/mman.h" HAVE_POSIX_MADVISE)

configure_file(${PROJECT_SOURCE_DIR}/programs/config.h.in 
               ${PROJECT_SOURCE_DIR}/programs/config.h)

set(TEST_PROGRAMS_DEFINITIONS _POSIX_C_SOURCE=200809L
                              _FILE_OFFSET_BITS=64
                              HAVE_CONFIG_H)

if (DEFLATE_TEST_SUPPORT__DO_NOT_USE)
  list(APPEND TEST_PROGRAMS_DEFINITIONS TEST_SUPPORT__DO_NOT_USE)
endif()

if (BUILD_SHARED_LIBS)
  set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

# Small library for common code
add_library(programs)
target_sources(programs PRIVATE prog_util.c test_util.c tgetopt.c)
target_compile_definitions(programs PRIVATE ${TEST_PROGRAMS_DEFINITIONS})
target_link_libraries(programs deflate)

# test checksums
set(PROGRAM_TARGET_NAMES
  gzip
  benchmark
  checksum
  test_checksums
  test_custom_malloc
  test_incomplete_codes
  test_litrunlen_overflow
  test_slow_decompression
  test_trailing_bytes
)
foreach(PROGRAM_TARGET_NAME ${PROGRAM_TARGET_NAMES})
  add_executable(${PROGRAM_TARGET_NAME} ${PROGRAM_TARGET_NAME}.c)
  target_link_libraries(${PROGRAM_TARGET_NAME} programs deflate z)
  target_compile_definitions(${PROGRAM_TARGET_NAME} PRIVATE ${TEST_PROGRAMS_DEFINITIONS})
endforeach()
